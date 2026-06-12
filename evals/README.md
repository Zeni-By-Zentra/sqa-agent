# Evals — SQA Agent

Casos de evaluación para verificar que el skill se comporta como se espera tras cada cambio.

## Estructura

`evals.json` contiene un array `evals`. Cada caso tiene:

- `id` — identificador numérico
- `mode` — modo/flag que ejercita
- `prompt` — entrada del usuario
- `expected_output` — descripción en prosa del comportamiento correcto
- `assertions` — lista de comprobaciones discretas y verificables
- `files` — archivos de contexto que el caso necesita (vacío si no aplica)

## Cobertura actual (14 casos)

| # | Modo | Qué valida |
|---|------|-----------|
| 1 | `--plan` | 8 secciones, STRIDE-lite, datos sensibles, Definition of Ready |
| 2 | `--security` | IDOR → A01:2025, numeración 2025, evidencia tipada |
| 3 | `--security` | Política de passwords vs NIST 800-63B-4 (sin composición/rotación) |
| 4 | `--a11y` | WCAG 2.2: target size 2.5.8, auth 3.3.8, 4.1.1 eliminado |
| 5 | `--breach-check` | SSRF → Capital One 2019, A01:2025 |
| 6 | `--breach-check` | Supply chain → A03:2025, pin por SHA, xz-utils |
| 7 | `--infra` + `--stack` | PaaS: omite secciones no aplicables, no FAIL |
| 8 | `--infra` + `--stack zentra` | Carga perfil, resuelve variables, aplica gotchas |
| 9 | `--colombia` | Ley 1581 + 1266 + radar PL IA como EN TRÁMITE |
| 10 | `--fix` | Impact graph, permisos por riesgo, test stubs |
| 11 | `--pentest` | Guardrail: rechaza objetivo de terceros sin autorización |
| 12 | `--quick` | Máx 5 críticos, sin ruido, no inventa hallazgos |
| 13 | `--ai-audit` | Math.random/MD5 → A04:2025 + CSPRNG/argon2 |
| 14 | exceptional conditions | fail-open en authz → A10:2025, fail-closed |

## Cómo correrlos

No hay runner automático embebido (el skill es content-only). Para evaluar:

1. **Manual / smoke**: invoca el skill con cada `prompt` y verifica las `assertions` a mano. Útil tras editar checklists.
2. **LLM-as-judge**: pasa `prompt` al skill, captura la respuesta y pídele a un modelo que marque cada `assertion` como pass/fail con justificación. Pondera críticos (guardrails #11, fail-open #14) como bloqueantes.
3. **Regresión de estándares**: tras cualquier `--meta-audit` que actualice normas, re-corre al menos los casos 2, 3, 4, 6 y 14 (los acoplados a versiones de norma).

## Mantenimiento

Cuando cambie una norma (p.ej. OWASP 2025 → 2027), actualiza las `assertions` que citan IDs/versiones específicas y añade un caso por cada categoría nueva.
