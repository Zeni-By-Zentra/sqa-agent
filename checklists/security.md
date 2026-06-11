# Security Audit Checklist — OWASP Top 10:2021 + OWASP API:2023 + ISO 27001:2022

> Auditoría de seguridad de nivel enterprise. Cada sección cruza con `references/breach-database.md`:
> los controles no son arbitrarios, cada uno previene una brecha real documentada.
> Marca cada hallazgo con su tipo de evidencia: **[CMD]** verificado por comando · **[CODE]** verificado leyendo código · **[INF]** inferido.

**Estándares:** OWASP Top 10:2021, OWASP API Security Top 10:2023, OWASP ASVS 4.0, ISO/IEC 27001:2022, NIST SP 800-63B, CIS Benchmarks, NIST SP 800-53.

## Normativa Colombiana de Seguridad

| Norma | Descripción | Requisito |
|-------|-------------|-----------|
| Ley 1581/2012 | Protección de datos personales / Habeas Data | Cifrado, control de acceso, auditoría |
| Circular 007 SIC/2018 | Operadores de datos | Medidas técnicas y administrativas |
| Decreto 1377/2013 | Reglamentación Ley 1581 | Política de tratamiento, registro RNBD |
| Ley 1273/2009 | Delitos informáticos | Penaliza acceso no autorizado y daño informático |
| Resolución 1519/2016 | Entidades públicas | Seguridad en publicación de información |

---

# PARTE A — OWASP TOP 10:2021 (ítem por ítem)

## A01:2021 — Broken Access Control (la #1 más explotada)

- [ ] Toda acción server-side verifica permisos — el control NUNCA vive solo en el frontend (ocultar un botón ≠ autorización)
- [ ] **IDOR:** cada request que recibe un ID verifica pertenencia al usuario/tenant antes de operar (`WHERE id = $1 AND tenant_id = $2`)
- [ ] Multi-tenant: imposible acceder a datos de otro tenant cambiando un parámetro (probar activamente)
- [ ] Negación por defecto: rutas nuevas requieren auth salvo allowlist explícita pública
- [ ] Endpoints de admin protegidos por rol verificado server-side, no solo por ruta "secreta"
- [ ] CORS no usa `Access-Control-Allow-Origin: *` con credenciales; orígenes en allowlist
- [ ] No se exponen IDs secuenciales enumerables para recursos sensibles (usar UUID/nanoid)
- [ ] Path traversal imposible en endpoints que sirven archivos (`../../etc/passwd`)
- [ ] JWT: se verifica firma Y claims (`exp`, `aud`, `iss`); algoritmo fijado server-side (rechazar `alg: none`)
> Precedente: **Optus 2022** (API sin auth + IDs secuenciales → 9.8M registros). **Parler 2021** (IDs secuenciales enumerables).

## A02:2021 — Cryptographic Failures

- [ ] TLS 1.2+ en TODOS los endpoints; HSTS con `includeSubDomains; preload`
- [ ] Passwords con **bcrypt (cost≥12)**, **argon2id** o **scrypt** — NUNCA MD5/SHA1/SHA256 a secas
- [ ] Datos sensibles cifrados en reposo (AES-256-GCM); backups cifrados
- [ ] Sin datos sensibles en URLs/query strings (quedan en logs de Nginx y Cloudflare)
- [ ] Tokens y secretos generados con CSPRNG (`crypto.randomBytes`), nunca `Math.random()`
- [ ] Sin algoritmos rotos: DES, 3DES, RC4, MD5, SHA1 para firmas, ECB mode
- [ ] IV/nonce únicos por operación; AEAD (GCM/ChaCha20-Poly1305) sobre CBC sin MAC
- [ ] Certificados válidos, no autofirmados en producción, renovación automatizada
- [ ] API keys en variables de entorno o secrets manager; CERO secrets en código o commits
- [ ] Rotación periódica de credenciales privilegiadas (≤90 días)
> Precedente: **Heartbleed 2014** (CVE-2014-0160). **Adobe 2013** (passwords con 3DES-ECB → 153M). **LinkedIn 2012** (SHA1 sin sal → 117M).

