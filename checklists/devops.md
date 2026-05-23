# DevOps Quality Checklist — CI/CD + Docker + Infrastructure + Web Server

## 1. PIPELINE CI/CD (ISO/IEC 25010 §4.6 — Maintainability)

### 1.1 Calidad del Pipeline
- [ ] El pipeline corre en cada push a cualquier rama y en cada PR
- [ ] Fast fail: lint primero → tests unitarios → build → security scan → tests integración → deploy
- [ ] El tiempo total del pipeline es < 10 minutos para ciclo rápido de feedback
- [ ] Los stages están claramente separados y nombrados
- [ ] El pipeline es reproducible localmente con los mismos comandos (no "solo funciona en CI")
- [ ] Los secrets se inyectan como variables de entorno desde el secret manager — CERO en el código o en el YAML del pipeline

### 1.2 Calidad de Builds
- [ ] Los builds son reproducibles y deterministas (mismo código = mismo artifact)
- [ ] Los artifacts están versionados con git tag o commit SHA (no `latest` para producción)
- [ ] Los builds no dependen de estado local del runner (sin cache implícito no declarado)
- [ ] Los artifacts se archivan y son trazables a su commit origen

### 1.3 Gates de Calidad (Quality Gates)
- [ ] Merge a main/master bloqueado si los tests fallan
- [ ] Merge bloqueado si el security scan encuentra CVEs críticos (CVSS ≥ 7.0)
- [ ] La cobertura de tests no puede caer más de un umbral definido (ej: -5%)
- [ ] Code review aprobado por mínimo 1 reviewer antes de merge
- [ ] Lint y format automático pasan sin errores
- [ ] Sin secrets detectados en el código (git-secrets, gitleaks, trufflehog en CI)

### 1.4 Deployment Automation
- [ ] Los deploys a producción son automatizados (no SSH manual en producción)
- [ ] Solo se despliega a producción lo que pasó todos los gates anteriores
- [ ] El deploy se puede pausar o cancelar en cualquier punto
- [ ] Existe aprobación manual para deploys a producción si el equipo lo requiere

---

## 2. DOCKER Y CONTENEDORES

### 2.1 Dockerfile Best Practices
- [ ] Multi-stage build: stage de build separado del stage de runtime (imagen final liviana)
- [ ] Base image con versión específica y pinneada (`node:20.11-alpine`, no `node:latest`)
- [ ] El proceso NO corre como root — `USER appuser` antes del `CMD`/`ENTRYPOINT`
- [ ] `.dockerignore` excluye: `node_modules`, `.env`, `.git`, `dist` local, `*.log`, `.next` local (para Next.js usar standalone)
- [ ] Instrucciones ordenadas de menos a más cambiante (`COPY package.json` antes de `COPY . .`)
- [ ] Sin secrets en capas del Dockerfile (`ENV`, `ARG` con valores sensibles)
- [ ] `HEALTHCHECK` definido para que el orchestrator detecte el estado real de la app
- [ ] Imagen final es la mínima necesaria (Alpine o Distroless cuando sea posible)

### 2.2 Seguridad de Contenedores
- [ ] Imagen escaneada en CI con **Trivy** o **Grype** antes de cada deploy
- [ ] 0 CVEs críticos aceptados para deploy a producción
- [ ] Sin paquetes innecesarios en imagen final
- [ ] Filesystem de la imagen es read-only donde sea posible
- [ ] Sin comunicación entre contenedores que no la necesiten (network segmentation)
- [ ] Los contenedores no tienen privilegios elevados (sin `--privileged`)

### 2.3 Docker Compose y Orquestación
- [ ] Healthchecks definidos en `docker-compose.yml` para cada servicio
- [ ] Las dependencias usan `depends_on` con `condition: service_healthy`
- [ ] Los volúmenes para datos persistentes están declarados explícitamente
- [ ] Los recursos (memory limits, CPU limits) están definidos para producción
- [ ] Las variables de entorno sensibles vienen de `.env` files o secret manager (no hardcodeadas en compose)
- [ ] El compose de producción es diferente al de desarrollo (sin bind mounts de código fuente en prod)

---

## 3. GESTIÓN DE CONFIGURACIÓN Y SECRETOS

### 3.1 Separación de Config por Ambiente
- [ ] Config separada por ambiente: development, staging, production
- [ ] Sin valores de producción en archivos de config de desarrollo
- [ ] Las URLs, hosts y puertos son configurables por variable de entorno
- [ ] El código no tiene `if (NODE_ENV === 'production')` para comportamientos críticos de seguridad

### 3.2 Manejo de Secretos (Crítico)
- [ ] CERO secrets en el repositorio de código — incluyendo el historial de git completo
- [ ] Verificar historial: `git log --all --full-history -- .env` no debe mostrar .env commiteado
- [ ] `.env` en `.gitignore` con verificación activa
- [ ] Secrets en variables de entorno del CI/CD (GitHub Actions Secrets, GitLab CI Variables)
- [ ] Rotación de secretos posible sin downtime
- [ ] Acceso a secrets auditado (quién accede a qué, cuándo)
- [ ] Secrets de staging son DIFERENTES a los de producción (nunca compartir)
- [ ] API keys expuestas son rotadas inmediatamente

---

## 4. DEPLOYMENT Y ROLLBACK

### 4.1 Estrategia de Deployment
- [ ] La estrategia está documentada: blue/green, canary, rolling update, o in-place
- [ ] Los deploys son zero-downtime (no requieren maintenance window para cambios normales)
- [ ] Las migraciones de DB son backward-compatible con la versión anterior del código
- [ ] Los static assets tienen cache-busting (hash en nombre de archivo o query param)

