# Meta-Audit Checklist — Auditoría de Checklists SQA

Aplica este checklist a cada checklist del repositorio SQA Agent.
Propósito: garantizar que los propios controles de calidad estén vigentes, sean verificables y cubran los vectores actuales.

**Norma base:** ISO/IEC 25010:2023 §4.6 (Mantenibilidad), ISO/IEC 27001:2022 §9 (Evaluación del desempeño), CMMI L3 (Verificación/Validación).

---

## 1. VIGENCIA DE NORMAS

- [ ] Cada ítem cita una norma con número de sección específico (no solo "OWASP" o "ISO 27001" sin más)
- [ ] Las normas referenciadas están en su versión actual (ISO 27001:2022 no 2013; OWASP Top 10:2021 no 2017; WCAG 2.1/2.2 no 2.0)
- [ ] No se citan normas derogadas o superadas (PCI-DSS 3.x, OWASP 2017 como única referencia, WCAG 2.0 sin mención de versiones posteriores)
- [ ] Los links o referencias externas son accesibles y no devuelven 404
- [ ] La fecha de última revisión del checklist tiene menos de 12 meses

## 2. CALIDAD DE LOS ÍTEMS

- [ ] Cada ítem es binario y verificable: se puede responder SÍ o NO sin ambigüedad
- [ ] Ningún ítem tiene dos condiciones en uno (ej: "verifica cifrado Y autenticación" → deben ser ítems separados)
- [ ] Los ítems usan verbos de acción concretos: "verifica", "comprueba", "confirma" — no "considera" o "evalúa"
- [ ] Los ítems de seguridad especifican el vector de ataque que mitigan, no solo el control genérico
- [ ] Los ítems de rendimiento especifican umbrales medibles (ej: "< 200ms p95", no "rápido")
- [ ] Los ítems de accesibilidad especifican el nivel WCAG (A, AA, AAA) y el criterio de éxito exacto

## 3. COBERTURA

- [ ] El checklist cubre los vectores del OWASP Top 10:2021 relevantes al área auditada
- [ ] El checklist incluye al menos un ítem para el vector más reciente o emergente del área (LLM injection, supply chain attacks, etc.)
- [ ] No hay brechas obvias vs el checklist oficial de la norma base
- [ ] La normativa colombiana relevante al área está incluida (Ley 1581, Ley 1273, Ley 527 según aplique)
- [ ] El checklist incluye casos de borde comunes (null, vacío, tamaño máximo, concurrencia)

## 4. FALSOS POSITIVOS Y NEGATIVOS

- [ ] Los ítems no son tan genéricos que siempre apliquen independientemente del contexto (generan hallazgos inflados)
- [ ] Los ítems no son tan específicos que rara vez apliquen (quedan ciegos en casos reales)
- [ ] Los ítems de seguridad han sido validados contra al menos un caso real de brecha documentada
- [ ] Existe al menos un ítem que diferencie entre código legacy y código nuevo cuando el criterio difiere
- [ ] Los ítems de mantenibilidad incluyen umbrales razonables para el contexto (startup vs enterprise)

## 5. USABILIDAD DEL CHECKLIST

- [ ] El checklist tiene secciones numeradas y títulos claros
- [ ] El orden de secciones refleja prioridad (lo más crítico primero)
- [ ] El formato de reporte al final del checklist incluye severidad, norma, archivo y corrección
- [ ] El checklist puede completarse en menos de 30 minutos para una revisión media
- [ ] Existe al menos un ejemplo de hallazgo completo con todos los campos

## 6. MANTENIBILIDAD DEL CHECKLIST

- [ ] El checklist tiene un header con: versión o fecha de última revisión y norma base
- [ ] Los ítems son independientes entre sí (ninguno presupone que otro fue verificado antes)
- [ ] Los ítems no duplican controles de otros checklists del repositorio (DRY entre checklists)
- [ ] El archivo tiene menos de 250 líneas — si es mayor, considerar dividir por subárea
- [ ] Los cambios al checklist tienen trazabilidad en CHANGELOG.md

---

## FORMATO DE REPORTE

## Meta-Audit Report — [nombre del checklist]
**Fecha:** [fecha] | **Checklist auditado:** [archivo.md]
**Norma base del checklist:** [norma principal que cubre]
**Última revisión documentada:** [fecha o "no documentada"]

### Hallazgos

🔴/🟡/🟢 [META-001] **Título**
- **Sección:** [número de sección de este meta-checklist]
- **Confianza:** ALTA | MEDIA | BAJA
- **Problema:** [descripción específica del defecto en el checklist]
- **Impacto:** [qué tipo de brecha real puede quedar sin detectar]
- **Corrección:** [cómo mejorar el ítem o sección afectada]

### Meta-Scorecard

| Categoría | Ítems OK | Problemas | Estado |
|-----------|----------|-----------|--------|
| Vigencia de normas | N/5 | N | ✅/⚠️/❌ |
| Calidad de ítems | N/6 | N | ✅/⚠️/❌ |
| Cobertura | N/5 | N | ✅/⚠️/❌ |
| Falsos positivos | N/5 | N | ✅/⚠️/❌ |
| Usabilidad | N/5 | N | ✅/⚠️/❌ |
| Mantenibilidad | N/5 | N | ✅/⚠️/❌ |

**Veredicto:** ✅ Robusto / ⚠️ Mejoras disponibles / ❌ Desactualizado

**Acciones recomendadas:**
1. [acción concreta con esfuerzo estimado]