## A03:2021 — Injection

- [ ] **SQL:** SIEMPRE queries parametrizadas (`$1, $2` en node-postgres) — CERO concatenación de strings en SQL
- [ ] Verificar activamente: `grep -rn "query(\`.*\${" src/` y similares
- [ ] ORM/query builder no se usa en modo raw concatenado
- [ ] **XSS:** salida escapada por defecto (React lo hace); auditar todo `dangerouslySetInnerHTML` y `innerHTML`
- [ ] Sin `eval()`, `Function()`, `setTimeout(string)`, `child_process.exec` con input de usuario
- [ ] **Command injection:** usar `execFile`/`spawn` con array de args, nunca `exec` con string interpolado
- [ ] Validación de entrada con allowlist (tipo, rango, longitud, formato) en el borde
- [ ] **NoSQL / LDAP / template injection** evaluados si el stack los usa
- [ ] Headers de respuesta sanitizados (CRLF injection / response splitting)
> Precedente: **Equifax 2017** (CVE-2017-5638, Struts RCE → 147M). **Heartland 2008** (SQLi → 130M tarjetas). **TalkTalk 2015** (SQLi).

## A04:2021 — Insecure Design

- [ ] Existe threat model de los flujos críticos (ver pre-dev-planning §2.4)
- [ ] Límites de negocio aplicados: máximo de intentos, cuotas, límites de monto/cantidad
- [ ] Flujos de recuperación de cuenta no permiten enumeración ni bypass de MFA
- [ ] No se confía en datos del cliente para decisiones de seguridad o precio (recalcular server-side)
- [ ] Rate limiting de diseño en operaciones costosas (envío de SMS/email, generación con LLM)
> Precedente: robos de millas/puntos por lógica de negocio; fraude de cupones por validación client-side.

## A05:2021 — Security Misconfiguration

