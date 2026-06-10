# Infrastructure Audit Checklist — VPS · Nginx · Docker Runtime · PM2 · Cloudflare · Backups · DR

> Auditoría de la infraestructura en operación (no del pipeline — eso es `devops.md`).
> Enfocado en el stack real Zentra: **Hetzner VPS + Nginx + Docker/PM2 + Cloudflare + Chatwoot + n8n**.
> Complementa `security.md` Parte B (vectores específicos del stack). Marca evidencia: **[CMD]** / **[CODE]** / **[INF]**.

**Estándares:** CIS Benchmarks (Linux, Docker, Nginx), ISO/IEC 27001:2022 A.8/A.12, NIST SP 800-123 (server hardening).

---

## 1. HARDENING DEL VPS (Hetzner / Linux)

### 1.1 Acceso
- [ ] SSH solo por llave pública — `PasswordAuthentication no` en `sshd_config`
- [ ] `PermitRootLogin no`; acceso por usuario sudo nominal
- [ ] Cambiar puerto SSH default es opcional (security by obscurity, no sustituye lo anterior)
- [ ] Llaves SSH por persona (no compartidas); revocación documentada al salir alguien del equipo
- [ ] `fail2ban` activo con jail para sshd y para endpoints de login de la app
- [ ] Acceso administrativo desde IPs conocidas o vía Tailscale/VPN cuando sea posible (patrón Zentra: túnel Tailscale a PC local)
> Comandos: `sshd -T | grep -E 'passwordauthentication|permitrootlogin'`, `fail2ban-client status`

### 1.2 Firewall y superficie
- [ ] `ufw`/nftables con política **deny por defecto**; solo 22/80/443 abiertos a internet
- [ ] Servicios internos (PostgreSQL 5432, Redis 6379, n8n 5678, Chatwoot, apps Node) **bind a 127.0.0.1** o red Docker interna — NUNCA `0.0.0.0` expuesto
- [ ] Verificar puertos escuchando hacia fuera: `ss -tlnp` — todo lo que escuche en `0.0.0.0` debe justificarse
- [ ] Origen detrás de Cloudflare: firewall acepta 80/443 SOLO desde rangos de Cloudflare (evita bypass del WAF por IP directa)

### 1.3 Sistema operativo
- [ ] Actualizaciones de seguridad automatizadas (`unattended-upgrades` en Debian/Ubuntu)
- [ ] Sin CVEs críticos pendientes en el OS (`apt list --upgradable`)
- [ ] Reloj sincronizado (NTP/chrony) — crítico para TLS, logs y tokens anti-replay
- [ ] Espacio en disco monitoreado (un disco lleno tumba Postgres, PM2 y Docker) — alerta a 80%
- [ ] Swap configurado para servicios que pueden picar en memoria (Node, Postgres)
- [ ] Usuarios del sistema sin shells innecesarios; sin cuentas huérfanas

---

## 2. NGINX (reverse proxy en operación)

- [ ] `server_tokens off;` — no exponer versión
- [ ] TLS 1.2+ únicamente; ciphers modernos; HSTS con `includeSubDomains`; OCSP stapling
- [ ] Headers de seguridad con `always`: CSP, `X-Content-Type-Options`, `X-Frame-Options`/`frame-ancestors`, `Referrer-Policy`, `Permissions-Policy`
- [ ] `location ~ /\. { deny all; }` — bloquea servir `.git`, `.env`, `.htpasswd`
- [ ] `client_max_body_size` explícito por endpoint (uploads vs API)
- [ ] Timeouts: `client_body_timeout`, `client_header_timeout`, `proxy_read_timeout`, `send_timeout`
- [ ] `limit_req_zone` en login y endpoints sensibles
- [ ] Gzip/Brotli para texto; cache headers correctos (assets con hash `immutable`, HTML/API `no-store`/corto)
- [ ] `proxy_pass` apunta a `127.0.0.1`/upstream interno, nunca a un servicio público
- [ ] `real_ip` configurado desde rangos Cloudflare para que rate limit y logs vean la IP real del cliente
- [ ] Renovación de certificados automatizada (certbot cron o cert de origen Cloudflare con vencimiento monitoreado)
- [ ] Config probada antes de recargar: `nginx -t && systemctl reload nginx` (nunca `restart` a ciegas en prod)
> Nota Zentra: cuidado con `location` exact-match vs prefix (gotcha documentado en proyectos Zeni/SEO).

