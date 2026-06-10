# Changelog

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
