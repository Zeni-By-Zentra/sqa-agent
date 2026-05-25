# AI-Generated Code Audit — Seguridad y Calidad

## Activación
Este checklist se activa cuando el usuario menciona:
- "código de Copilot"
- "código de ChatGPT"
- "código generado por IA"
- "vibe coding"
- "Cursor"
- "Claude generó"
- "--ai-audit"

## Por qué auditar código de IA
Los modelos de lenguaje generan código que puede contener patrones inseguros de datos de entrenamiento desactualizados, no conoce el contexto de seguridad del proyecto, puede introducir dependencias vulnerables, y genera código que "parece correcto" pero tiene bugs sutiles.

## Checklist de Auditoría

### Seguridad — OWASP Top 10 en código IA
| ID | Verificación | Riesgo | Severidad |
|----|-------------|--------|-----------|
| AI-001 | ¿Hay credenciales hardcodeadas? | Exposición de secretos | 🔴 Crítico |
| AI-002 | ¿Las queries SQL usan parámetros (no concatenación)? | SQLi | 🔴 Crítico |
| AI-003 | ¿El input del usuario se sanitiza antes de renderizar? | XSS | 🔴 Crítico |
| AI-004 | ¿Los JWT tienen expiración y algoritmo seguro? | AuthN bypass | 🔴 Crítico |
| AI-005 | ¿Las dependencias están actualizadas (última versión estable)? | Supply chain | 🟡 Importante |
| AI-006 | ¿Se usa HTTPS/TLS para todas las comunicaciones? | MitM | 🔴 Crítico |
| AI-007 | ¿Hay manejo de errores que no exponga información interna? | Info disclosure | 🟡 Importante |
| AI-008 | ¿Los archivos subidos tienen validación de tipo y tamaño? | File upload | 🔴 Crítico |
| AI-009 | ¿Se usa CORS de forma restrictiva (no wildcard *)? | CORS abuse | 🟡 Importante |
| AI-010 | ¿Hay rate limiting en endpoints sensibles? | DoS/BF | 🟡 Importante |

### Calidad de Código
| ID | Verificación | Severidad |
|----|-------------|-----------|
| AI-011 | ¿Hay código duplicado > 30 líneas? | 🟡 Importante |
| AI-012 | ¿Las funciones tienen > 50 líneas? (God functions) | 🟡 Importante |
| AI-013 | ¿Hay manejo de errores en todas las llamadas async? | 🔴 Crítico |
| AI-014 | ¿Los tipos son explícitos (no any en TypeScript)? | 🟡 Importante |
| AI-015 | ¿Hay tests para el código generado? | 🔴 Crítico |
| AI-016 | ¿El código tiene comentarios que expliquen el "por qué"? | 🟢 Sugerencia |

### Patrones Anti-IA Comunes
| ID | Patrón a detectar | Riesgo | Severidad |
|----|------------------|--------|-----------|
| AI-017 | eval() con input del usuario | RCE | 🔴 Crítico |
| AI-018 | innerHTML = con datos externos | XSS | 🔴 Crítico |
| AI-019 | Math.random() para tokens/ids de seguridad | Insecure randomness | 🔴 Crítico |
| AI-020 | Contraseñas en MD5 o SHA1 sin sal | Weak hashing | 🔴 Crítico |
| AI-021 | SELECT * en queries de producción | Over-fetching | 🟡 Importante |
| AI-022 | Catch vacío catch(e) {} sin logging | Silent failure | 🟡 Importante |
| AI-023 | Variables globales mutables | Race condition | 🟡 Importante |
| AI-024 | setTimeout/setInterval sin clearTimeout | Memory leak | 🟡 Importante |

### Licencias y Atribución
| ID | Verificación | Severidad |
|----|-------------|-----------|
| AI-025 | ¿El código generado reproduce fragmentos de proyectos con licencia restrictiva? | 🟡 Importante |
| AI-026 | ¿Las dependencias añadidas son compatibles con la licencia del proyecto? | 🔴 Crítico |
