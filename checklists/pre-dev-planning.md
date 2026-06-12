# Pre-Development Planning — Framework de Arquitectura Antes del Código

> **Regla de oro:** ninguna línea de código de producción se escribe sin un `ARCHITECTURE_PLAN.md` aprobado.
> Un bug de diseño detectado en planificación cuesta 1x. En desarrollo 10x. En producción 100x (IBM Systems Sciences Institute).

**Normas base:** ISO/IEC/IEEE 42010 (descripción de arquitectura), ISO/IEC 25010:2023 (atributos de calidad), OWASP SAMM (Design), NIST SP 800-160 (ingeniería de seguridad de sistemas).

---

## PASO 0: DISCOVERY — 12 preguntas obligatorias

Si el brief ya responde alguna, no repreguntar. Listar las que queden sin respuesta como **vacíos de discovery** (bloquean el Definition of Ready si son las preguntas 1, 3, 4 o 9).

1. **Objetivo de negocio:** ¿qué problema resuelve y cómo se mide el éxito? (KPI concreto)
2. **Usuarios:** ¿quiénes, cuántos al inicio, cuántos a 12 meses? ¿internos o públicos?
3. **Datos sensibles:** ¿maneja PII, datos financieros, salud, menores de edad? (define el modelo de seguridad y la normativa)
4. **Multi-tenancy:** ¿un cliente o muchos? ¿aislamiento por fila (tenant_id), por schema o por base de datos?
5. **Integraciones:** ¿con qué sistemas externos habla? (pagos, WhatsApp/Meta, CRM, email, webhooks)
6. **Volumen estimado:** requests/día, registros/mes, tamaño de archivos, picos esperados
7. **Disponibilidad requerida:** ¿qué pasa si se cae 1 hora? ¿SLA comprometido con el cliente?
8. **Presupuesto de infraestructura:** mensual, en USD/COP (define VPS vs managed vs serverless)
9. **Plazo y alcance:** ¿fecha dura o flexible? ¿MVP o producto completo?
10. **Equipo:** ¿quién desarrolla, quién opera, quién responde incidentes a las 3am?
11. **Ciclo de vida:** ¿proyecto de entrega única o producto con evolución continua?
12. **Restricciones impuestas:** stack obligado por el cliente, hosting requerido, normativa específica

---

## PASO 1: MATRIZ DE DECISIÓN DE STACK

### 1.1 Procedimiento

Para cada capa (frontend, backend, BD, hosting), evaluar candidatos con scoring 1-5 ponderado:

| Criterio | Peso | Justificación |
|----------|------|---------------|
| Experiencia del equipo | ×3 | El mejor stack es el que el equipo opera con confianza a las 3am |
| Fit con requisitos (1-12 del discovery) | ×3 | SSR/SEO, realtime, volumen, integraciones |
| Madurez y ecosistema | ×2 | Librerías, documentación, CVE response del proyecto |
| Costo total (licencias + infra + tiempo) | ×2 | Incluir costo de operación, no solo desarrollo |
| Contratabilidad / bus factor | ×1 | ¿Se consigue otro dev que lo mantenga? |

**Regla anti-resume-driven-development:** una tecnología nueva para el equipo solo entra si su score supera al stack conocido por ≥20% Y el proyecto tolera la curva de aprendizaje en el timeline.

### 1.2 Stack de referencia (opinionado — solo aplica si el proyecto adopta un perfil)

> Esta tabla es el stack por defecto del **perfil Zentra** (`profiles/zentra.md`). Para
> proyectos con otro perfil o sin perfil, NO impongas estas tecnologías: usa la matriz de
> decisión de stack (§2) partiendo de los requisitos reales. El valor del checklist es
> **forzar la decisión documentada**, no prescribir un stack único.

| Capa | Default del perfil Zentra | Cuándo desviarse |
|------|---------------------------|------------------|
| Frontend web app | Next.js 14+ App Router + TypeScript | SPA interna sin SEO → React + Vite |
| Landing / sitio contenido | Next.js SSG o WordPress (si el cliente edita) | — |
| API backend | Next.js API routes (monolito) o Node + Express (servicio separado) | Cargas CPU-bound → worker separado |
| Base de datos | PostgreSQL (queries parametrizadas) | Cache/colas → Redis. No elijas un motor "porque es rápido empezar" sin evaluar consistencia |
| Jobs / automatización | n8n (workflows de negocio) o node-cron/runtime (jobs de app) | — |
| Mensajería cliente | Chatwoot self-hosted + WhatsApp Cloud API | — |
| Deploy | Docker Compose / gestor de procesos en VPS + reverse proxy | Tráfico global estático → PaaS/edge (Vercel, Cloudflare Pages) |
| DNS/CDN/WAF | Cloudflare (proxied) | — |
| Monitoreo | Sentry + uptime checks + logs del runtime | — |

