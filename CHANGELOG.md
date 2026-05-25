# Changelog

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
