# Stack Profile — Default (auto-detección)

> Perfil genérico, agnóstico de proveedor. Es el que se usa cuando el proyecto NO trae
> un `.sqa/profile.md` propio. El agente detecta el stack real leyendo los archivos del
> repositorio y aplica solo las secciones de checklist que correspondan.

## Cómo se detecta el stack

| Señal en el repo | Componente inferido | Checklist relevante |
|------------------|---------------------|---------------------|
| `package.json` (deps: next) | Next.js / Node web app | security, performance, a11y |
| `package.json` (deps: react/vue/svelte) | SPA frontend | a11y, performance |
| `package.json` (deps: express/fastify/nest) | API Node | security, api-quality |
| `requirements.txt` / `pyproject.toml` | Python (Django/Flask/FastAPI) | security, api-quality |
| `composer.json` | PHP (Laravel/WordPress) | security |
| `go.mod` / `Cargo.toml` / `pom.xml` | Go / Rust / Java | security, api-quality |
| `Dockerfile` / `compose.y*ml` | Contenedores | infrastructure (§Docker) |
| `*.tf` / `*.bicep` / `cloudformation` | IaC | infrastructure (§Cloud) |
| `nginx*.conf` / `Caddyfile` | Reverse proxy | infrastructure (§Proxy) |
| `.github/workflows/` / `.gitlab-ci.yml` | CI/CD | devops |
| `prisma/` / `*.sql` / `drizzle` | Base de datos | database, data-quality |

## Reglas de aplicación

1. **No asumas proveedores.** Donde el checklist mencione un proveedor concreto (un VPS,
   un CDN, una herramienta de orquestación), trátalo como **ejemplo**, no como requisito.
   Evalúa el control subyacente contra lo que el proyecto realmente use.
2. **No reportes secciones que no apliquen.** Si no hay contenedores, omite Docker. Si no
   hay reverse proxy, omite esa sección — no lo marques como FAIL.
3. **Hosting/PaaS gestionado** (Vercel, Netlify, Render, Fly, Railway, Cloud Run, App Runner):
   gran parte del hardening de SO/red lo cubre el proveedor. Enfócate en config de la app,
   variables de entorno, dominios, y los controles que SIGUEN siendo responsabilidad del equipo.
4. **Cloud (AWS/GCP/Azure):** aplica la sección Cloud de `infrastructure.md` (IAM, grupos de
   seguridad, storage público, secretos, logging) en vez de la de VPS.

## Variables del perfil (placeholders que el checklist resuelve)

| Variable | Default | Significado |
|----------|---------|-------------|
| `{{RUNTIME}}` | auto | Proceso de la app (PaaS / contenedor / gestor de procesos / systemd) |
| `{{PROXY}}` | auto | Reverse proxy o ingress frente a la app |
| `{{CDN_WAF}}` | auto | CDN/WAF si existe |
| `{{SECRETS_STORE}}` | env vars del runtime | Dónde viven los secretos |
| `{{BACKUP_TARGET}}` | off-site cifrado | Destino de backups |
| `{{DB}}` | auto | Motor de base de datos |
| `{{PRIV_USER}}` | usuario sin privilegios dedicado | Usuario bajo el que corre la app |

Si el proyecto define un `.sqa/profile.md`, sus valores **sobrescriben** estos defaults.