---

## 3. RUNTIME DE CONTENEDORES (Docker / Compose en operación)

- [ ] Contenedores corren como usuario no-root (`USER` en Dockerfile o `user:` en compose) — CIS Docker 4.1
- [ ] `restart: unless-stopped` o `always` en servicios de producción
- [ ] **Healthchecks** definidos por servicio + `depends_on: condition: service_healthy`
- [ ] Resource limits: `mem_limit`/`cpus` (o `deploy.resources`) — un contenedor no debe poder tumbar el host
- [ ] Volúmenes nombrados para datos persistentes (Postgres, Chatwoot uploads) — declarados explícitamente
- [ ] `/var/run/docker.sock` NO montado en contenedores expuestos (= root del host)
- [ ] `no-new-privileges:true`; `cap_drop: [ALL]` + caps mínimos; `read_only` donde se pueda
- [ ] Red Docker segmentada: BD/Redis en red interna sin `ports:` publicados al host
- [ ] Logs con driver y rotación configurada (`max-size`, `max-file`) — no llenar disco
- [ ] Imágenes pinneadas por versión; re-escaneo periódico con Trivy (CVEs nuevos aparecen en imágenes viejas)
- [ ] Compose de producción separado del de desarrollo (sin bind mounts de código fuente en prod)
> Comandos: `docker ps --format '{{.Names}}: {{.Status}}'` (healthy?), `docker inspect <c> | jq '.[0].HostConfig.Memory'`

---

## 4. PM2 (apps Node)

- [ ] PM2 corre bajo usuario sin privilegios dedicado (patrón Zeni: usuario `zeni`, NO root)
- [ ] ⚠️ Gotcha Zentra documentado: pueden coexistir DOS PM2 (root y usuario). Usar SIEMPRE `su - zeni -c 'pm2 ...'`; el PM2 de root suele estar errored y se ignora
- [ ] Secrets cargados desde `secrets.env` (source antes de `pm2 restart --update-env`), nunca hardcodeados en `ecosystem.config.js`
- [ ] `pm2 save` ejecutado tras cambios de estado; `pm2 startup` configurado (sobrevive reboot)
- [ ] `pm2-logrotate` instalado y configurado (max size + retención)
- [ ] `max_memory_restart` definido (auto-restart ante memory leak)
- [ ] `instances` y `exec_mode: cluster` evaluados según carga; sticky sessions si hay estado en memoria
- [ ] App siempre detrás de Nginx, nunca el puerto de PM2 expuesto a internet
- [ ] Procedimiento de restart documentado: `source secrets.env → pm2 restart <app> --update-env → pm2 save`

---

## 5. CLOUDFLARE

- [ ] Registros DNS críticos en modo "proxied" (naranja) — oculta IP de origen
- [ ] SSL/TLS mode = **Full (Strict)** con cert de origen — NUNCA "Flexible" (deja claro el tramo Cloudflare↔origen)
- [ ] WAF managed rules activas; rate limiting en login/API
- [ ] Bot fight mode / challenge según el caso
- [ ] DNSSEC habilitado
- [ ] API tokens de Cloudflare con permisos mínimos y rotables (incidente histórico Zentra: token CF expuesto — ver memoria)
- [ ] Page rules / cache rules no cacheando respuestas autenticadas o con datos personales
- [ ] Origin no alcanzable por IP directa (firewall del VPS solo acepta rangos Cloudflare)