### 1.3 Decisiones que SIEMPRE deben quedar escritas (formato ADR)

```markdown
## ADR-001: [Decisión]
- **Contexto:** [qué problema fuerza la decisión]
- **Opciones evaluadas:** [A, B, C con score de la matriz]
- **Decisión:** [elegida]
- **Consecuencias:** [trade-offs aceptados, qué se hace más difícil]
- **Revisar cuando:** [condición que invalida la decisión, ej: >10k usuarios]
```

Mínimo obligatorio de ADRs por proyecto: stack por capa, estrategia multi-tenant, estrategia de auth, monolito vs servicios, build vs buy de cada integración mayor.

---

## PASO 2: MODELO DE SEGURIDAD (antes del código, no después)

### 2.1 Clasificación de datos

| Clase | Ejemplos | Requisito mínimo |
|-------|----------|------------------|
| Público | Landing, docs | Integridad (no defacement) |
| Interno | Métricas, configs | Auth requerida |
| Confidencial | PII, conversaciones, emails | Cifrado en tránsito + reposo, acceso auditado, Ley 1581 |
| Restringido | Passwords, tokens, datos financieros/salud | Hashing/cifrado fuerte, acceso mínimo, retención definida |

Entregable: tabla con CADA dato que el sistema almacena y su clase. Si un dato no tiene clase, no se almacena.

### 2.2 Decisión de autenticación

| Escenario | Elección |
|-----------|----------|
| SaaS multi-tenant web | Sesiones server-side con cookie `HttpOnly; Secure; SameSite=Lax` o NextAuth/Auth.js |
| API consumida por terceros | API keys por tenant con scopes + rate limit, o OAuth2 client credentials |
| SPA + API propia | Access token corto (15 min) + refresh token con rotación y revocación |
| Admin/back-office | Igual que usuarios + MFA obligatorio |

Definir SIEMPRE: política de passwords (≥12 chars, NIST SP 800-63B: sin rotación forzada, sí check contra listas filtradas), lockout (5-10 intentos + backoff), recuperación de cuenta, expiración e invalidación de sesión.

### 2.3 Decisión de autorización

- [ ] Modelo elegido y documentado: RBAC (roles fijos) / ABAC (atributos) / por-tenant
- [ ] Multi-tenant: TODA query lleva `tenant_id` en el WHERE — decidir si se refuerza con RLS de PostgreSQL
- [ ] Matriz rol × recurso × acción escrita ANTES de crear endpoints
- [ ] Regla IDOR: ningún endpoint recibe IDs de recursos sin verificar pertenencia (precedente: Optus 2022, ver breach-database)
- [ ] IDs públicos no enumerables (UUID v4 o nanoid, no seriales expuestos)

### 2.4 Threat model STRIDE-lite (30 minutos, no 3 días)

Para cada flujo crítico (login, pago, webhook entrante, upload, admin):

| Amenaza | Pregunta | Mitigación planificada |
|---------|----------|------------------------|
| **S**poofing | ¿Cómo sé que el caller es quien dice ser? | [auth, firma HMAC de webhooks] |
| **T**ampering | ¿Puede alterar datos en tránsito o reposo? | [TLS, validación server-side, constraints] |
| **R**epudiation | ¿Puedo probar quién hizo qué? | [audit log con user, IP, timestamp] |
| **I**nfo disclosure | ¿Qué se filtra si esto falla? | [errores genéricos, campos mínimos en respuesta] |
| **D**enial of service | ¿Qué pasa con 1000 req/s aquí? | [rate limit, límites de payload, timeouts] |
| **E**levation | ¿Puede un user volverse admin? | [checks server-side, nunca confiar en el cliente] |

### 2.5 Plan de secretos