### 4.2 Rollback
- [ ] El rollback está automatizado o tiene runbook de < 5 pasos
- [ ] El tiempo de rollback es < 10 minutos
- [ ] Las migraciones de DB tienen `down` migration para rollback completo
- [ ] El equipo sabe cómo hacer rollback (no solo el tech lead)

### 4.3 Verificación Post-Deploy
- [ ] Smoke tests automáticos inmediatamente después del deploy
- [ ] Las métricas de error rate y latencia se monitorean automáticamente post-deploy
- [ ] Si el error rate sube X% tras el deploy, se dispara alerta automática
- [ ] El equipo tiene visibility del estado del deploy en tiempo real

---

## 5. SERVIDOR E INFRAESTRUCTURA

### 5.1 Hardening del Servidor
- [ ] Actualizaciones de seguridad automáticas o proceso de patching documentado con SLA
- [ ] SSH: solo autenticación por llave pública, `PasswordAuthentication no`, `PermitRootLogin no`
- [ ] Firewall configurado con mínimo privilegio: solo puertos necesarios abiertos (80, 443, y SSH desde IPs específicas)
- [ ] Fail2ban o equivalente activo contra fuerza bruta en SSH y servicios expuestos
- [ ] Swap configurado apropiadamente para prevenir OOM kills inesperados
- [ ] Usuarios del sistema con mínimo privilegio (la app no corre como root en el servidor)

### 5.2 Monitoreo y Observabilidad
- [ ] Métricas de aplicación monitoreadas: error rate, latencia P50/P95/P99, throughput
- [ ] Métricas de infra monitoreadas: CPU, RAM, disco (alerta en >80%), red
- [ ] Logs centralizados y consultables sin acceder al servidor directamente
- [ ] Alertas configuradas para:
  - Servidor caído (uptime monitoring externo)
  - Error rate > umbral definido
  - Disco > 80%
  - RAM > 90%
  - Certificado SSL expira en < 30 días
- [ ] Uptime monitoring externo desde ubicación diferente al servidor (Better Uptime, UptimeRobot)
- [ ] Runbook documentado para cada alerta crítica (qué hacer cuando suena)

### 5.3 Backup y Disaster Recovery
- [ ] Backups automáticos de DB con frecuencia apropiada a criticidad (hourly/daily)
- [ ] Restore de backup probado en los últimos 30 días con datos reales
- [ ] Backups en ubicación geográfica diferente al servidor principal
- [ ] Backups cifrados si contienen datos personales (Ley 1581/2012 Colombia)
- [ ] RTO (Recovery Time Objective) y RPO (Recovery Point Objective) definidos y documentados
- [ ] Procedimiento de disaster recovery documentado paso a paso y conocido por el equipo

---

## 6. NGINX Y WEB SERVER

- [ ] HTTPS forzado con redirect 301 permanente de HTTP a HTTPS
- [ ] HSTS configurado: `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- [ ] Headers de seguridad presentes en todas las respuestas:
  - `X-Frame-Options: DENY` o `SAMEORIGIN`
  - `X-Content-Type-Options: nosniff`
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Permissions-Policy` para features no usadas
- [ ] Versión del servidor no expuesta en headers (`server_tokens off` en nginx)
- [ ] Rate limiting configurado a nivel nginx para endpoints críticos (login, API)
- [ ] Logs de acceso y error con retención apropiada y rotación
- [ ] Gzip/Brotli activo para assets de texto (JS, CSS, HTML, JSON)
- [ ] Cache de assets estáticos correctamente configurado:
  - Assets con hash en nombre: `Cache-Control: public, max-age=31536000, immutable`
  - HTML y API responses: `Cache-Control: no-store` o corto (60s)
- [ ] Timeouts configurados: `client_body_timeout`, `client_header_timeout`, `proxy_read_timeout`
- [ ] Client max body size configurado explícitamente para prevenir ataques de payload grande
- [ ] SSL/TLS: TLS 1.2+ únicamente, ciphers seguros (sin RC4, sin 3DES, sin SSLv3)
- [ ] Certificados SSL con renovación automática (certbot con cron, o Let's Encrypt con auto-renewal)

---

## FORMATO DE REPORTE DEVOPS REVIEW

```
## DevOps Quality Report — [Proyecto]
**Stack:** Docker + [GitHub Actions / GitLab CI] + [VPS / Cloud]
**Servidor:** [IP/hostname]
**Revisado:** [fecha]

### Pipeline CI/CD
🔴/🟡/🟢 [hallazgo]

### Docker / Contenedores
🔴/🟡/🟢 [hallazgo]

### Infraestructura y Servidor
🔴/🟡/🟢 [hallazgo]

### Nginx / Web Server
🔴/🟡/🟢 [hallazgo]

### Puntuación
| Área | Estado |
|------|--------|
| Pipeline CI/CD | OK/WARN/FAIL |
| Docker y contenedores | OK/WARN/FAIL |
| Secrets management | OK/WARN/FAIL |
| Deployment / Rollback | OK/WARN/FAIL |
| Monitoreo y alertas | OK/WARN/FAIL |
| Backup y DR | OK/WARN/FAIL |
| Nginx / Web server | OK/WARN/FAIL |
| Hardening de servidor | OK/WARN/FAIL |

**Riesgo operacional:** ALTO / MEDIO / BAJO
**Listo para producción:** SÍ / NO — [razón principal si es NO]
```