- [ ] Sin credenciales/cuentas por defecto activas
- [ ] Mensajes de error de producción genéricos — sin stack traces, rutas ni versiones expuestas
- [ ] Headers de seguridad presentes: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`/`frame-ancestors`, `Referrer-Policy`, `Strict-Transport-Security`, `Permissions-Policy`
- [ ] Directory listing deshabilitado en Nginx; archivos `.env`, `.git`, `.bak` no servibles
- [ ] Swagger/OpenAPI docs deshabilitados o autenticados en producción
- [ ] `NODE_ENV=production`; source maps no expuestos públicamente
- [ ] Servicios de admin (Adminer, n8n UI, Chatwoot) no expuestos a internet sin auth/VPN
- [ ] Puertos innecesarios cerrados; bind a `127.0.0.1` para servicios internos (no `0.0.0.0`)
> Precedente: incontables buckets/Elasticsearch/Mongo abiertos (Exactis 340M). **Tesla 2018** (consola Kubernetes sin password → cryptojacking).

## A06:2021 — Vulnerable & Outdated Components (ver Parte C)

- [ ] `npm audit` / `yarn audit` sin vulnerabilidades críticas/altas sin mitigar
- [ ] Lockfile commiteado y respetado en CI (`npm ci`, no `npm install`)
- [ ] Imágenes Docker escaneadas (Trivy/Grype); base images pinneadas por versión
- [ ] Inventario de dependencias directas y transitivas; SBOM si el cliente lo requiere
- [ ] Proceso de actualización: dependabot/renovate activo
> Precedente: **Log4Shell 2021** (CVE-2021-44228, CVSS 10.0). **Equifax 2017** (Struts sin parchar 2 meses). **event-stream 2018** (dependencia npm troyanizada).

## A07:2021 — Identification & Authentication Failures

- [ ] MFA para cuentas privilegiadas y admin (ISO 27001 A.9.4.2)
- [ ] Passwords ≥12 chars; check contra listas de filtradas (HIBP); sin reglas de composición absurdas ni rotación forzada (NIST SP 800-63B §5.1.1)
- [ ] Bloqueo tras 5-10 intentos + backoff exponencial o CAPTCHA
- [ ] Tokens de sesión con CSPRNG, ≥128 bits
- [ ] Regeneración de session ID tras login (anti session fixation)
- [ ] Invalidación de sesión en logout — SERVER-SIDE, no solo borrar cookie en cliente
- [ ] Timeout de sesión inactiva (15-30 min en apps sensibles)
- [ ] Cookies de sesión `HttpOnly; Secure; SameSite=Lax|Strict`
- [ ] Notificación de logins desde nuevos dispositivos
- [ ] Protección contra credential stuffing (rate limit por IP + por cuenta + detección de patrones)
> Precedente: **Colonial Pipeline 2021** (VPN sin MFA + password filtrada → paró el combustible de la costa este de EEUU). **Uber 2022** (MFA fatigue / push bombing).

## A08:2021 — Software & Data Integrity Failures

- [ ] Dependencias de fuentes confiables; integridad verificada (lockfile con hashes)
- [ ] CI/CD: acciones de terceros pinneadas por SHA, no por tag mutable
- [ ] Updates/auto-updates firmados y verificados
- [ ] Deserialización: no deserializar datos no confiables sin validación
- [ ] Webhooks entrantes con firma HMAC verificada + protección anti-replay (timestamp)
> Precedente: **SolarWinds 2020** (build pipeline comprometido → backdoor SUNBURST firmado → 18.000 orgs, agencias federales). **Codecov 2021** (script CI alterado → robo de secrets). **3CX 2023** (cadena de suministro).

## A09:2021 — Security Logging & Monitoring Failures

- [ ] Logs de auth (éxito/fallo) con timestamp, IP, user-agent, user id
- [ ] Logs de cambios en datos críticos y acciones de admin (audit trail)
- [ ] Logs SIN passwords, tokens, PII en claro (enmascarar)
- [ ] Alertas para anomalías (picos de 401/403, fuerza bruta, errores 5xx)
- [ ] Retención ≥90 días; logs centralizados y protegidos contra borrado por el atacante
- [ ] Integración con Sentry (errores) + uptime (UptimeRobot)
- [ ] Existe forma de detectar una brecha en curso (no enterarse 6 meses después)
> Precedente: **Marriott/Starwood 2018** (brecha sin detectar ~4 años, 383M). **Target 2013** (alertas FireEye ignoradas).

## A10:2021 — Server-Side Request Forgery (SSRF)

- [ ] Toda URL provista por el usuario que el servidor fetchea pasa por allowlist de dominios/IPs
- [ ] Bloqueo de rangos internos: `127.0.0.0/8`, `10/8`, `172.16/12`, `192.168/16`, `169.254.169.254` (metadata cloud), IPv6 equivalentes
- [ ] Redirecciones no saltan la validación (validar tras cada redirect)
- [ ] Funciones de "import desde URL", avatar por URL, webhooks salientes, previews de link → todas validadas
- [ ] Metadata endpoint del proveedor cloud no alcanzable desde la app
> Precedente: **Capital One 2019** (SSRF → robo de credenciales del metadata service de AWS EC2 → 100M registros, multa $80M).

---

# PARTE B — VECTORES ESPECÍFICOS DEL STACK ZENTRA

> Aplicar SOLO las secciones de tecnologías presentes en el proyecto.

## B1. Next.js (14+ / 16)

- [ ] **Server/Client boundary:** secretos y lógica sensible SOLO en Server Components / Server Actions / route handlers — nunca en `"use client"`
- [ ] Variables `NEXT_PUBLIC_*` NO contienen secretos (se hornean en el bundle y son públicas)
- [ ] **CVE-2025-29927 (middleware bypass):** versión parcheada — un atacante podía saltarse el middleware (incl. auth) con el header `x-middleware-subrequest`. Verificar Next >= 14.2.25 / 15.2.3 o línea parcheada
- [ ] Server Actions: validar input y re-verificar autorización DENTRO de la action (son endpoints públicos)
- [ ] No confiar en el middleware como ÚNICA capa de auth — re-verificar en el data layer
- [ ] `images.remotePatterns` restrictivo (el optimizador de imágenes puede ser vector SSRF/DoS)
- [ ] API routes con rate limiting (no lo traen por defecto)
- [ ] Sin secretos en `next.config.js` que se filtren al cliente

## B2. Node.js / Express

- [ ] `helmet` o headers de seguridad equivalentes configurados
- [ ] Límite de tamaño de body (`express.json({ limit: '100kb' })`) — anti-DoS
- [ ] `express-rate-limit` o equivalente en endpoints públicos y de auth
- [ ] Sin prototype pollution: validar/sanitizar objetos de entrada (`__proto__`, `constructor`)
- [ ] CORS configurado con allowlist explícita, no `cors()` abierto
- [ ] Manejo de errores que no filtra stack traces
- [ ] Sin `child_process.exec` con input de usuario (usar `execFile` con args)
- [ ] Dependencias de auth (`jsonwebtoken`, `passport`) actualizadas y bien configuradas

## B3. PostgreSQL (node-postgres directo)

- [ ] **100% queries parametrizadas** — auditar cada `.query()`: ningún `${}` ni `+` construyendo SQL
- [ ] Usuario de aplicación con permisos mínimos (no `postgres`, no superuser, no owner del schema)
- [ ] `ssl: { rejectUnauthorized: true }` si la BD es remota
- [ ] Connection pooling con límites (`max`) para no agotar conexiones
- [ ] `statement_timeout` configurado (anti query-bomb / DoS)
- [ ] Row Level Security considerada para aislamiento multi-tenant fuerte
- [ ] Backups automatizados + cifrados + restore probado
- [ ] BD no escuchando en `0.0.0.0` expuesta a internet (bind interno o firewall)
- [ ] `pg_hba.conf` no usa `trust`; requiere password/cert

## B4. Docker / Docker Compose

- [ ] Contenedores NO corren como root (`USER appuser`) — CIS Docker 4.1
- [ ] Imágenes base pinneadas y escaneadas (Trivy); multi-stage para imagen mínima
- [ ] Sin secretos en `ENV`/`ARG` de capas (quedan en el historial) — verificar `docker history`
- [ ] Sin `--privileged`, sin caps innecesarios; `cap_drop: ALL` + caps mínimos
- [ ] `no-new-privileges:true`; filesystem `read_only` donde se pueda + `tmpfs` para writable
- [ ] Docker socket (`/var/run/docker.sock`) NUNCA montado en contenedores expuestos (= root del host)
- [ ] Resource limits (memory, cpu) definidos — anti DoS por contenedor
- [ ] Red segmentada: servicios internos en red sin exponer puertos al host
- [ ] `.dockerignore` excluye `.env`, `.git`, `node_modules`
> Precedente: **Tesla 2018** (consola de Kubernetes sin password → cryptojacking en su infra cloud).

## B5. Nginx (reverse proxy)

- [ ] `server_tokens off;` (no exponer versión)
- [ ] TLS 1.2+ con ciphers modernos; OCSP stapling; HSTS
- [ ] Headers de seguridad inyectados con `always` (CSP, X-Frame-Options, etc.)
- [ ] `client_max_body_size` limitado por endpoint
- [ ] Rate limiting (`limit_req_zone`) en login y endpoints sensibles
- [ ] No servir dot-files: `location ~ /\. { deny all; }` (bloquea `.git`, `.env`)
- [ ] `proxy_pass` a `127.0.0.1`/red interna, no a servicios expuestos
- [ ] Si está detrás de Cloudflare: `real_ip` configurado solo desde rangos de Cloudflare (no confiar en `X-Forwarded-For` arbitrario para rate limit)

## B6. PM2

- [ ] PM2 corre como usuario sin privilegios (patrón Zeni: usuario `zeni`, NO root)
- [ ] Secrets vía `secrets.env` cargado, no hardcodeados en `ecosystem.config.js`
- [ ] `pm2 save` tras cambios; `pm2 startup` configurado para reinicio tras reboot
- [ ] Logs rotados (`pm2-logrotate`) — no llenar el disco
- [ ] App nunca expuesta directamente; siempre detrás de Nginx
- [ ] `max_memory_restart` configurado (anti memory leak tumbando el server)

## B7. Chatwoot (Ruby on Rails self-hosted)

- [ ] Versión actualizada (Rails y Chatwoot tienen CVEs periódicos)
- [ ] `SECRET_KEY_BASE` / `RAILS_MASTER_KEY` fuertes y fuera de git
- [ ] Postgres y Redis de Chatwoot NO expuestos a internet
- [ ] Panel solo por HTTPS detrás de Nginx; IP allowlist para super admin recomendable
- [ ] Webhooks de Chatwoot firmados/verificados
- [ ] CSP correcto si se embebe el widget (gotcha conocido del proyecto Zeni)
- [ ] Uploads de adjuntos limitados en tamaño y tipo
- [ ] Runbook de restart documentado (`docker restart chatwoot_app` ante 502)

## B8. n8n (workflow automation)

- [ ] UI de n8n NUNCA expuesta a internet sin auth (`N8N_BASIC_AUTH_ACTIVE=true` o detrás de SSO/VPN) — n8n ejecuta código, UI abierta = RCE
- [ ] Credenciales cifradas (`N8N_ENCRYPTION_KEY` set y respaldado)
- [ ] Webhooks de n8n con auth/firma — no dejar webhooks abiertos que disparen workflows sensibles
- [ ] Nodos "Execute Command" / "Code" auditados — ejecutan comandos arbitrarios
- [ ] `N8N_SECURE_COOKIE=true`; detrás de HTTPS
- [ ] Workflows con secretos usan credenciales de n8n, no valores hardcodeados en nodos
- [ ] Instancia actualizada (n8n parchea CVEs con regularidad)

## B9. Cloudflare / Hetzner VPS

- [ ] Cloudflare en modo "proxied" (naranja) para ocultar IP de origen
- [ ] Cert de origen Cloudflare + SSL mode "Full (Strict)" — NO "Flexible" (deja tráfico en claro al origen)
- [ ] Firewall del origen solo acepta 80/443 desde rangos de Cloudflare (si no, el atacante saltea el WAF yendo a la IP directa)
- [ ] WAF rules / rate limiting de Cloudflare activos
- [ ] Hetzner: SSH solo por llave pública, root login deshabilitado
- [ ] `ufw`/firewall: deny por defecto, solo puertos necesarios
- [ ] `fail2ban` activo para SSH y login
- [ ] Actualizaciones de seguridad del OS automatizadas (`unattended-upgrades`)
- [ ] Snapshots/backups del VPS configurados

---

# PARTE C — ESCANEO DE DEPENDENCIAS Y SUPPLY CHAIN

- [ ] **Comando base:** `npm audit --omit=dev` — reportar críticas y altas
- [ ] `npm audit fix` aplicado donde no rompe; las que requieren major bump → `[MANUAL]`
- [ ] Lockfile presente y commiteado; CI usa `npm ci` (instala exacto del lock)
- [ ] Dependabot o Renovate activo con auto-PR de patches de seguridad
- [ ] **Imágenes Docker:** `trivy image <imagen>` o `grype` en CI — bloquear deploy con críticas
- [ ] **Secrets en código/historial:** `gitleaks detect --source . -v` y `git log --all --full-history -- .env`
- [ ] Dependencias abandonadas (sin releases >2 años) marcadas para reemplazo
- [ ] Typosquatting: revisar nombres sospechosos al instalar paquetes nuevos
- [ ] `package.json` sin `postinstall` scripts no auditados de dependencias dudosas
- [ ] SBOM generable (`npm sbom` / Syft) si el cliente lo exige

**Comandos de verificación rápida (ejecutar y reportar resultado real):**
```bash
npm audit --omit=dev --json | jq '.metadata.vulnerabilities'   # conteo por severidad
gitleaks detect --source . --no-banner 2>&1 | tail -20         # secrets en repo
git log --all --full-history --oneline -- '*.env' | head       # .env commiteado alguna vez
grep -rnE 'query\(\s*[`"'"'"'].*\$\{' src/ 2>/dev/null          # posible SQLi por interpolación
grep -rnE 'eval\(|new Function\(|dangerouslySetInnerHTML|innerHTML\s*=' src/ 2>/dev/null
docker history <imagen> --no-trunc | grep -iE 'PASSWORD|SECRET|KEY|TOKEN'  # secrets en capas
```

---

# PARTE D — CHECKLIST DE PEN-TEST (modo --pentest)

> SOLO sobre sistemas propios o con AUTORIZACIÓN ESCRITA. No ejecutar explotación destructiva sin confirmación por comando.

### Fase 1 — Recon
- [ ] Subdominios (`subfinder`, CT logs vía crt.sh), puertos (`nmap -sV`), tecnologías (`whatweb`, headers)
- [ ] Endpoints y rutas (`gobuster`/`ffuf`), archivos expuestos (`.git`, `.env`, `/backup`)
- [ ] ¿La IP de origen es alcanzable saltando Cloudflare?

### Fase 2 — Autenticación
- [ ] Fuerza bruta / credential stuffing (¿hay lockout y rate limit?)
- [ ] Enumeración de usuarios (¿el login revela si el email existe?)
- [ ] Recuperación de password (¿token predecible? ¿no expira? ¿permite bypass de MFA?)
- [ ] JWT: probar `alg: none`, firma con clave débil, claims manipulados

### Fase 3 — Autorización / IDOR
- [ ] Cambiar IDs en URLs y bodies → ¿accedo a datos de otro usuario/tenant?
- [ ] Escalada horizontal y vertical (usuario normal → acciones de admin)
- [ ] Forced browsing a rutas de admin sin sesión privilegiada

### Fase 4 — Inyección y entrada
- [ ] SQLi (`sqlmap` en endpoints autorizados), XSS reflejado/almacenado/DOM, command injection
- [ ] SSRF en funciones que fetchean URLs; XXE si parsea XML; template injection
- [ ] Upload de archivos: tipo/tamaño/path, ¿se puede subir un ejecutable?

### Fase 5 — Lógica de negocio
- [ ] Manipulación de precio/cantidad/montos en el cliente
- [ ] Race conditions (doble gasto, doble canje de cupón) con requests concurrentes
- [ ] Saltarse pasos de un flujo multi-step (pagar sin completar verificación)

### Fase 6 — Infra / configuración
- [ ] Headers de seguridad, TLS (`testssl.sh`), servicios de admin expuestos (n8n, Adminer, Chatwoot)
- [ ] Mensajes de error con stack traces; versiones expuestas; default creds en servicios

---

## GESTIÓN DE INCIDENTES (ISO 27001 A.16)

- [ ] Procedimiento documentado de respuesta a incidentes
- [ ] Canal de reporte de vulnerabilidades (security@dominio)
- [ ] Contactos de emergencia actualizados
- [ ] Plan de recuperación testeado periódicamente
- [ ] Runbook: qué hacer si se filtra un secreto (rotar, revocar, auditar accesos)

---

## FORMATO DE REPORTE DE SEGURIDAD

```
## Security Audit Report — [Aplicación]
**Estándares:** OWASP Top 10:2021, OWASP API:2023, ISO/IEC 27001:2022, NIST SP 800-63B
**Stack auditado:** [Next.js / Node / PostgreSQL / Docker / Nginx / PM2 / Chatwoot / n8n / Cloudflare]
**Alcance:** [endpoints/módulos/infra revisados]
**Método:** [code-read / comandos ejecutados / pentest autorizado]

