# Colombia Compliance Checklist — Normativa Legal Colombiana

## Activación
Este checklist se activa cuando el usuario usa `--colombia` o menciona:
- "normativa colombiana"
- "Ley 1581"
- "datos personales Colombia"
- "SIC"
- "Superintendencia"
- "DIAN"
- "cumplimiento legal Colombia"
- "habeas data"

## Marco Legal Aplicable

Colombia tiene un ecosistema legal digital robusto. Este checklist cubre las normas más relevantes para proyectos de software.

---

## 1. Ley 1581/2012 — Protección de Datos Personales (Habeas Data)

**Ente regulador:** Superintendencia de Industria y Comercio (SIC)
**Aplica a:** Todo proyecto que recolecte, procese o almacene datos personales de ciudadanos colombianos.

| ID | Verificación | Norma | Severidad |
|----|-------------|-------|-----------|
| COL-001 | ¿Existe una Política de Tratamiento de Datos publicada? | Ley 1581 Art. 13 | 🔴 Crítico |
| COL-002 | ¿Se obtiene consentimiento explícito antes de recolectar datos? | Ley 1581 Art. 9 | 🔴 Crítico |
| COL-003 | ¿El sistema está registrado en el RNBD (Registro Nacional de Bases de Datos)? | Ley 1581 + Decreto 90/2018 | 🔴 Crítico |
| COL-004 | ¿Hay mecanismo para que el titular ejerza derechos (consulta, corrección, supresión)? | Ley 1581 Art. 8 | 🔴 Crítico |
| COL-005 | ¿Los datos sensibles (salud, biométricos, religión, etc.) tienen protección especial? | Ley 1581 Art. 6 | 🔴 Crítico |
| COL-006 | ¿Hay proceso documentado para notificar incidentes de seguridad a la SIC? | Circular 007 SIC/2018 | 🔴 Crítico |
| COL-007 | ¿Los datos de menores de edad tienen consentimiento verificable de padres/tutores? | Ley 1581 Art. 7 | 🔴 Crítico |
| COL-008 | ¿Existe política de retención y eliminación de datos definida? | Ley 1581 Art. 4(e) | 🟡 Importante |
| COL-009 | ¿Se cifran los datos personales en reposo? (AES-256 mínimo) | Circular 007 SIC/2018 | 🔴 Crítico |
| COL-010 | ¿Los logs de acceso a datos personales se conservan mínimo 5 años? | Circular 007 SIC/2018 | 🟡 Importante |

**Consecuencias de incumplimiento:** Multas hasta 2.000 SMMLV (~$2.600 millones COP en 2026). Cierre temporal del sistema.

---

## 2. Ley 527/1999 — Comercio Electrónico y Firma Digital

**Ente regulador:** Ministerio de Comercio, Industria y Turismo + Entidades de Certificación
**Aplica a:** Proyectos de e-commerce, contratos digitales, transacciones en línea.

| ID | Verificación | Norma | Severidad |
|----|-------------|-------|-----------|
| COL-011 | ¿Los contratos electrónicos tienen los elementos de validez jurídica? | Ley 527 Art. 14 | 🔴 Crítico |
| COL-012 | ¿Las firmas electrónicas usan entidades certificadas (Certicámara, GSE, etc.)? | Ley 527 Art. 28 | 🟡 Importante |
| COL-013 | ¿Los mensajes de datos se conservan en formato íntegro y accesible? | Ley 527 Art. 12 | 🟡 Importante |
| COL-014 | ¿El sitio tiene términos y condiciones claros antes de la transacción? | Ley 527 + Estatuto del Consumidor | 🔴 Crítico |
| COL-015 | ¿Se provee confirmación de la transacción al consumidor? | Ley 1480/2011 Art. 50 | 🔴 Crítico |

---

## 3. Ley 1273/2009 — Delitos Informáticos

**Ente regulador:** Fiscalía General de la Nación
**Aplica a:** Todo proyecto — define responsabilidades penales del desarrollador.

| ID | Verificación | Norma | Severidad |
|----|-------------|-------|-----------|
| COL-016 | ¿El sistema impide acceso no autorizado a sistemas de terceros? | Ley 1273 Art. 269A | 🔴 Crítico |
| COL-017 | ¿No hay funcionalidades que intercepten datos de comunicaciones sin autorización? | Ley 1273 Art. 269C | 🔴 Crítico |
| COL-018 | ¿El código no puede ser usado para suplantar sitios web (phishing)? | Ley 1273 Art. 269G | 🔴 Crítico |
| COL-019 | ¿Hay controles para prevenir uso del sistema en fraude electrónico? | Ley 1273 Art. 269I | 🔴 Crítico |

**Consecuencias de incumplimiento:** Penas de prisión de 48 a 96 meses para el responsable técnico.

---

## 4. Ley 1266/2008 — Habeas Data Financiero

**Ente regulador:** Superintendencia Financiera de Colombia (SFC)
**Aplica a:** Proyectos que manejen historial crediticio, scores financieros, datos bancarios.

