# Quality in Use Assessment — ISO 9126-4 / ISO 25010

## Activación
Este checklist se activa cuando el usuario menciona:
- "calidad en uso"
- "experiencia de usuario"
- "usabilidad"
- "satisfacción del usuario"
- "efectividad"
- "productividad"

## Normas Aplicadas
- ISO/IEC 9126-4:2004 — Quality in Use Metrics
- ISO/IEC 25010:2023 §4.1.4 — Quality in Use Model
- ISO/IEC 25022:2016 — Measurement of Quality in Use

## Dimensiones de Calidad en Uso

### 1. Efectividad (Effectiveness)
| ID | Criterio | Métrica | Severidad |
|----|---------|---------|-----------|
| QU-001 | Los usuarios completan tareas principales sin error | ≥ 95% tasa de éxito | 🔴 Crítico |
| QU-002 | Flujo de onboarding completable sin documentación | Test con usuario nuevo | 🔴 Crítico |
| QU-003 | Mensajes de error son accionables | 100% de errores | 🟡 Importante |
| QU-004 | Formularios con validación en línea | Antes de submit | 🟡 Importante |

### 2. Eficiencia (Efficiency)
| ID | Criterio | Métrica | Severidad |
|----|---------|---------|-----------|
| QU-005 | Tarea principal completable en ≤ 3 clics/pasos | Test de flujo principal | 🟡 Importante |
| QU-006 | Tiempo de respuesta percibido | ≤ 3 segundos (RAIL model) | 🔴 Crítico |
| QU-007 | Atajos de teclado para operaciones frecuentes | Documentados | 🟢 Sugerencia |

### 3. Satisfacción (Satisfaction)
| ID | Criterio | Métrica | Severidad |
|----|---------|---------|-----------|
| QU-008 | Net Promoter Score (NPS) | ≥ 40 | 🟡 Importante |
| QU-009 | System Usability Scale (SUS) | ≥ 68 (percentil 50) | 🟡 Importante |
| QU-010 | Tasa de abandono en flujo principal | ≤ 20% | 🔴 Crítico |

### 4. Libertad de riesgo (Freedom from Risk)
| ID | Criterio | Métrica | Severidad |
|----|---------|---------|-----------|
| QU-011 | Confirmación antes de acciones destructivas | 100% | 🔴 Crítico |
| QU-012 | Estado de la aplicación recuperable tras error | 100% | 🔴 Crítico |
| QU-013 | Datos del usuario no se pierden en fallos de red | Implementar autosave | 🟡 Importante |

### 5. Contexto de cobertura (Context Coverage)
| ID | Criterio | Métrica | Severidad |
|----|---------|---------|-----------|
| QU-014 | Soporte para mobile (responsive) | 100% funcionalidad | 🔴 Crítico |
| QU-015 | Funciona en modo offline parcial | Operaciones básicas | 🟢 Sugerencia |
| QU-016 | Accesibilidad WCAG 2.1 AA | 0 violaciones críticas | 🔴 Crítico |

## Herramientas Sugeridas
- Hotjar / FullStory: Mapas de calor y grabaciones de sesión
- Google Analytics: Funnels de conversión y abandono
- Maze / Useberry: Tests de usabilidad moderados
- axe DevTools: Auditoría de accesibilidad automatizada
