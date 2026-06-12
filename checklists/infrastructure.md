# Infrastructure Audit Checklist — Runtime · Proxy · Contenedores · CDN/WAF · Backups · DR

> Auditoría de la infraestructura en operación (no del pipeline — eso es `devops.md`).
> **Agnóstico de proveedor.** Aplica según el stack que detectes o el `.sqa/profile.md` del
> proyecto. Los nombres de productos concretos (un VPS, un CDN, un gestor de procesos) son
> **ejemplos** — evalúa el control subyacente contra lo que el proyecto realmente use.
> Complementa `security.md` Parte B. Marca evidencia: **[CMD]** / **[CODE]** / **[INF]**.

> **Perfil de stack:** lee primero `profiles/default.md` (auto-detección) o el
> `.sqa/profile.md` del proyecto. Resuelve las variables `{{RUNTIME}}`, `{{PROXY}}`,
> `{{CDN_WAF}}`, `{{SECRETS_STORE}}`, `{{BACKUP_TARGET}}`, `{{DB}}`, `{{PRIV_USER}}`, `{{HOST}}`
> antes de auditar. Si una sección no aplica al stack (p.ej. no hay reverse proxy propio en
> un PaaS gestionado), **omítela — no la marques FAIL**.

**Estándares:** CIS Benchmarks (Linux, Docker, Nginx), ISO/IEC 27001:2022 A.8/A.12, NIST SP 800-123 (server hardening).

---

## 1. HARDENING DEL HOST `{{HOST}}` (VPS/servidor Linux — omitir si es PaaS gestionado)

### 1.1 Acceso
- [ ] SSH solo por llave pública — `PasswordAuthentication no` en `sshd_config`
- [ ] `PermitRootLogin no`; acceso por usuario sudo nominal
- [ ] Cambiar puerto SSH default es opcional (security by obscurity, no sustituye lo anterior)
- [ ] Llaves SSH por persona (no compartidas); revocación documentada al salir alguien del equipo
- [ ] `fail2ban` activo con jail para sshd y para endpoints de login de la app
- [ ] Acceso administrativo desde IPs conocidas o vía Tailscale/VPN cuando sea posible
> Comandos: `sshd -T | grep -E 'passwordauthentication|permitrootlogin'`, `fail2ban-client status`

### 1.2 Firewall y superficie
- [ ] `ufw`/nftables con política **deny por defecto**; solo 22/80/443 abiertos a internet
- [ ] Servicios internos (base de datos, cache, colas, workers — p.ej. PostgreSQL 5432, Redis 6379) **bind a 127.0.0.1** o red interna del orquestador — NUNCA `0.0.0.0` expuesto
- [ ] Verificar puertos escuchando hacia fuera: `ss -tlnp` — todo lo que escuche en `0.0.0.0` debe justificarse
- [ ] Si hay `{{CDN_WAF}}` delante: firewall acepta 80/443 SOLO desde los rangos del CDN/WAF (evita bypass por IP directa al origen)

### 1.3 Sistema operativo
- [ ] Actualizaciones de seguridad automatizadas (`unattended-upgrades` en Debian/Ubuntu)
- [ ] Sin CVEs críticos pendientes en el OS (`apt list --upgradable`)
- [ ] Reloj sincronizado (NTP/chrony) — crítico para TLS, logs y tokens anti-replay
- [ ] Espacio en disco monitoreado (un disco lleno tumba DB, runtime y contenedores) — alerta a 80%
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
- [ ] `real_ip`/`X-Forwarded-For` confiable configurado desde el CDN/WAF para que rate limit y logs vean la IP real del cliente
- [ ] Renovación de certificados automatizada (ACME/certbot, o cert de origen del CDN con vencimiento monitoreado)
- [ ] Config probada antes de recargar: `nginx -t && systemctl reload nginx` (nunca `restart` a ciegas en prod)
> Nota: cuidado con `location` exact-match vs prefix (gotcha común que rompe rutas). Ver gotchas del perfil de stack si el proyecto define uno.

---

## 3. RUNTIME DE CONTENEDORES (Docker / Compose en operación)

- [ ] Contenedores corren como usuario no-root (`USER` en Dockerfile o `user:` en compose) — CIS Docker 4.1
- [ ] `restart: unless-stopped` o `always` en servicios de producción
- [ ] **Healthchecks** definidos por servicio + `depends_on: condition: service_healthy`
- [ ] Resource limits: `mem_limit`/`cpus` (o `deploy.resources`) — un contenedor no debe poder tumbar el host
- [ ] Volúmenes nombrados para datos persistentes (DB, uploads de la app) — declarados explícitamente
- [ ] `/var/run/docker.sock` NO montado en contenedores expuestos (= root del host)
- [ ] `no-new-privileges:true`; `cap_drop: [ALL]` + caps mínimos; `read_only` donde se pueda
- [ ] Red Docker segmentada: BD/Redis en red interna sin `ports:` publicados al host
- [ ] Logs con driver y rotación configurada (`max-size`, `max-file`) — no llenar disco
- [ ] Imágenes pinneadas por versión; re-escaneo periódico con Trivy (CVEs nuevos aparecen en imágenes viejas)
- [ ] Compose de producción separado del de desarrollo (sin bind mounts de código fuente en prod)
> Comandos: `docker ps --format '{{.Names}}: {{.Status}}'` (healthy?), `docker inspect <c> | jq '.[0].HostConfig.Memory'`

---

## 4. RUNTIME DE LA APP `{{RUNTIME}}` (gestor de procesos / contenedor / systemd — omitir si es PaaS)