---

## 6. BACKUPS Y RECUPERACIÓN ANTE DESASTRES (DR)

- [ ] Backup automatizado de PostgreSQL (`pg_dump`/`pg_basebackup`) con cron
- [ ] Backups **cifrados** y almacenados **off-site** (ej: Cloudflare R2 — patrón Zeni Sprint 8)
- [ ] Retención definida (diaria 7d / semanal 4w / mensual 6m, o según criticidad)
- [ ] **Restore probado** — un backup no verificado no es un backup. Agendar prueba de restore periódica
- [ ] RPO y RTO definidos por proyecto (¿cuántos datos podemos perder? ¿cuánto downtime toleramos?)
- [ ] Backup de volúmenes de Docker (uploads de Chatwoot, datos de n8n, `N8N_ENCRYPTION_KEY`)
- [ ] Snapshots del VPS (Hetzner) configurados como capa adicional
- [ ] Runbook de recuperación: pasos exactos para restaurar desde cero en un VPS nuevo
- [ ] Las credenciales/secrets necesarios para restaurar NO viven solo en el servidor que podría perderse

---

## 7. MONITOREO, LOGS Y ALERTAS

- [ ] Uptime monitoring externo (UptimeRobot — patrón Zeni Sprint 7) en URLs críticas
- [ ] Error tracking (Sentry) integrado en apps con DSN fuera de git
- [ ] Healthchecks usan `127.0.0.1` explícito donde IPv6/localhost da problemas (gotcha TechBridge discovery bot)
- [ ] Alertas de: caída de servicio, disco >80%, memoria alta, picos de 5xx, fallos de backup
- [ ] Logs centralizados o al menos accesibles y rotados (Nginx, PM2, Docker, Postgres)
- [ ] Retención de logs ≥90 días; logs de auth y cambios críticos protegidos contra borrado
- [ ] Logs sin secretos/PII en claro
- [ ] Un humano recibe las alertas por un canal que realmente mira (WhatsApp/Slack/email), no solo un dashboard que nadie abre

---

## 8. RESILIENCIA Y CONTINUIDAD

- [ ] Single points of failure identificados (¿un solo VPS? ¿una sola persona que sabe operarlo?)
- [ ] Procedimiento ante caída del proveedor (Hetzner down): ¿cómo levanto en otro lado?
- [ ] Dependencias externas con fallback o degradación elegante (WhatsApp API, LLM, pasarela down → no tumba toda la app)
- [ ] Migraciones de BD backward-compatible (deploy nuevo no rompe instancia vieja durante el rollout)
- [ ] Rollback ensayado: tag anterior + `pm2 restart` / `docker compose up` de la versión previa

---

## FORMATO DE REPORTE INFRASTRUCTURE

```
## Infrastructure Audit Report — [Proyecto]
**Servidor:** [proveedor / IP/hostname]
**Componentes:** [VPS / Nginx / Docker / PM2 / Cloudflare / Chatwoot / n8n]
**Método:** [comandos ejecutados / lectura de config / inferido]
**Revisado:** [fecha]

### Hallazgos por área
🔴/🟡/🟢 [hallazgo con evidencia y corrección]

### Puntuación
| Área | Estado | Evidencia |
|------|--------|-----------|
| Hardening VPS | OK/WARN/FAIL | CMD/CODE/INF |
| Nginx | OK/WARN/FAIL | |
| Runtime contenedores | OK/WARN/FAIL | |
| PM2 | OK/WARN/FAIL | |
| Cloudflare | OK/WARN/FAIL | |
| Backups / DR | OK/WARN/FAIL | |
| Monitoreo / Alertas | OK/WARN/FAIL | |
| Resiliencia | OK/WARN/FAIL | |

**Riesgo operacional:** ALTO / MEDIO / BAJO
**Listo para producción:** SÍ / NO — [razón principal]
**RPO/RTO:** [definidos / NO definidos]
```
