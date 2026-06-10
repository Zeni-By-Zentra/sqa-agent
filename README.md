# SQA Agent v6.0 — Software Quality Assurance · Enterprise Edition

> Agente especialista en calidad de software de nivel enterprise. Cubre el ciclo completo: **planifica la arquitectura ANTES del código, audita DURANTE el desarrollo y bloquea el deploy si algo es inseguro.** Aplica OWASP Top 10:2021, OWASP API:2023, ISO/IEC 25010, ISO 27001:2022, WCAG 2.1 AA, NIST SP 800-63B, CIS Benchmarks, CMMI y normativa colombiana. 16 checklists + base de datos de 20 brechas históricas, cargados dinámicamente desde GitHub. **Planifica, audita y corrige.**

### Novedades v6
- **`--plan`** — framework de planificación pre-desarrollo: matriz de decisión de stack, modelo de seguridad, borrador de schema, contrato de API, deployment, timeline PERT, matriz de riesgos y gate Definition of Ready.
- **`--security` / `--pentest`** — auditoría de seguridad enterprise: OWASP Top 10:2021 ítem por ítem, vectores específicos del stack (Next.js, Node, PostgreSQL, Docker, Nginx, PM2, Chatwoot, n8n, Cloudflare), escaneo de dependencias y checklist de pen-test.
- **`--breach-check`** — audita el proyecto contra 20 brechas históricas documentadas (Equifax, Log4Shell, SolarWinds, Capital One, Heartbleed, Colonial Pipeline, Optus…).
- **`--pagespeed`** — Core Web Vitals + presupuesto de performance + análisis estático y medición real.
- **`--a11y` / `--infra` / `--cicd`** — módulos dedicados de accesibilidad WCAG 2.1 AA, infraestructura y CI/CD.