### Hallazgos Críticos 🔴   (acción inmediata, bloquean deploy)
[con CVSS estimado, precedente histórico, explotabilidad y corrección]

### Hallazgos Importantes 🟡   (próximo sprint)

### Sugerencias 🟢

### Puntuación de Seguridad
| Categoría | Estado | Evidencia |
|-----------|--------|-----------|
| A01 Access Control | OK/WARN/FAIL | CMD/CODE/INF |
| A02 Cryptographic | OK/WARN/FAIL | |
| A03 Injection | OK/WARN/FAIL | |
| A04 Insecure Design | OK/WARN/FAIL | |
| A05 Misconfiguration | OK/WARN/FAIL | |
| A06 Vulnerable Components | OK/WARN/FAIL | |
| A07 Auth Failures | OK/WARN/FAIL | |
| A08 Integrity Failures | OK/WARN/FAIL | |
| A09 Logging/Monitoring | OK/WARN/FAIL | |
| A10 SSRF | OK/WARN/FAIL | |
| Stack-specific (B1-B9) | OK/WARN/FAIL | |
| Dependencies / Supply chain | OK/WARN/FAIL | |

**Nivel de riesgo global:** CRÍTICO / ALTO / MEDIO / BAJO
**Veredicto de deploy:** ✅ APROBADO / ⛔ BLOQUEADO — [razón]

