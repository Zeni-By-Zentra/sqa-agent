# Changelog

## v7.1.0 — 2026-06-12 · Parametrización + Evals

### Nuevo
- **Perfiles de stack** (`profiles/`): el agente ya no asume un stack fijo. Resolución en orden: `.sqa/profile.md` del proyecto → `--stack <nombre>` → `profiles/default.md` (auto-detección leyendo package.json, Dockerfile, compose, IaC, configs de proxy y CI).
  - `profiles/default.md` — genérico, agnóstico de proveedor, con tabla de detección y regla "sección no aplicable = omitir, no FAIL".
  - `profiles/zentra.md` — el stack Zentra (PM2/Nginx/Cloudflare/Hetzner/Chatwoot/n8n) con sus gotchas, ahora opt-in en vez de hardcodeado.
- **Suite de evals** (`evals/evals.json`): 14 casos cubriendo detección de modo, estándares 2025/2026, parametrización y guardrails (pentest sin autorización, fail-open). `evals/README.md` documenta cómo correrlos.
- Nuevo flag `--stack <perfil>` para forzar un perfil concreto.

### Cambiado
- `infrastructure.md` reescrito como **agnóstico de proveedor**: variables `{{RUNTIME}}`, `{{PROXY}}`, `{{CDN_WAF}}`, `{{SECRETS_STORE}}`, `{{BACKUP_TARGET}}`, `{{DB}}`, `{{PRIV_USER}}`, `{{HOST}}` resueltas desde el perfil. Los productos concretos pasan a ser ejemplos.
- `pre-dev-planning.md`: la tabla de stack pasó de "default Zentra obligatorio" a "referencia del perfil Zentra" — para otros proyectos se usa la matriz de decisión.

## v7.0.0 — 2026-06-12 · Standards Refresh 2026

### Cambios mayores
- **OWASP Top 10:2025** (final ene-2026) reemplaza 2021 en todo el checklist de seguridad:
  - Nuevo A03: Software Supply Chain Failures (con ítems de CI/CD, registries, typosquatting)
  - Nuevo A10: Mishandling of Exceptional Conditions (fail-closed, circuit breakers, canary deploys)
  - SSRF absorbido en A01 · Misconfiguration sube a #2 · Crypto baja a #4
- **NIST SP 800-63B-4** (final jul-2025): passwords min 8/recomendado 15 chars, prohibidas reglas de composición y rotación forzada, passkeys/FIDO2 = AAL2, MFA phishing-resistant como baseline
- **WCAG 2.2 AA** reemplaza 2.1: 6 criterios nuevos integrados (Target Size 24px, Accessible Authentication, Dragging, Focus Not Obscured, Consistent Help, Redundant Entry) + 4.1.1 Parser eliminado + contexto EAA (exigible UE jun-2025)
- **Breach database 20→24**: xz-utils 2024 (supply chain), Change Healthcare 2024 (MFA), Snowflake 2024 (SaaS shared responsibility), CrowdStrike 2024 (exceptional conditions)
- **Radar normativo Colombia 2025-2026**: PL 043/2025 (regulación IA) y PL 214/2025C (reforma Ley 1581) documentados como EN TRÁMITE con recomendaciones de diseño anticipado
- **ASVS 5.0** (may-2025) referenciado como estándar de verificación


## v6.0.1 (2026-06-11) — Risk-based fix + Meta-audit + LLM injection

### Nuevas funcionalidades

- **Risk-based `--fix` permissions** — reemplaza la regla "≤5 líneas" por modelo orientado al riesgo real del cambio:
  - 🔴 Crítico + Confianza ALTA → confirmación explícita antes de aplicar
  - 🔴 Crítico + Confianza MEDIA/BAJA → siempre `[MANUAL]` automático
  - 🟡 Importante + ALTA/MEDIA → dry-run visible, aplica salvo interrupción
  - 🟢 Sugerencia → aplica directamente sin interrupción
- **Campo `Confianza` obligatorio** en todos los hallazgos (ALTA/MEDIA/BAJA) — guía el modelo de permisos de `--fix`
- **Impact graph en `--fix`** — mapea importadores del archivo antes de aplicar. Advertencia explícita si >10 importadores directos.
- **Test stubs junto a cada corrección** — genera `describe/it` esqueleto con `// TODO` en el archivo de tests más cercano
- **Auditoría diferencial post-fix** — re-audita archivos modificados + importadores directos tras aplicar. Regresiones como `⚠️ REGRESIÓN POST-FIX`
- **Nuevo modo `--meta-audit`** — audita los propios checklists del repositorio SQA

### Nuevos archivos

- `checklists/meta-audit.md` — 30 ítems, 6 secciones: vigencia de normas, calidad de ítems, cobertura, falsos positivos, usabilidad, mantenibilidad

### Checklists actualizados

- `checklists/security.md` — nueva sección **Parte E: Inyección en LLMs** (10 controles basados en OWASP LLM Top 10:2025: prompt injection, sensitive info disclosure, system prompt leakage, excessive agency, XSS indirecto vía LLM). Precedentes: Samsung 2023, Bing Chat 2023.

### Cambios en SKILL.md

- `--meta-audit` añadido a la tabla de modos y al bloque "Cómo operar"
- URL `checklists/meta-audit.md` añadida a la tabla de fetch
- Formato estándar de hallazgo: campo `Confianza` ahora obligatorio
- Sección `--fix` reescrita: modelo de permisos basado en riesgo + impact graph + test stubs + post-fix diff
- Reglas estrictas #19 y #20 añadidas
- `--report json` v6: nuevos campos `confidence`, `test_stub_generated`, `impact_graph`, `post_fix_regressions`

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

## v5.0.0 (2026-05-26)

### Nuevos Checklists
- quality-in-use.md — Calidad en Uso: ISO 9126-4, ISO 25010
- ai-generated-code.md — Auditoría de código generado por IA: 26 verificaciones
- data-quality.md — Calidad de Datos: ISO 25012, Ley 1581/2012
- performance.md — Performance completo: Web Vitals 2024, RAIL, backend, móvil

### Nuevos Modos
- --colombia — Cumplimiento normativo colombiano
- --ai-audit — Detección de patrones inseguros en código de IA
- --community — Badge de cumplimiento para README
- --learn — Modo educativo con referencia a norma aplicable

### Checklists Actualizados
- security.md — Normativa colombiana: Ley 1581, Circular 007 SIC, Ley 1273
- project-init.md — Sección de normatividad colombiana para proyectos nuevos
- performance.md — Expandido con métricas de BD y métricas móviles

### Nueva Documentación
- SECURITY.md — Política de seguridad y reporte de vulnerabilidades
- COMMUNITY.md — Guía de contribución y código de conducta
- LICENSE — Excepción Educativa Colombia

## v4.0.0 (2026-05-25)
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

## v3.0.0
- Modo `--full-audit` con Scorecard + Roadmap por sprints
- 9 checklists especializados

## v2.0.0
- Checklists database, testing, accessibility, mobile, devops

## v1.0.0
- SQA Agent inicial — ISO 25010, OWASP, ISO 27001, CMMI
