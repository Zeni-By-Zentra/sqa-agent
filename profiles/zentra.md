# Stack Profile — Zentra

> Perfil concreto del stack Zentra/Zeni. Cópialo a `.sqa/profile.md` en un proyecto Zentra
> (o referéncialo con `--profile zentra`) para activar los gotchas y rutas específicas.
> Para proyectos de terceros, usa `profiles/default.md` (auto-detección).

## Resolución de variables

| Variable | Valor Zentra |
|----------|--------------|
| `{{RUNTIME}}` | PM2 (apps Node) + Docker Compose (servicios) |
| `{{PROXY}}` | Nginx reverse proxy |
| `{{CDN_WAF}}` | Cloudflare (proxied, Full Strict + cert de origen) |
| `{{SECRETS_STORE}}` | `secrets.env` (source antes de `pm2 restart --update-env`) |
| `{{BACKUP_TARGET}}` | Cloudflare R2 cifrado off-site + snapshots Hetzner |
| `{{DB}}` | PostgreSQL |
| `{{PRIV_USER}}` | usuario `zeni` (NO root) |
| `{{HOST}}` | Hetzner VPS |

## Stack de referencia

**Frontend/app:** Next.js 14+ App Router + TypeScript
**Backend:** Next.js API routes (monolito) o Node + Express
**Jobs/automatización:** n8n (workflows de negocio), node-cron/PM2 (jobs de app)
**Mensajería cliente:** Chatwoot self-hosted + WhatsApp Cloud API
**Deploy:** Docker Compose / PM2 en Hetzner VPS, Nginx reverse proxy
**DNS/CDN/WAF:** Cloudflare (proxied)
**Monitoreo:** Sentry + UptimeRobot + logs PM2/Docker

## Gotchas específicos Zentra (verificar SIEMPRE en este perfil)

- ⚠️ **PM2 dual**: pueden coexistir DOS PM2 (root y usuario `zeni`). Usar SIEMPRE
  `su - zeni -c 'pm2 ...'`. El PM2 de root suele estar `errored` y se ignora.
- ⚠️ **Secrets en dos rutas**: `ecosystem.config.js` lee `/etc/zeni/secrets.env` +
  `/opt/zeni/portal/secrets.env`. Restart real: `pm2 delete && pm2 start ecosystem.config.js && pm2 save`.
- ⚠️ **Next.js standalone**: inlina `process.env` en build. Vars sensibles requieren
  rebuild — standalone NO incluye `.env.local` automáticamente.
- ⚠️ **Cloudflare**: SSL/TLS mode = Full (Strict), NUNCA "Flexible". Origin no alcanzable
  por IP directa (firewall del VPS solo acepta rangos Cloudflare → evita bypass del WAF).
- ⚠️ **502 Chatwoot**: si responde 502 → `docker restart chatwoot_app`.
- ⚠️ **Servicios internos** (PostgreSQL 5432, Redis 6379, n8n 5678, Chatwoot) → bind a
  `127.0.0.1` o red Docker interna, NUNCA `0.0.0.0`.
- ⚠️ **Backup de volúmenes Docker**: incluir uploads de Chatwoot, datos de n8n y
  `N8N_ENCRYPTION_KEY` (sin esa key los workflows quedan ilegibles).
- 📌 **Incidente histórico**: token Cloudflare expuesto (2026-05-24) — tokens CF con
  permisos mínimos y rotables.