Desarrollado por [Zeni by Zentra](https://github.com/Zeni-By-Zentra) · Jhonatan Ortega · [webzentra.com](https://webzentra.com)

**MIT License** © 2026 Zentra · Jhonatan Ortega · webzentra.com

Uso comercial, personal, educativo y open-source: **completamente libre**.
Solo pedimos atribución (mantener el aviso de copyright).

---

## Instalación

```bash
git clone https://github.com/Zeni-By-Zentra/sqa-agent ~/.claude/skills/sqa-agent
```

## Uso

```
/sqa-agent [código | descripción | patrón de archivos | flags]
```

---

## Modos disponibles

| Flag | Qué hace |
|------|----------|
| `--plan` | **Pre-desarrollo:** genera `ARCHITECTURE_PLAN.md` (stack, seguridad, schema, API, deploy, timeline, riesgos) + gate Definition of Ready. |
| `--security` | Auditoría de seguridad enterprise: OWASP Top 10:2021 + vectores del stack + dependencias. |
| `--pentest` | Checklist de pen-test guiado con comandos (solo sistemas propios/autorizados). |
| `--breach-check` | Audita contra 20 brechas históricas documentadas. |
| `--pagespeed [URL]` | Core Web Vitals + presupuesto de performance + medición real. |
| `--a11y` | Accesibilidad WCAG 2.1 AA completa + plan de testing manual. |
| `--infra` | Infraestructura: VPS, Nginx, Docker runtime, PM2, Cloudflare, backups, DR. |
| `--cicd` | Pipeline: quality gates, secrets en CI, supply chain, deploy/rollback. |
| *(sin flag)* | Auditoría completa 🔴🟡🟢 |
| `--quick` | Solo 🔴 Críticos. Máximo 5 hallazgos, < 2 min. Ideal para PR reviews. |
| `--full-audit` | Multi-área: todos los checklists relevantes + Scorecard + Roadmap por sprints. |
| `--fix` | Audita y **aplica correcciones mecánicas** directamente en los archivos (solo `[AUTO-FIXABLE]`). |
| `--fix --dry-run` | Muestra el diff propuesto sin modificar ningún archivo. |
| `--fix --all` | Como `--fix` pero también corrige hallazgos 🟡 Importantes. |
| `--diff` | Audita solo los cambios del **último commit** (`git diff HEAD~1..HEAD`). |
| `--diff HEAD~3` | Audita cambios desde un SHA o referencia específica. |
| `--diff --since main` | Audita cambios vs rama main/master. Para PR reviews completas. |
| `--staged` | Audita solo el staging area (`git diff --cached`). Para pre-commit hooks. |
| `--profile` | Análisis de performance: estático siempre + sondeo HTTP si se pasa URL. |
| `--colombia` | Cumplimiento normativo colombiano (Ley 1581, Circular 007 SIC, Ley 1273, Res. 1519). |
| `--ai-audit` | Detección de patrones inseguros en código generado por IA (Copilot, ChatGPT, Cursor). |
| `--community` | Badge de cumplimiento SQA Agent para README del proyecto. |
| `--learn` | Modo educativo con referencia a la norma aplicable en cada hallazgo. |
| `--report json` | Output machine-readable JSON para CI/CD integration. |

Los flags son combinables: `--full-audit --fix`, `--diff --quick`, `--profile --report json`.

---

## Ejemplos

```bash
# Code review estándar
/sqa-agent src/api/payments/route.ts

# Code review rápido de PR
/sqa-agent --quick src/api/auth/route.ts

# Solo auditar el último commit (ideal para CI)
/sqa-agent --diff

# Auditar cambios vs main antes de hacer merge
/sqa-agent --diff --since main --quick

# Ver qué corregiría antes de aplicar
/sqa-agent --fix --dry-run src/api/auth/route.ts

# Aplicar correcciones automáticas
/sqa-agent --fix src/api/auth/route.ts

# Análisis de performance estático + medir URL real
/sqa-agent --profile src/api/ https://myapp.com/api/health

# Auditoría completa de proyecto con output JSON para CI
/sqa-agent --full-audit --report json src/

# Pre-commit hook (solo lo que está en staging)
/sqa-agent --staged --quick

# Database quality
/sqa-agent src/lib/db.ts migrations/

# Accesibilidad
/sqa-agent src/components/forms/CheckoutForm.tsx

# App móvil
/sqa-agent Revisar seguridad de la app React Native de pedidos

# DevOps
/sqa-agent Dockerfile nginx.conf .github/workflows/deploy.yml

# Inicio de proyecto
/sqa-agent Voy a construir un SaaS de facturación para PyMEs colombianas con pagos Wompi
```

---

## Checklists disponibles

El agente detecta automáticamente qué checklist(s) usar. Se cargan desde GitHub en tiempo de ejecución.

| Checklist | Cuándo se activa | Normas aplicadas |
|-----------|-----------------|-----------------|
| [`code-review.md`](checklists/code-review.md) | Código fuente (TS, JS, Python, Go, etc.) | ISO/IEC 25010:2023 + OWASP Top 10:2021 |
| [`database.md`](checklists/database.md) | SQL, schema, migraciones, ORM models | ISO/IEC 25010 + PostgreSQL best practices + Ley 1581/2012 |
| [`testing-strategy.md`](checklists/testing-strategy.md) | Test files, coverage, "revisar tests" | ISTQB + F.I.R.S.T + Pirámide de testing + Mutation testing |
| [`accessibility.md`](checklists/accessibility.md) | HTML, componentes UI, ARIA, "accesibilidad" | WCAG 2.1 AA + Resolución 1519/2016 Colombia |
| [`mobile.md`](checklists/mobile.md) | React Native, Flutter, Swift, Kotlin | OWASP Mobile Top 10:2024 + iOS HIG + Material Design 3 |
| [`devops.md`](checklists/devops.md) | Dockerfile, CI/CD, nginx, infraestructura | CI/CD best practices + Docker + ISO 27001:2022 |
| [`api-quality.md`](checklists/api-quality.md) | Endpoints, rutas API, OpenAPI spec | REST Maturity Model + ISO/IEC 25010 |
| [`security.md`](checklists/security.md) | Solicitud explícita de auditoría de seguridad | OWASP Top 10:2021 + API Security:2023 + ISO 27001:2022 |
| [`performance.md`](checklists/performance.md) | "performance", "lento", "respuesta", URL, `--profile` | ISO/IEC 25010:2023 §4.2.1 + Web Vitals + RAIL Model |
| [`project-init.md`](checklists/project-init.md) | Descripción de proyecto nuevo | ISO por dominio + CMMI + Metodologías ágiles |
| [`quality-in-use.md`](checklists/quality-in-use.md) | "calidad en uso", "usabilidad", "satisfacción", "efectividad" | ISO 9126-4 + ISO 25010:2023 §4.1.4 + ISO 25022 |
| [`ai-generated-code.md`](checklists/ai-generated-code.md) | Código de Copilot, ChatGPT, Cursor, "vibe coding", `--ai-audit` | OWASP Top 10 + Anti-patrones IA + Licencias |
| [`data-quality.md`](checklists/data-quality.md) | "calidad de datos", "ETL", "migración de datos", "data quality" | ISO/IEC 25012 + Ley 1581/2012 + Ley 1266/2008 |

---

## Modo `--fix`: corrección automática con guardrails

El modo `--fix` aplica correcciones **solo cuando** cumplen todos estos criterios:
- El cambio está localizado en ≤ 5 líneas
- No altera la interfaz pública (API, exports, tipos)
- No requiere agregar dependencias externas
- La corrección es determinista (mismo resultado independientemente del contexto de negocio)

Los hallazgos que no cumplen estos criterios se marcan como `[MANUAL]` con la corrección propuesta pero sin aplicar.

```
## Correcciones aplicadas
| Archivo         | Hallazgo  | Estado              |
|-----------------|-----------|---------------------|
| src/api/auth.ts | [SEC-001] | ✅ Aplicado (línea 42) |
| src/lib/db.ts   | [DB-002]  | ⚠️ MANUAL — requiere migración |
```

---

## Clasificación de hallazgos

| Severidad | Significado | Acción |
|-----------|-------------|--------|
| 🔴 **Crítico** | Bloquea deploy. Riesgo de seguridad, pérdida de datos o violación normativa. | Resolver antes del próximo deploy |
| 🟡 **Importante** | Deuda técnica significativa o riesgo moderado. | Resolver en el próximo sprint |
| 🟢 **Sugerencia** | Mejora de calidad no urgente. | Backlog |

Cada hallazgo incluye: norma citada, clasificación `[AUTO-FIXABLE]` o `[MANUAL]`, archivo:línea, y esfuerzo estimado.

---

## Normas de referencia

| Norma | Versión | Área |
|-------|---------|------|
| ISO/IEC 25010 | 2023 | Modelo de calidad del producto software |
| OWASP Top 10 | 2021 | Seguridad aplicaciones web |
| OWASP API Security | 2023 | Seguridad de APIs REST/GraphQL |
| OWASP Mobile Top 10 | 2024 | Seguridad apps móviles |
| ISO/IEC 27001 | 2022 | Gestión de seguridad de la información |
| WCAG | 2.1 AA | Accesibilidad web (W3C) |
| CMMI DEV | 2.0 | Madurez de procesos de desarrollo |
| ISTQB | — | Testing: pirámide, F.I.R.S.T, mutation testing |
| NIST SP 800-63B | 2017 | Autenticación y gestión de identidades |
| PCI DSS | 4.0 | Seguridad en pagos con tarjeta |
| Google Web Vitals | 2024 | Performance web (LCP, FID, CLS, TTFB) |
| RAIL Model | — | Performance UX (Response, Animation, Idle, Load) |
| iOS HIG | 2024 | Human Interface Guidelines (Apple) |
| Material Design | 3 | Design system Android (Google) |
| Ley 1581 | 2012 | Protección de datos personales (Colombia) |
| Resolución 1519 | 2016 | Accesibilidad web entidades públicas (Colombia) |

---


## v6.0.0 (2026-06-10) — Enterprise Edition

Upgrade mayor a QA de nivel enterprise: el agente ahora cubre el ciclo completo, de la planificación de arquitectura al gate de deploy.

### Nuevos modos de ciclo de vida
- `--plan` — framework de planificación pre-desarrollo. Genera `ARCHITECTURE_PLAN.md` con 8 secciones (matriz de decisión de stack con scoring, modelo de seguridad + threat model STRIDE-lite, borrador de schema de BD, contrato de API, estrategia de deployment, timeline PERT, matriz de riesgos) y gate Definition of Ready que bloquea el inicio de desarrollo si faltan ítems críticos.
- `--security` — auditoría de seguridad enterprise: OWASP Top 10:2021 ítem por ítem.
- `--pentest` — checklist de pen-test en 6 fases con comandos reales (recon, auth, IDOR, inyección, lógica de negocio, infra).
- `--breach-check` — audita el proyecto contra 20 brechas históricas documentadas.
- `--pagespeed` — Core Web Vitals + presupuesto de performance + medición real/estática.
- `--a11y`, `--infra`, `--cicd` — módulos dedicados de accesibilidad, infraestructura y CI/CD.

### Nuevos archivos
- `checklists/pre-dev-planning.md` — framework completo de planificación de arquitectura (discovery, ADRs, stack de referencia Zentra, threat model, schema, API, deploy, PERT, riesgos, Definition of Ready).
- `checklists/infrastructure.md` — hardening de VPS Hetzner, Nginx, runtime Docker, PM2, Cloudflare, backups/DR, monitoreo y resiliencia, con gotchas reales del stack Zentra.
- `references/breach-database.md` — base de datos de 20 brechas públicas (Equifax/CVE-2017-5638, Log4Shell, SolarWinds, Capital One, Heartbleed, Colonial Pipeline, Uber, Optus, Marriott, Adobe, LinkedIn/Yahoo, Target, Codecov, event-stream, Tesla, Heartland, TalkTalk, 3CX, MOVEit, Cloudflare/Okta) mapeadas a controles OWASP y al checklist.

### Reescritos
- `checklists/security.md` — de checklist genérico a OWASP Top 10:2021 ítem por ítem + Parte B con vectores específicos del stack (Next.js incl. CVE-2025-29927, Node/Express, PostgreSQL, Docker, Nginx, PM2, Chatwoot, n8n, Cloudflare/Hetzner) + Parte C supply chain con comandos + Parte D pen-test. Cada control cita su precedente de brecha histórica.
- `checklists/performance.md` — añadido presupuesto de performance y auditoría `--pagespeed` (frontend Next.js, backend, CDN/caching, medición laboratorio vs campo).
- `SKILL.md` — v6.0.0: filosofía shift-left, evidencia tipada (CMD/CODE/INF), CVSS por severidad, veredicto de deploy, JSON con stack detectado y breach precedent.

## v5.1.0 (2026-05-26)

### Correcciones críticas
- SKILL.md: frontmatter actualizado a v5.0.0 (estaba en v4.0.0) — el agente ahora se identifica correctamente
- SKILL.md: licencia corregida de Business Source License → MIT (consistente con LICENSE y README)

### Nuevas funcionalidades implementadas en SKILL.md
- Lógica operativa completa para `--colombia`: detección de tipo de proyecto + normas aplicables + reporte por ente regulador
- Lógica operativa completa para `--ai-audit`: 26 ítems del checklist + resumen de verificaciones pasadas
- Lógica operativa completa para `--learn`: sección educativa en cada hallazgo con norma, riesgo y pasos
- Lógica operativa completa para `--community`: scoring de cumplimiento + niveles Oro/Plata/Bronce + badge

### Nuevos archivos
- `checklists/colombia-compliance.md` — 32 ítems de normativa colombiana: Ley 1581, 527, 1273, 1266, 1618 + sectores salud/fintech/público
- `sqa-cli` — Script bash para integración en CI/CD con `--fail-on-critical`

### Documentación mejorada
- `COMMUNITY.md` — Expandido con guía real de contribución, template de PR, guía de falsos positivos, entorno de desarrollo
- Tabla de detección en SKILL.md extendida con nuevos tipos: Data Quality, Quality in Use, AI Code Audit, Colombia Compliance
## Changelog

### v5.0.0 (2026-05-26)

- **Nuevo:** Checklist `quality-in-use.md` — Calidad en Uso: ISO 9126-4, ISO 25010
- **Nuevo:** Checklist `ai-generated-code.md` — Auditoría de código generado por IA: 26 verificaciones
- **Nuevo:** Checklist `data-quality.md` — Calidad de Datos: ISO 25012, Ley 1581/2012
- **Nuevo:** Checklist `performance.md` reescrito — Web Vitals 2024, RAIL, backend, móvil, BD
- **Nuevo:** Modo `--colombia` — Cumplimiento normativo colombiano
- **Nuevo:** Modo `--ai-audit` — Detección de patrones inseguros en código de IA
- **Nuevo:** Modo `--community` — Badge de cumplimiento para README
- **Nuevo:** Modo `--learn` — Modo educativo con referencia a norma aplicable
- **Nuevo:** `SECURITY.md` — Política de seguridad y reporte de vulnerabilidades
- **Nuevo:** `COMMUNITY.md` — Guía de contribución y código de conducta
- **Actualizado:** `security.md` — Normativa colombiana (Ley 1581, Circular 007 SIC, Ley 1273)
- **Actualizado:** `project-init.md` — Sección de normatividad colombiana
- **Actualizado:** `LICENSE` — Excepción Educativa Colombia (SENA, universidades, colegios)

### v4.0.0 (2026-05-25)
- **Nuevo:** Modo `--fix` — aplica correcciones mecánicas directamente en código con guardrails explícitos
- **Nuevo:** Modo `--fix --dry-run` — muestra diff propuesto sin modificar archivos
- **Nuevo:** Modo `--diff [SHA]` — audita solo cambios del último commit o desde un SHA específico
- **Nuevo:** Modo `--diff --since main` — audita cambios vs rama base (ideal para PRs)
- **Nuevo:** Modo `--staged` — audita staging area (para pre-commit hooks)
- **Nuevo:** Modo `--profile` — análisis de performance estático + sondeo HTTP dinámico
- **Nuevo:** Flag `--report json` — output machine-readable para integración CI/CD
- **Nuevo:** Checklist `performance.md` — N+1, índices, connection pooling, Web Vitals, compresión
- **Nuevo:** Clasificación `[AUTO-FIXABLE]` vs `[MANUAL]` en cada hallazgo
- **Nuevo:** Tabla de correcciones aplicadas en modo `--fix`
- **Cambio:** Licencia MIT → Business Source License 1.1
- **Fix:** Descripción YAML truncada para evitar errores de parsing

### v3.0.0
- Modo `--full-audit` con Scorecard + Roadmap por sprints
- 9 checklists especializados

### v2.0.0
- Checklists database, testing, accessibility, mobile, devops

### v1.0.0
- SQA Agent inicial — ISO 25010, OWASP, ISO 27001, CMMI

---


## Licencia

**MIT License** © 2026 Zentra · Jhonatan Ortega · webzentra.com

Uso comercial, personal, educativo y open-source: **completamente libre**.
Solo pedimos atribución (mantener el aviso de copyright).

**Excepción Educativa Colombia:** Uso gratuito garantizado para SENA, universidades,
instituciones técnicas y colegios de Colombia.

Hecho con ❤️ en Colombia 🇨🇴