- [ ] El runtime corre bajo usuario sin privilegios dedicado `{{PRIV_USER}}` (NO root)
- [ ] Sin instancias duplicadas del gestor de procesos bajo distintos usuarios (root vs app) que generen ambigüedad operativa — ver gotchas del perfil de stack si aplica
- [ ] Secrets cargados desde `{{SECRETS_STORE}}` en runtime, nunca hardcodeados en el archivo de configuración del proceso
- [ ] Estado del runtime persistido tras cambios y configurado para sobrevivir reboot (p.ej. `pm2 save` + `pm2 startup`, o unit de systemd `enabled`)
- [ ] Rotación de logs del runtime configurada (max size + retención)
- [ ] Auto-restart ante memory leak definido (p.ej. `max_memory_restart`, o `MemoryMax` en systemd)
- [ ] Concurrencia evaluada según carga (cluster/instancias); sticky sessions si hay estado en memoria
- [ ] App siempre detrás de `{{PROXY}}`, nunca el puerto del runtime expuesto directo a internet
- [ ] Procedimiento de restart documentado y reproducible (incluye recarga de secretos desde `{{SECRETS_STORE}}`)
> Comandos de ejemplo según `{{RUNTIME}}` (resolver desde el perfil de stack). Para el perfil Zentra (PM2), ver `profiles/zentra.md`.

---

## 5. CDN / WAF `{{CDN_WAF}}` (omitir si el proyecto no usa uno; ejemplos con Cloudflare)

- [ ] Registros DNS críticos en modo "proxied" — oculta IP de origen
- [ ] Si hay CDN/WAF con TLS: modo extremo-a-extremo (**Full Strict** o equivalente) con cert de origen — NUNCA terminación que deje el tramo CDN↔origen en claro
- [ ] WAF managed rules activas; rate limiting en login/API
- [ ] Bot fight mode / challenge según el caso
- [ ] DNSSEC habilitado
- [ ] API tokens del CDN/WAF con permisos mínimos y rotables
- [ ] Page rules / cache rules no cacheando respuestas autenticadas o con datos personales
- [ ] Origen no alcanzable por IP directa (firewall solo acepta rangos del CDN/WAF)

---

## 6. BACKUPS Y RECUPERACIÓN ANTE DESASTRES (DR)

- [ ] Backup automatizado de PostgreSQL (`pg_dump`/`pg_basebackup`) con cron
- [ ] Backups **cifrados** y almacenados **off-site** en `{{BACKUP_TARGET}}`
- [ ] Retención definida (diaria 7d / semanal 4w / mensual 6m, o según criticidad)
- [ ] **Restore probado** — un backup no verificado no es un backup. Agendar prueba de restore periódica
- [ ] RPO y RTO definidos por proyecto (¿cuántos datos podemos perder? ¿cuánto downtime toleramos?)
- [ ] Backup de volúmenes de contenedores (uploads, datos de servicios y sus claves de cifrado — sin la encryption key los datos quedan ilegibles)
- [ ] Snapshots del host configurados como capa adicional (si el proveedor los ofrece)
- [ ] Runbook de recuperación: pasos exactos para restaurar desde cero en un VPS nuevo
- [ ] Las credenciales/secrets necesarios para restaurar NO viven solo en el servidor que podría perderse

---

## 7. MONITOREO, LOGS Y ALERTAS

- [ ] Uptime monitoring externo (UptimeRobot — patrón Zeni Sprint 7) en URLs críticas
- [ ] Error tracking (Sentry) integrado en apps con DSN fuera de git
- [ ] Healthchecks usan `127.0.0.1` explícito donde IPv6/localhost da problemas (gotcha TechBridge discovery bot)
- [ ] Alertas de: caída de servicio, disco >80%, memoria alta, picos de 5xx, fallos de backup
- [ ] Logs centralizados o al menos accesibles y rotados (proxy, runtime, contenedores, DB)
- [ ] Retención de logs ≥90 días; logs de auth y cambios críticos protegidos contra borrado
- [ ] Logs sin secretos/PII en claro
- [ ] Un humano recibe las alertas por un canal que realmente mira (WhatsApp/Slack/email), no solo un dashboard que nadie abre

---

## 8. RESILIENCIA Y CONTINUIDAD

- [ ] Single points of failure identificados (¿un solo VPS? ¿una sola persona que sabe operarlo?)
- [ ] Procedimiento ante caída del proveedor de hosting: ¿cómo levanto en otro lado?
- [ ] Dependencias externas con fallback o degradación elegante (WhatsApp API, LLM, pasarela down → no tumba toda la app)
- [ ] Migraciones de BD backward-compatible (deploy nuevo no rompe instancia vieja durante el rollout)
- [ ] Rollback ensayado: tag anterior + `pm2 restart` / `docker compose up` de la versión previa

---

## FORMATO DE REPORTE INFRASTRUCTURE

```
## Infrastructure Audit Report — [Proyecto]
**Servidor:** [proveedor / IP/hostname]
**Componentes:** [resolver según stack detectado / `.sqa/profile.md`]
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
| Runtime `{{RUNTIME}}` | OK/WARN/FAIL | |
| CDN/WAF `{{CDN_WAF}}` | OK/WARN/FAIL | |
| Backups / DR | OK/WARN/FAIL | |
| Monitoreo / Alertas | OK/WARN/FAIL | |
| Resiliencia | OK/WARN/FAIL | |

**Riesgo operacional:** ALTO / MEDIO / BAJO
**Listo para producción:** SÍ / NO — [razón principal]
**RPO/RTO:** [definidos / NO definidos]
```
