# SQA Agent — Software Quality Assurance

> Agente especialista en calidad de software que aplica normas ISO/IEC 25010, OWASP, ISO 27001, WCAG 2.1 y CMMI de forma práctica y pragmática. 9 checklists especializados que se cargan dinámicamente desde GitHub.

Desarrollado por [Zeni by Zentra](https://github.com/Zeni-By-Zentra).

---

## Instalación

```bash
git clone https://github.com/Zeni-By-Zentra/sqa-agent ~/.claude/skills/sqa-agent
```

## Uso

```
/sqa-agent [código | descripción de proyecto | patrón de archivos | --quick]
```

### Modo rápido (`--quick`)

Solo revisa ítems 🔴 Críticos. Ideal para revisiones rápidas de PRs. Respuesta en < 2 minutos, máximo 5 hallazgos.

```
/sqa-agent --quick src/api/auth/route.ts
```

### Auditoría completa (sin flag)

Revisa todos los niveles: 🔴 Crítico, 🟡 Importante, 🟢 Sugerencia. Reporte completo con norma citada en cada hallazgo.

---

## Ejemplos

```bash
# Code review
/sqa-agent src/api/payments/route.ts

# Code review rápido de PR
/sqa-agent --quick src/api/auth/route.ts

# Database quality
/sqa-agent src/lib/db.ts migrations/

# Testing strategy
/sqa-agent src/__tests__/ jest.config.ts

# Accesibilidad
/sqa-agent src/components/forms/CheckoutForm.tsx

# App móvil
/sqa-agent Revisar seguridad de la app React Native de pedidos

# DevOps
/sqa-agent Dockerfile nginx.conf .github/workflows/deploy.yml

# Auditoría de seguridad
/sqa-agent Revisar seguridad completa del módulo de pagos

# Inicio de proyecto
/sqa-agent Voy a construir un SaaS de facturación para PyMEs colombianas con pagos Wompi
```

---

## Checklists disponibles

El agente detecta automáticamente qué checklist(s) usar según la entrada. Los checklists se cargan desde GitHub en tiempo de ejecución — el SKILL.md local es ligero (~300 tokens).

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
| [`project-init.md`](checklists/project-init.md) | Descripción de proyecto nuevo | ISO por dominio + CMMI + Metodologías ágiles |

---

## Clasificación de hallazgos

| Severidad | Significado | Acción |
|-----------|-------------|--------|
| 🔴 **Crítico** | Bloquea deploy. Riesgo de seguridad, pérdida de datos o violación normativa grave. | Resolver antes del próximo deploy |
| 🟡 **Importante** | Deuda técnica significativa o riesgo moderado. | Resolver en el próximo sprint |
| 🟢 **Sugerencia** | Mejora de calidad no urgente: mantenibilidad, legibilidad, eficiencia. | Backlog |

Cada hallazgo incluye la norma específica citada: `OWASP A03:2021`, `ISO/IEC 25010:2023 §4.5`, `WCAG 1.3.1 (Level A)`, `OWASP Mobile M3`, etc.

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
| iOS HIG | 2024 | Human Interface Guidelines (Apple) |
| Material Design | 3 | Design system Android (Google) |
| Ley 1581 | 2012 | Protección de datos personales (Colombia) |
| Resolución 1519 | 2016 | Accesibilidad web entidades públicas (Colombia) |

---

## Licencia

MIT — libre de usar, modificar y distribuir.