- [ ] Inventario de secretos del proyecto (DB, APIs externas, JWT secret, SMTP, webhooks)
- [ ] Dónde viven: `.env` fuera de git + secrets del CI + secrets manager si aplica
- [ ] Quién accede a cuáles, cómo se rotan, qué pasa si se filtran (runbook de 5 líneas)

---

## PASO 3: BORRADOR DE SCHEMA DE BD

### 3.1 Convenciones obligatorias (PostgreSQL)

- [ ] Nombres `snake_case`, tablas en plural (`users`, `bot_flows`)
- [ ] PK: `id` (uuid v4 default `gen_random_uuid()`, o bigint identity si no se expone)
- [ ] `created_at timestamptz NOT NULL DEFAULT now()`, `updated_at timestamptz` en TODA tabla
- [ ] Multi-tenant: `tenant_id` NOT NULL + FK en TODA tabla de datos de cliente + índice compuesto `(tenant_id, ...)`
- [ ] FKs explícitas con `ON DELETE` decidido conscientemente (RESTRICT por default, CASCADE solo justificado)
- [ ] Decisión de soft-delete documentada (columna `deleted_at` vs borrado real vs tabla de archivo) — considerar Ley 1581 (derecho de supresión)
- [ ] `NOT NULL` por default; nullable es la excepción justificada
- [ ] Money: `numeric(12,2)` o enteros en centavos — NUNCA float
- [ ] Enums de negocio: `CHECK` constraint o tabla de referencia (no magic strings)

### 3.2 Entregable: borrador de schema

Para cada entidad: nombre, columnas con tipos, relaciones (1-N, N-M), índices previstos (toda FK + toda columna de WHERE frecuente), volumen estimado a 12 meses y campos con datos de clase Confidencial/Restringido marcados.

### 3.3 Estrategia de migración

- [ ] Herramienta definida (node-pg-migrate / Prisma Migrate / etc.) desde el día 1 — nunca `psql` manual contra producción
- [ ] Migraciones backward-compatible con la versión anterior del código (expand → migrate → contract)
- [ ] Plan de seed data para desarrollo y staging
- [ ] Usuario de aplicación con permisos mínimos (no superuser, no owner del schema)

---

## PASO 4: CONTRATO DE API

### 4.1 Convenciones (decidir UNA vez, aplicar siempre)

- [ ] Estilo: REST con recursos en plural (`GET /api/v1/contacts/:id`)
- [ ] Versionado en path (`/api/v1/`) desde el primer endpoint
- [ ] Formato de error único en TODA la API:
```json
{ "error": { "code": "VALIDATION_ERROR", "message": "email is required", "details": [], "request_id": "..." } }
```
- [ ] Códigos HTTP correctos: 400 validación, 401 sin auth, 403 sin permiso, 404 no existe O no es tuyo (no filtrar existencia), 409 conflicto, 422 semántica, 429 rate limit, 5xx solo errores del servidor
- [ ] Paginación por cursor para listas que crecen (`?cursor=...&limit=50`, máx 100); offset solo para datasets pequeños
- [ ] Idempotencia: mutaciones críticas (pagos, envíos) aceptan `Idempotency-Key`
- [ ] Rate limits documentados por endpoint + headers `X-RateLimit-*` / `Retry-After`
- [ ] Validación de entrada con schema (zod/joi) en el borde — tipos, rangos, longitudes, allowlist

### 4.2 Webhooks (entrantes y salientes)

- [ ] Entrantes: firma verificada (HMAC-SHA256 con timestamp anti-replay) — NUNCA un webhook sin auth (gotcha histórico del propio stack)
- [ ] Salientes: firma propia + reintentos con backoff exponencial + dead letter
- [ ] Respuesta rápida (<5s): encolar y responder 200, procesar async

### 4.3 Entregable

Stub OpenAPI 3.1 (o tabla equivalente) con: endpoints del MVP, auth por endpoint, request/response de ejemplo, errores posibles. No hace falta el spec completo — sí el contrato de los flujos críticos.

---

## PASO 5: ESTRATEGIA DE DEPLOYMENT

