# SQA Agent v7.0 — Software Quality Assurance · Enterprise Edition

> Agente especialista en calidad de software de nivel enterprise. Cubre el ciclo completo: **planifica la arquitectura ANTES del código, audita DURANTE el desarrollo y bloquea el deploy si algo es inseguro.**
> Aplica OWASP Top 10:2025, OWASP API:2023, OWASP LLM Top 10:2025, OWASP ASVS 5.0, ISO/IEC 25010:2023, ISO 27001:2022, ISO/IEC 42001 (IA), WCAG 2.2 AA + EAA, NIST SP 800-63B-4, NIST CSF 2.0, CIS Benchmarks, CMMI y normativa colombiana.
> **17 checklists + base de datos de 24 brechas históricas**, cargados dinámicamente desde GitHub.

Desarrollado por [Zeni by Zentra](https://github.com/Zeni-By-Zentra) · Jhonatan Ortega · [webzentra.com](https://webzentra.com)

**MIT License** © 2026 Zentra · Jhonatan Ortega · webzentra.com — Uso libre: comercial, personal, educativo, open-source. Solo mantén la atribución.

**Excepción Educativa Colombia:** Uso gratuito garantizado para SENA, universidades, instituciones técnicas y colegios de Colombia.

---

## Instalación

```bash
git clone https://github.com/Zeni-By-Zentra/sqa-agent ~/.claude/skills/sqa-agent
```

## Uso

```
/sqa-agent [código | descripción | URL | patrón de archivos | flags]
```

---

## Modos disponibles

### Modos de ciclo de vida (Enterprise — v7.0)

| Flag | Fase | Qué hace |
|------|------|----------|
| `--plan` | Pre-desarrollo | Genera `ARCHITECTURE_PLAN.md` con 8 secciones: matriz de decisión de stack, modelo de seguridad + threat model STRIDE-lite, borrador de schema, contrato de API, estrategia de deployment, timeline PERT y matriz de riesgos. Termina con gate **Definition of Ready**. |
| `--security` | Desarrollo / Pre-deploy | Auditoría de seguridad enterprise: OWASP Top 10:2025 ítem por ítem, vectores específicos del stack (Next.js, Node, PostgreSQL, Docker, Nginx, PM2, Chatwoot, n8n, Cloudflare), escaneo de dependencias. |
| `--pentest` | Pre-deploy | Checklist de pen-test guiado en 6 fases con comandos reales (recon → auth → IDOR → inyección → lógica de negocio → infra). Solo sobre sistemas propios o con autorización escrita. |
| `--breach-check` | Cualquiera | Audita el proyecto contra los patrones raíz de 24 brechas históricas documentadas (Equifax, Log4Shell, SolarWinds, Capital One, xz-utils, Change Healthcare, Snowflake, CrowdStrike…). Responde: "¿este código repite el error que hundió a X?" |
| `--pagespeed [URL]` | Pre-deploy | Core Web Vitals, presupuesto de bundle, imágenes, fuentes, caching/CDN, backend y PostgreSQL. Con URL: mediciones reales (3 muestras + promedio). Sin URL: análisis estático. |
| `--a11y` | Desarrollo / Pre-deploy | Accesibilidad WCAG 2.2 AA completa (4 principios: perceivable, operable, understandable, robust) + plan de testing manual. |
| `--infra` | Pre-deploy / Operación | Infraestructura: hardening de VPS Hetzner, Nginx, Docker runtime, PM2, Cloudflare, backups, monitoreo y DR. |
| `--cicd` | Desarrollo | Pipeline: quality gates, builds reproducibles, secrets en CI, supply chain (acciones sin pin SHA), deploy y rollback. |

### Modos de auditoría y desarrollo

| Flag | Qué hace |
|------|----------|
| *(sin flag)* | Auditoría completa 🔴🟡🟢 — detecta el tipo de análisis automáticamente. |
| `--quick` | Solo 🔴 Críticos. Máximo 5 hallazgos, < 2 min. Ideal para PR reviews rápidas. |
| `--full-audit` | Multi-área: todos los checklists relevantes en paralelo + Scorecard + Roadmap por sprints. |
| `--fix` | Audita y aplica correcciones. Modelo de permisos basado en riesgo (ver sección abajo). |
| `--fix --dry-run` | Muestra el diff propuesto de cada corrección sin modificar archivos. |
| `--fix --all` | Como `--fix` pero también aplica hallazgos 🟡 Importantes. |
| `--diff [SHA]` | Audita solo los cambios del último commit o desde un SHA específico. |
| `--diff --since main` | Audita cambios vs rama main/master. Para PR reviews completas. |
| `--staged` | Audita solo el staging area (`git diff --cached`). Para pre-commit hooks. |
| `--profile` | Alias de `--pagespeed` (compatibilidad v5). |
| `--colombia` | Cumplimiento normativo colombiano: Ley 1581, Ley 527, Ley 1273, Ley 1266, Ley 1618, Circular 007 SIC + radar 2025-2026 (PL 043/2025 IA, PL 214/2025C). |
| `--ai-audit` | Detecta patrones inseguros en código generado por IA (Copilot, ChatGPT, Cursor, Gemini). 26 verificaciones. |
| `--learn` | Modo educativo: cada hallazgo incluye norma, riesgo concreto, corrección paso a paso y caso de estudio de brecha real. |
| `--community` | Badge de cumplimiento SQA Agent para el README del proyecto (niveles Oro/Plata/Bronce). |
| `--meta-audit` | Audita los propios checklists del repositorio SQA: vigencia de normas, cobertura, calidad de ítems, falsos positivos. |
| `--report json` | Output machine-readable JSON para CI/CD. Incluye `confidence`, `impact_graph` y `post_fix_regressions`. |

Los flags son combinables: `--security --fix --dry-run`, `--plan --colombia`, `--breach-check --learn`, `--diff --quick`, `--full-audit --report json`.

---

## Ejemplos

```bash
# Planificación de arquitectura antes de escribir código
/sqa-agent --plan Voy a construir un SaaS de CRM para PyMEs colombianas con pagos Wompi

# Auditoría de seguridad enterprise sobre el proyecto actual
/sqa-agent --security src/

# Verificar si el proyecto repite patrones de brechas históricas
/sqa-agent --breach-check src/ docker-compose.yml

# Pen-test guiado (solo sistemas propios o autorizados)
/sqa-agent --pentest https://miapp.com

# Performance + medición real de URL
/sqa-agent --pagespeed https://miapp.com/api/health

# Accesibilidad completa
/sqa-agent --a11y src/components/

# Auditoría de infraestructura
/sqa-agent --infra docker-compose.yml nginx.conf ecosystem.config.js

# Cumplimiento CI/CD
/sqa-agent --cicd .github/workflows/deploy.yml Dockerfile

# Code review rápido de PR (solo críticos)
/sqa-agent --quick src/api/auth/route.ts

# Ver qué corregiría antes de aplicar
/sqa-agent --fix --dry-run src/api/auth/route.ts

# Aplicar correcciones con guardrails de riesgo
/sqa-agent --fix src/api/auth/route.ts

# Auditar solo el último commit (ideal para CI)
/sqa-agent --diff

# Auditar cambios vs main antes de hacer merge
/sqa-agent --diff --since main --quick

# Pre-commit hook
/sqa-agent --staged --quick

# Auditoría completa con JSON para GitHub Actions
/sqa-agent --full-audit --report json src/

# Normativa colombiana + modo educativo
/sqa-agent --colombia --learn src/

# Detectar patrones inseguros en código generado por IA
/sqa-agent --ai-audit src/

# Auditar los checklists del propio SQA Agent
/sqa-agent --meta-audit

# Code review estándar
/sqa-agent src/api/payments/route.ts

# Auditoría de base de datos
/sqa-agent src/lib/db.ts migrations/

# App móvil
/sqa-agent Revisar seguridad de la app React Native de pedidos
```

---

## Modo `--fix`: modelo de permisos basado en riesgo

Reemplaza la regla anterior "≤ 5 líneas" por un modelo orientado al riesgo real del cambio:

| Severidad | Confianza | Acción |
|-----------|-----------|--------|
| 🔴 Crítico | ALTA | Muestra diff + **pide confirmación explícita** antes de aplicar |
| 🔴 Crítico | MEDIA o BAJA | Marca `[MANUAL]` automáticamente — no aplica |
| 🟡 Importante | ALTA o MEDIA | Muestra diff; aplica salvo que el usuario interrumpa |
| 🟡 Importante | BAJA | Marca `[MANUAL]` |
| 🟢 Sugerencia | cualquiera | Aplica directamente sin interrupción |

**Condiciones que siempre fuerzan `[MANUAL]`:** altera interfaz pública (API/exports/tipos), requiere dependencia externa, corrección no determinista.

**Antes de aplicar:** construye el impact graph (mapea importadores con `grep -r`). Si >10 importadores: advertencia explícita.

**Junto a cada corrección:** genera test stub `describe/it` esqueleto marcado `// TODO — generado por SQA Agent v7`.

**Tras aplicar:** re-audita archivos modificados + importadores directos. Regresiones como `⚠️ REGRESIÓN POST-FIX`.

```
## Correcciones aplicadas
| Archivo         | Línea | Hallazgo  | Confianza | Estado                          |
|-----------------|-------|-----------|-----------|----------------------------------|
| src/api/auth.ts | 42    | [SEC-001] | ALTA      | ✅ Aplicado                      |
| src/lib/db.ts   | 15    | [DB-002]  | MEDIA     | ⚠️ MANUAL — requiere migración   |
```

---

## Clasificación de hallazgos

| Severidad | Significado | CVSS equiv. | SLA |
|-----------|-------------|-------------|-----|
| 🔴 **Crítico** | Bloquea deploy. Explotable, pérdida de datos o violación normativa. | 7.0–10.0 | Antes del deploy / 24-72h en prod |
| 🟡 **Importante** | Deuda técnica significativa o riesgo moderado. | 4.0–6.9 | Próximo sprint |
| 🟢 **Sugerencia** | Mejora de mantenibilidad, legibilidad o eficiencia. | 0.1–3.9 | Backlog |

Cada hallazgo incluye: **norma citada**, **campo Confianza** (ALTA/MEDIA/BAJA), clasificación `[AUTO-FIXABLE]` o `[MANUAL]`, archivo:línea, evidencia y esfuerzo estimado. Los hallazgos de seguridad incluyen también precedente histórico y explotabilidad.

---

## Checklists disponibles

17 checklists + 1 base de referencias. Se detectan automáticamente y se cargan desde GitHub en tiempo de ejecución.

### Checklists Enterprise (v7.0)

| Checklist | Modo | Descripción |
|-----------|------|-------------|
| [`pre-dev-planning.md`](checklists/pre-dev-planning.md) | `--plan` | Framework de planificación pre-desarrollo: discovery, ADRs, threat model, schema, API, deploy, PERT, riesgos, Definition of Ready |
| [`infrastructure.md`](checklists/infrastructure.md) | `--infra` | Hardening VPS Hetzner, Nginx, Docker runtime, PM2, Cloudflare, backups/DR, monitoreo — con gotchas reales del stack Zentra |
| [`meta-audit.md`](checklists/meta-audit.md) | `--meta-audit` | Auditoría de los propios checklists SQA: 30 ítems en 6 secciones (vigencia, calidad, cobertura, falsos positivos, usabilidad, mantenibilidad) |

### Checklists de Seguridad

| Checklist | Cuándo se activa | Normas aplicadas |
|-----------|-----------------|-----------------|
| [`security.md`](checklists/security.md) | `--security` / auditoría explícita | OWASP Top 10:2025 ítem por ítem + stack-specific (Next.js, Node, PostgreSQL, Docker, Nginx, PM2, Chatwoot, n8n, Cloudflare) + supply chain + pen-test + **OWASP LLM Top 10:2025** |
| [`references/breach-database.md`](references/breach-database.md) | `--breach-check` / `--learn` | 24 brechas históricas documentadas: Equifax, Log4Shell, SolarWinds, Capital One, Colonial Pipeline, Uber, Optus, Codecov, MOVEit, xz-utils, Change Healthcare, Snowflake, CrowdStrike… |

### Checklists de Calidad de Código

| Checklist | Cuándo se activa | Normas aplicadas |
|-----------|-----------------|-----------------|
| [`code-review.md`](checklists/code-review.md) | Código fuente (TS, JS, Python, Go, etc.) | ISO/IEC 25010:2023 + OWASP Top 10:2025 |
| [`database.md`](checklists/database.md) | SQL, schema, migraciones, ORM models | ISO/IEC 25010 + PostgreSQL best practices + Ley 1581/2012 |
| [`testing-strategy.md`](checklists/testing-strategy.md) | Test files, coverage, "revisar tests" | ISTQB + F.I.R.S.T + Pirámide de testing + Mutation testing |
| [`api-quality.md`](checklists/api-quality.md) | Endpoints, rutas API, OpenAPI spec | OWASP API Security:2023 + REST Maturity Model + ISO/IEC 25010 |
| [`ai-generated-code.md`](checklists/ai-generated-code.md) | Código de Copilot, ChatGPT, Cursor, "vibe coding" | OWASP Top 10 + 26 anti-patrones IA + Licencias |
| [`data-quality.md`](checklists/data-quality.md) | "calidad de datos", ETL, migración de datos | ISO/IEC 25012 + Ley 1581/2012 + Ley 1266/2008 |

### Checklists de Plataforma y Experiencia

| Checklist | Cuándo se activa | Normas aplicadas |
|-----------|-----------------|-----------------|
| [`performance.md`](checklists/performance.md) | `--pagespeed`, "performance", URL | ISO/IEC 25010:2023 §4.2.1 + Core Web Vitals + RAIL Model + presupuesto de performance |
| [`accessibility.md`](checklists/accessibility.md) | `--a11y`, HTML, componentes UI, ARIA | WCAG 2.2 AA + Resolución 1519/2016 Colombia |
| [`mobile.md`](checklists/mobile.md) | React Native, Flutter, Swift, Kotlin | OWASP Mobile Top 10:2024 + iOS HIG + Material Design 3 |
| [`devops.md`](checklists/devops.md) | `--cicd`, Dockerfile, CI/CD, GitHub Actions | CI/CD best practices + Docker + ISO 27001:2022 |
| [`quality-in-use.md`](checklists/quality-in-use.md) | "calidad en uso", usabilidad, experiencia de usuario | ISO 9126-4 + ISO 25010:2023 §4.1.4 + ISO 25022 |

### Checklists de Cumplimiento y Proyecto

| Checklist | Cuándo se activa | Normas aplicadas |
|-----------|-----------------|-----------------|
| [`colombia-compliance.md`](checklists/colombia-compliance.md) | `--colombia`, "Ley 1581", normativa colombiana | Ley 1581/2012, Ley 527/1999, Ley 1273/2009, Ley 1266/2008, Ley 1618/2013, Circular 007 SIC |
| [`project-init.md`](checklists/project-init.md) | Descripción de proyecto nuevo | ISO por dominio + CMMI + Metodologías ágiles + normativa colombiana |

---

## Normas de referencia

| Norma | Versión | Área |
|-------|---------|------|
| ISO/IEC 25010 | 2023 | Modelo de calidad del producto software |
| OWASP Top 10 | 2025 (final ene-2026) | Seguridad aplicaciones web |
| OWASP ASVS | 5.0 (2025) | Verificación de seguridad de aplicaciones |
| OWASP API Security | 2023 | Seguridad de APIs REST/GraphQL |
| OWASP Mobile Top 10 | 2024 | Seguridad apps móviles |
| OWASP LLM Top 10 | 2025 | Seguridad en aplicaciones con LLMs |
| ISO/IEC 27001 | 2022 | Gestión de seguridad de la información |
| NIST SP 800-63B | Rev. 4 (2025) | Autenticación: passwords 8/15+, passkeys AAL2, sin rotación forzada |
| NIST CSF | 2.0 (2024) | Marco de ciberseguridad — incluye función GOVERN |
| ISO/IEC 42001 | 2023 | Sistema de gestión de IA (gobernanza de modelos) |
| NIST SP 800-53 | Rev. 5 | Controles de seguridad y privacidad |
| CIS Benchmarks | 2024 | Hardening de sistemas e infraestructura |
| WCAG | 2.2 AA | Accesibilidad web (W3C) — baseline EN 301 549 / EAA (UE, jun-2025) |
| CMMI DEV | 2.0 | Madurez de procesos de desarrollo |
| ISTQB | — | Testing: pirámide, F.I.R.S.T, mutation testing |
| PCI DSS | 4.0.1 | Seguridad en pagos con tarjeta |
| Core Web Vitals | 2024 | Performance web (LCP, INP, CLS, TTFB) |
| RAIL Model | — | Performance UX (Response, Animation, Idle, Load) |
| iOS HIG | 2024 | Human Interface Guidelines (Apple) |
| Material Design | 3 | Design system Android (Google) |
| Ley 1581 | 2012 | Protección de datos personales (Colombia) |
| Ley 1273 | 2009 | Delitos informáticos (Colombia) |
| Ley 527 | 1999 | Comercio electrónico / firma digital (Colombia) |
| Resolución 1519 | 2016 | Accesibilidad web entidades públicas (Colombia) |
| Circular 007 SIC | 2018 | Seguridad de la información — operadores de datos (Colombia) |

---

## Changelog

### v7.0.0 (2026-06-12) — Standards Refresh 2026
- **OWASP Top 10:2025** (final ene-2026): checklist renumerado completo. Nuevos A03 Software Supply Chain Failures y A10 Mishandling of Exceptional Conditions. SSRF absorbido en A01.
- **NIST SP 800-63B-4** (jul-2025): passwords mín 8 / recomendado 15 chars, prohibidas reglas de composición y rotación forzada, passkeys/FIDO2 = AAL2.
- **WCAG 2.2 AA + EAA**: 6 criterios nuevos integrados, 4.1.1 Parser eliminado, contexto European Accessibility Act (exigible UE jun-2025).
- **Breach database 20→24**: xz-utils, Change Healthcare, Snowflake, CrowdStrike (2024).
- **Radar normativo Colombia**: PL 043/2025 (IA) y PL 214/2025C (reforma Ley 1581) en trámite.
- **ISO/IEC 42001 + NIST CSF 2.0** en `--ai-audit` · **ASVS 5.0** como estándar de verificación.

### v6.0.1 (2026-06-11)
- **Nuevo:** Modo `--meta-audit` + checklist `meta-audit.md` (30 ítems, 6 secciones)
- **Nuevo:** Modelo de permisos basado en riesgo para `--fix` (reemplaza regla "≤5 líneas")
- **Nuevo:** Campo `Confianza` obligatorio en todos los hallazgos (ALTA/MEDIA/BAJA)
- **Nuevo:** Impact graph + test stubs + auditoría diferencial post-fix en `--fix`
- **Actualizado:** `security.md` — Parte E: Inyección en LLMs (OWASP LLM Top 10:2025, 10 controles)

### v6.0.0 (2026-06-10) — Enterprise Edition
- **Nuevos modos:** `--plan`, `--security`, `--pentest`, `--breach-check`, `--pagespeed`, `--a11y`, `--infra`, `--cicd`
- **Nuevos archivos:** `pre-dev-planning.md`, `infrastructure.md`, `references/breach-database.md`
- **Reescritos:** `security.md` (OWASP A01-A10 ítem por ítem + stack-specific + supply chain + pen-test), `performance.md` (presupuesto + pagespeed), `SKILL.md` (shift-left, evidencia tipada, veredicto de deploy)

### v5.1.0 (2026-05-26)
- Lógica operativa completa para `--colombia`, `--ai-audit`, `--learn`, `--community`
- Nuevo: `colombia-compliance.md` (32 ítems), `sqa-cli` para CI/CD

### v5.0.0 (2026-05-26)
- Nuevos checklists: `quality-in-use.md`, `ai-generated-code.md`, `data-quality.md`, `performance.md`
- Nuevos modos: `--colombia`, `--ai-audit`, `--community`, `--learn`

### v4.0.0 (2026-05-25)
- Nuevos modos: `--fix`, `--fix --dry-run`, `--diff`, `--staged`, `--profile`, `--report json`
- Clasificación `[AUTO-FIXABLE]` vs `[MANUAL]`

Ver historial completo en [CHANGELOG.md](CHANGELOG.md).

---

## Licencia

**MIT License** © 2026 Zentra · Jhonatan Ortega · webzentra.com

Hecho con ❤️ en Colombia 🇨🇴
