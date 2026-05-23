# SQA Agent — Software Quality Assurance

> Agente especialista en calidad de software que aplica normas ISO/IEC 25010, OWASP Top 10, ISO 27001 y CMMI de forma práctica y pragmática.

Desarrollado por [Zeni by Zentra](https://github.com/Zeni-By-Zentra).

## Instalación

```bash
git clone https://github.com/Zeni-By-Zentra/sqa-agent ~/.claude/skills/sqa-agent
```

## Uso

```
/sqa-agent [código | descripción de proyecto | patrón de archivos]
```

### Ejemplos

```
# Code review de un archivo
/sqa-agent src/api/auth/route.ts

# Auditoría de seguridad
/sqa-agent Revisar seguridad de los endpoints de pagos

# Inicio de proyecto
/sqa-agent Voy a construir un SaaS de facturación para PyMEs colombianas

# API review
/sqa-agent Revisar calidad de la API en app/api/
```

## Tipos de Análisis

| Tipo | Normas aplicadas |
|------|-----------------|
| Code Review | ISO/IEC 25010:2023 + OWASP Top 10:2021 |
| Security Audit | OWASP Top 10 + API Security + ISO 27001:2022 |
| API Quality | REST Maturity Model + ISO/IEC 25010 |
| Project Init | ISO por dominio + CMMI + Metodologías ágiles |

## Clasificación de Hallazgos

- 🔴 Crítico — bloquea deploy. Riesgo de seguridad o pérdida de datos.
- 🟡 Importante — resolver en el próximo sprint. Deuda técnica significativa.
- 🟢 Sugerencia — mejora de calidad no urgente.

## Normas de Referencia

| Norma | Versión | Área |
|-------|---------|------|
| ISO/IEC 25010 | 2023 | Calidad del producto software |
| OWASP Top 10 | 2021 | Seguridad web |
| OWASP API Security | 2023 | Seguridad APIs |
| ISO/IEC 27001 | 2022 | Seguridad de la información |
| CMMI DEV | 2.0 | Madurez de procesos |
| NIST SP 800-63B | 2017 | Autenticación e identidades |
| PCI DSS | 4.0 | Seguridad en pagos |
| WCAG | 2.1 AA | Accesibilidad web |

## Licencia

MIT — libre de usar, modificar y distribuir.