---

## PARTE E — INYECCIÓN EN MODELOS DE LENGUAJE (OWASP LLM Top 10:2025)

Aplica cuando la aplicación integra un LLM (OpenAI, Anthropic, Gemini, modelos locales).

- [ ] Los inputs del usuario **no se concatenan directamente** al system prompt o historial sin sanitización (LLM01 — Prompt Injection) `[CMD: grep -rn "systemPrompt.*req\|prompt.*\${" src/]`
- [ ] Existe sanitización que elimina tokens de sistema (`<s>`, `[INST]`, `###`), prefijos de rol (`Assistant:`, `System:`, `Human:`), XML de instrucciones (`<instructions>`, `<|im_start|>`) y code fences anidados
- [ ] Los outputs del LLM se tratan como datos no confiables: no se ejecutan como código (`eval`), no se inyectan en SQL/HTML sin escape
- [ ] El system prompt **no contiene secretos recuperables** (API keys, contraseñas, PII de otros usuarios) (LLM02 — Sensitive Info Disclosure)
- [ ] Longitud máxima del input del usuario está limitada (previene desbordamiento de contexto y ataques de exfiltración por acumulación)
- [ ] El historial de conversación tiene límite de mensajes o tokens (evita que conversaciones largas filtren contexto de otras sesiones)
- [ ] Las respuestas del LLM que van a la UI se escapan para HTML (previene XSS indirecto vía LLM)
- [ ] El system prompt completo **no se expone** al usuario ni en mensajes de error (LLM07 — System Prompt Leakage)
- [ ] Si el LLM puede llamar herramientas/funciones: cada tool valida sus propios argumentos, no confía ciegamente en los parámetros generados por el modelo (LLM06 — Excessive Agency)
- [ ] Los logs de conversación no persisten PII sin enmascarar (Ley 1581/2012 Art. 17 + LLM02)
> Precedente: **Samsung 2023** (empleados filtraron código fuente a ChatGPT via system prompt). **Prompt injection en Bing Chat 2023** (instrucciones ocultas en páginas web manipulaban al modelo).
```