| ID | Verificación | Norma | Severidad |
|----|-------------|-------|-----------|
| COL-020 | ¿Se consultan centrales de riesgo solo con autorización del titular? | Ley 1266 Art. 13 | 🔴 Crítico |
| COL-021 | ¿El sistema permite al titular consultar su información en centrales de riesgo? | Ley 1266 Art. 6 | 🔴 Crítico |
| COL-022 | ¿Los datos negativos se eliminan al pagar la deuda (máx. el doble del tiempo en mora)? | Ley 1266 Art. 13 | 🔴 Crítico |
| COL-023 | ¿Las integraciones con Datacrédito/TransUnion usan canales seguros? | SFC Circular 007 | 🔴 Crítico |

---

## 5. Ley 1618/2013 — Accesibilidad para Personas con Discapacidad

**Ente regulador:** Ministerio de Tecnologías de la Información (MinTIC)
**Aplica a:** Aplicaciones y sitios web públicos, especialmente de entidades estatales o con uso masivo.

| ID | Verificación | Norma | Severidad |
|----|-------------|-------|-----------|
| COL-024 | ¿La interfaz cumple WCAG 2.1 Nivel AA? | Ley 1618 + Resolución 1519/2016 | 🔴 Crítico |
| COL-025 | ¿Hay alternativas de texto para imágenes y contenido multimedia? | Ley 1618 Art. 8 | 🔴 Crítico |
| COL-026 | ¿La navegación es posible solo con teclado? | WCAG 2.1 Criterio 2.1.1 | 🟡 Importante |

---

## 6. Normativa Sectorial Adicional

### Sector Salud
| ID | Verificación | Norma | Severidad |
|----|-------------|-------|-----------|
| COL-027 | ¿Las historias clínicas electrónicas cumplen la Resolución 1995/1999? | Min. Salud Res. 1995 | 🔴 Crítico |
| COL-028 | ¿Los datos de salud tienen cifrado adicional al de datos personales comunes? | Ley 1581 Art. 6 (dato sensible) | 🔴 Crítico |

### Sector Financiero
| ID | Verificación | Norma | Severidad |
|----|-------------|-------|-----------|
| COL-029 | ¿Las pasarelas de pago integradas son autorizadas por la SFC? | SFC Circular Básica Jurídica | 🔴 Crítico |
| COL-030 | ¿El sistema implementa autenticación fuerte para transacciones financieras? | SFC Circular 007 Cap. 12 | 🔴 Crítico |

### Entidades Públicas
| ID | Verificación | Norma | Severidad |
|----|-------------|-------|-----------|
| COL-031 | ¿El sitio publica la información mínima requerida por Ley de Transparencia? | Ley 1712/2014 | 🔴 Crítico |
| COL-032 | ¿Se usa el dominio .gov.co para entidades gubernamentales? | Decreto 2504/2014 | 🟡 Importante |

---

## Tabla de Cumplimiento por Tipo de Proyecto

| Tipo de Proyecto | Normas Obligatorias | Normas Recomendadas |
|-----------------|--------------------|--------------------|
| SaaS B2B/B2C | Ley 1581, Ley 1273, Ley 527 | Ley 1618 |
| E-commerce | Ley 527, Ley 1480, Ley 1581, Ley 1273 | Ley 1618 |
| Fintech / App financiera | Ley 1266, SFC Circular 007, Ley 1581, Ley 1273 | Ley 527 |
| App de salud | Ley 1581, Res. 1995/1999, Ley 1273 | Ley 1618 |
| Entidad pública | Ley 1712, Ley 1581, Res. 1519, Ley 1618 | Ley 527 |
| API / Backend interno | Ley 1273, Ley 1581 (si hay datos) | — |

---

## Recursos Oficiales

- **SIC (Superintendencia de Industria y Comercio):** [sic.gov.co](https://www.sic.gov.co)
- **RNBD (Registro Nacional de Bases de Datos):** [rnbd.sic.gov.co](https://rnbd.sic.gov.co)
- **MinTIC — Normativa TIC:** [mintic.gov.co](https://www.mintic.gov.co)
- **Secretaría del Senado (leyes):** [secretariasenado.gov.co](http://www.secretariasenado.gov.co)

---

## RADAR NORMATIVO 2025-2026 (proyectos de ley EN TRÁMITE — monitorear, aún NO exigibles)

| Iniciativa | Estado | Impacto esperado en software |
|-----------|--------|------------------------------|
| **PL 043/2025 Senado — Regulación de IA** | Radicado jul-2025 (MinCiencias) | Desarrollo ético/responsable de IA: clasificación de riesgo, transparencia algorítmica. Si el proyecto usa IA (bots, scoring, perfilado), documentar desde ya: propósito del modelo, datos de entrenamiento, supervisión humana |
| **PL Estatutario 214/2025C — Reforma Ley 1581** | Radicado 2025 (SIC + MinCiencias) | Moderniza habeas data hacia estándares GDPR: algoritmos explicables y sin sesgo, intervención humana en decisiones significativas, derechos reforzados del titular |
| **Doc. SIC: datos personales + IA** | Publicado | Insumos técnicos del regulador — anticipa criterios de fiscalización sobre tratamiento de datos con IA |

**Recomendación práctica:** diseñar hoy con el estándar de mañana — consent management granular,
registro de decisiones automatizadas, y capacidad de explicar cualquier output de IA que afecte al titular.