- [ ] Ambientes definidos: development (local) / staging (obligatorio si hay cliente) / production — con secrets DIFERENTES
- [ ] Runtime decidido: Docker Compose (aislamiento, múltiples servicios) vs PM2 (apps Node simples en VPS compartido) — ADR
- [ ] Topología Nginx: dominios, TLS (Cloudflare Full Strict + cert de origen), proxy_pass, websockets si aplica
- [ ] CI/CD desde el día 1: lint → test → build → security scan → deploy (ver `checklists/devops.md`)
- [ ] Rollback definido y ENSAYADO antes del primer deploy real (tag anterior + migraciones backward-compatible)
- [ ] Backups: qué, frecuencia, destino off-site (ej: R2), retención, y **prueba de restore agendada**
- [ ] Monitoreo desde el día 1: uptime (UptimeRobot), errores (Sentry), disco/memoria del VPS
- [ ] Runbook inicial: cómo se despliega, cómo se hace rollback, cómo se reinicia, dónde están los logs, a quién llamar

---

## PASO 6: TIMELINE ESTIMADO

### 6.1 Método: PERT de 3 puntos por bloque de trabajo

`Estimado = (Optimista + 4×Probable + Pesimista) / 6`

- [ ] Desglosar en bloques de ≤3 días; un bloque que no se puede estimar es un bloque que no se entiende → spike primero
- [ ] Buffer global ×1.4–1.5 sobre la suma (integraciones con terceros: ×2 — Meta/WhatsApp reviews, pasarelas de pago)
- [ ] Incluir SIEMPRE como bloques con horas propias: setup CI/CD e infra, testing, auditoría SQA pre-deploy, corrección post-audit, deploy + estabilización
- [ ] Hitos verificables, no porcentajes: "staging con login + flujo X funcionando", no "60% del backend"
- [ ] App reviews externos (Meta, TikTok, App Store) en el camino crítico desde la semana 1 — son semanas de espera no comprimibles

### 6.2 Entregable

Tabla: bloque · estimado PERT · dependencia · hito · semana. Veredicto explícito: ¿la fecha pedida es viable? SÍ / NO / SÍ-recortando-[alcance].

---

## PASO 7: MATRIZ DE RIESGOS

Score = Probabilidad (1-3) × Impacto (1-3). ≥6 = plan de mitigación obligatorio antes de empezar.

| # | Riesgo | P | I | Score | Mitigación | Dueño | Trigger de revisión |
|---|--------|---|---|-------|------------|-------|---------------------|

Catálogo mínimo a evaluar en todo proyecto (marcar aplica/no aplica):
1. Dependencia de aprobación de terceros (Meta App Review, pasarela) en camino crítico
2. Requisito normativo no contemplado (Ley 1581, PCI si tocan tarjetas)
3. Volumen real >> volumen estimado (¿el diseño aguanta 10x?)
4. Single point of failure humano (bus factor = 1)
5. Integración con sistema legado sin documentación
6. Scope creep sin proceso de cambio acordado
7. Datos sensibles sin clasificar descubiertos a mitad de desarrollo
8. Costos de API externa (LLM, WhatsApp) que escalan con uso
9. Cliente sin ambiente de pruebas para integraciones
10. Deuda de seguridad aplazada "para después del MVP" (después = nunca)

---

## PASO 8: GATE — DEFINITION OF READY

El desarrollo NO inicia hasta que todo ítem **[B]loqueante** esté ✅:

- [ ] **[B]** Discovery completo (preguntas 1, 3, 4, 9 respondidas)
- [ ] **[B]** Matriz de stack con ADRs de decisiones mayores
- [ ] **[B]** Clasificación de datos + modelo authn/authz documentado
- [ ] **[B]** Threat model STRIDE-lite de flujos críticos
- [ ] **[B]** Borrador de schema con convenciones y estrategia de migración
- [ ] **[B]** Contrato de API de flujos críticos (formato de error, auth, paginación)
- [ ] **[B]** Estrategia de deploy + rollback + backups definida
- [ ] **[B]** Timeline PERT con buffer y veredicto de viabilidad
- [ ] **[B]** Matriz de riesgos con mitigación para score ≥6
- [ ] Repo creado con CI básico (lint+test) y `.gitignore` correcto
- [ ] Secrets inventariados y `.env.example` creado
- [ ] Definition of Done del equipo acordado
- [ ] Branch strategy definida

**Veredicto final del modo --plan:**
```
✅ READY — desarrollo puede iniciar
⛔ NOT READY — bloqueantes pendientes: [lista]
```
