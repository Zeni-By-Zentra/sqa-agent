# Project Init — Metodología + Normas por Dominio

## PASO 1: ANALIZA ESTAS DIMENSIONES INTERNAMENTE

**Tipo de software:** web/móvil/embebido/api/desktop/data
**Criticidad:** Crítica (salud/finanzas/infraestructura) / Alta (SaaS/pagos) / Media (interno) / Baja (MVP)
**Tamaño equipo:** 1-3 / 4-9 / 10-50 / 50+
**Requisitos:** estables o cambiantes

## PASO 2: MATRIZ DE DECISIÓN METODOLOGÍA

- Equipo 1-5 + requisitos cambiantes → Kanban
- Equipo 3-9 + entregas regulares → Scrum (sprints 1-2 semanas)
- Equipo 5-20 + producto técnicamente complejo → XP + Scrum
- Requisitos muy estables + entrega única → Iterativo-Incremental
- Múltiples equipos + empresa mediana → SAFe Essentials
- Organización en transición ágil → Scrumban

NUNCA recomendar: Waterfall puro, CMMI Nivel 4-5 a equipos <50 personas, SAFe completo a <3 equipos.

## PASO 3: NORMAS POR DOMINIO

**Software Financiero / Pagos:**
- PCI DSS 4.0 — obligatorio si procesa tarjetas
- ISO/IEC 27001:2022
- OWASP Top 10
- ISO 8583 si usa mensajería financiera

**Software de Salud:**
- HIPAA (EE.UU.) o Resolución 1995/1999 (Colombia) — datos clínicos
- ISO 13485 — dispositivos médicos
- ISO/IEC 62304 — ciclo de vida software médico
- HL7 FHIR — interoperabilidad datos clínicos

**SaaS / Apps Web (general):**
- ISO/IEC 25010:2023 — calidad del producto
- OWASP Top 10:2021
- OWASP API Security:2023
- WCAG 2.1 AA — accesibilidad
- Ley 1581/2012 Colombia — protección datos personales

**E-commerce:**
- PCI DSS (si procesa pagos directamente)
- ISO/IEC 27001
- OWASP Top 10
- WCAG 2.1

**Apps Móviles:**
- OWASP Mobile Top 10:2023
- Google Play / App Store privacy guidelines
- WCAG 2.1

**Software Educativo:**
- ISO/IEC 19796-1
- SCORM 1.2 / xAPI
- WCAG 2.1 AA

## PASO 4: EQUIPO MÍNIMO VIABLE

- 1-3 personas: Full Stack + QA parcial. GitHub Issues + Kanban.
- 4-8 personas: Tech Lead + 2-4 Devs + QA + DevOps parcial. Jira + Scrum.
- 9-20 personas: EM + 2 Tech Leads + 4-10 Devs + 2 QA + DevOps + UX. Jira + Confluence + SAFe Essentials.

## PASO 5: CHECKLIST DE ARRANQUE

### Diseño/Arquitectura
- [ ] Architecture Decision Records (ADR)
- [ ] Threat Modeling para apps con datos sensibles
- [ ] Definition of Done del equipo
- [ ] Branch strategy definida (GitFlow o Trunk-Based)

### Desarrollo
- [ ] Code review obligatorio antes de merge (mínimo 1 reviewer)
- [ ] Linting y format automático en CI
- [ ] Tests unitarios para lógica crítica (70% cobertura mínima)
- [ ] Secrets management configurado

### Testing
- [ ] Tests de integración para APIs y flujos críticos
- [ ] Tests E2E para happy paths (Playwright/Cypress)
- [ ] Security scan en CI (Snyk, OWASP Dependency Check)
- [ ] Performance baseline establecido

### Deploy
- [ ] Pipeline CI/CD automatizado
- [ ] Rollback automatizado
- [ ] Monitoreo y alertas configurados
- [ ] Runbook de operaciones

## FORMATO DE REPORTE

## Análisis de Proyecto — [Nombre]
**Tipo:** [web/mobile/api/...]
**Criticidad:** [Crítica/Alta/Media/Baja]
**Dominio:** [fintech/salud/saas/...]

### Metodología Recomendada
**Principal:** [metodología] — [justificación en 2-3 líneas]
**Complementaria:** [si aplica]
**Por qué NO [alternativa obvia]:** [razón específica]

### Normas Aplicables
| Norma | Obligatoriedad | Justificación |
|-------|---------------|---------------|
| [norma] | Obligatoria/Recomendada | [razón] |

### Estructura de Equipo
[Roles mínimos para este proyecto]

### Riesgos Identificados
🔴 [riesgo crítico a mitigar desde el inicio]
🟡 [riesgo a monitorear]

## Normatividad Colombiana Aplicable

### Para proyectos que manejen datos personales
| Norma | Aplica a | Requisito clave |
|-------|---------|-----------------|
| Ley 1581/2012 | Todo proyecto con datos personales | Aviso de privacidad, consentimiento, política de tratamiento |
| Decreto 1377/2013 | Empresas que recolectan datos | Registro en RNBD de la SIC |
| Ley 1266/2008 | Sistemas de crédito/financieros | Habeas data financiero |
| Circular 007 SIC/2018 | Operadores de datos | Medidas de seguridad técnicas |

### Checklist de cumplimiento inicial Colombia
- ¿El proyecto maneja datos personales? → Registrar en RNBD (SIC)
- ¿Hay pagos en línea? → Integrar PSE con certificado PCI-DSS
- ¿Es una app pública? → Cumplir Ley 1618/2013 (accesibilidad)
- ¿Hay firma de contratos? → Firma electrónica válida (Ley 527)
- ¿Datos de menores de edad? → Consentimiento verificable de padres/tutores (Ley 1581 §7)
