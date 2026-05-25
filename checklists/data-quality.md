# Data Quality Assessment — ISO/IEC 25012

## Activación
Este checklist se activa cuando el usuario menciona:
- "calidad de datos"
- "data quality"
- "datos maestros"
- "limpieza de datos"
- "migración de datos"
- "ETL"

## Normas Aplicadas
- ISO/IEC 25012:2008 — Data Quality Model
- ISO/IEC 25024:2015 — Measurement of Data Quality
- Ley 1581/2012 — Protección de Datos Personales (Colombia)
- Ley 1266/2008 — Habeas Data Financiero (Colombia)

## Dimensiones de Calidad de Datos

### 1. Completitud (Completeness)
| ID | Métrica | Criterio | Severidad |
|----|---------|----------|-----------|
| DQ-001 | % de registros sin valores nulos | >= 99% | 🔴 Crítico |
| DQ-002 | % de columnas obligatorias pobladas | 100% | 🔴 Crítico |
| DQ-003 | Migraciones sin pérdida de datos | 100% registros migrados | 🔴 Crítico |

### 2. Exactitud (Accuracy)
| ID | Métrica | Criterio | Severidad |
|----|---------|----------|-----------|
| DQ-004 | % de registros que pasan reglas de negocio | >= 99.5% | 🔴 Crítico |
| DQ-005 | Formato de email válido | 100% regex RFC 5322 | 🟡 Importante |
| DQ-006 | NIT/Cédula válidos (Colombia) | 100% validación DIAN/Registraduría | 🟡 Importante |

### 3. Consistencia (Consistency)
| ID | Métrica | Criterio | Severidad |
|----|---------|----------|-----------|
| DQ-007 | Claves foráneas íntegras | 100% FK resueltas | 🔴 Crítico |
| DQ-008 | Unidades de medida uniformes | 100% misma unidad | 🟡 Importante |
| DQ-009 | Formatos de fecha consistentes | 100% ISO 8601 o definido | 🟡 Importante |

### 4. Actualidad (Timeliness)
| ID | Métrica | Criterio | Severidad |
|----|---------|----------|-----------|
| DQ-010 | Tiempo desde captura hasta disponibilidad | <= 5 min (real-time), <= 24h (batch) | 🟡 Importante |
| DQ-011 | Fecha de última actualización visible | 100% registros con timestamp | 🟢 Sugerencia |

### 5. Unicidad (Uniqueness)
| ID | Métrica | Criterio | Severidad |
|----|---------|----------|-----------|
| DQ-012 | % de registros duplicados | <= 0.1% | 🔴 Crítico |
| DQ-013 | Constraint UNIQUE en columnas críticas | 100% implementado | 🔴 Crítico |

### 6. Accesibilidad y Protección (Colombia — Ley 1581/2012)
| ID | Métrica | Criterio | Severidad |
|----|---------|----------|-----------|
| DQ-014 | Permisos de acceso por rol | 100% RBAC implementado | 🔴 Crítico |
| DQ-015 | Cifrado de datos sensibles (Ley 1581/2012) | AES-256 o superior | 🔴 Crítico |
| DQ-016 | Logs de acceso a datos sensibles | 100% auditoría de consultas | 🟡 Importante |
| DQ-017 | Política de retención de datos definida | Documentada y aplicada | 🔴 Crítico |
| DQ-018 | Datos personales anonimizables bajo petición | Implementado | 🔴 Crítico |
