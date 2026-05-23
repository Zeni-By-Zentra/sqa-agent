# Code Review Checklist — ISO/IEC 25010:2023 + OWASP

Aplica este checklist al código recibido. Reporta hallazgos con severidad 🔴🟡🟢 y cita la norma.

## 1. FUNCIONALIDAD (ISO/IEC 25010 §4.1 — Functional Suitability)

### 1.1 Completitud Funcional
- [ ] La función hace lo que su nombre promete
- [ ] Cubre todos los casos de uso descritos o implícitos
- [ ] Maneja valores límite (cero, nulo, vacío, máximo)

### 1.2 Corrección Funcional
- [ ] Los cálculos producen resultados correctos
- [ ] La lógica de negocio es precisa
- [ ] Los tipos de datos son apropiados (int vs float, tipos de fecha)

### 1.3 Pertinencia Funcional
- [ ] No existe código muerto o features no utilizadas
- [ ] No hay lógica duplicada que debería unificarse (DRY)

## 2. SEGURIDAD (ISO/IEC 25010 §4.5 — Security + OWASP Top 10:2021)

### 2.1 OWASP A01 — Broken Access Control
- [ ] Las rutas/endpoints verifican autorización antes de ejecutar
- [ ] Los IDs expuestos en URL son opacos (UUID) o no predecibles
- [ ] Existe validación de ownership (usuario accede solo a sus datos)
- [ ] Los métodos HTTP son restrictivos (GET no modifica datos)

### 2.2 OWASP A02 — Cryptographic Failures
- [ ] Los datos sensibles nunca viajan en texto plano
- [ ] Los passwords usan bcrypt/argon2 con salt (NO MD5, NO SHA1 sin salt)
- [ ] Las claves API/secrets están en variables de entorno, NO hardcodeadas
- [ ] HTTPS forzado, cookies con Secure + HttpOnly + SameSite

### 2.3 OWASP A03 — Injection
- [ ] Las queries SQL usan parámetros preparados (NUNCA concatenación de strings)
- [ ] Los inputs del usuario se sanitizan antes de usar en comandos del sistema
- [ ] Las salidas hacia HTML se escapan para prevenir XSS
- [ ] Los templates escapan variables por defecto

### 2.4 OWASP A04 — Insecure Design
- [ ] Rate limiting en endpoints críticos (login, OTP, reset password)
- [ ] Los mensajes de error no revelan información interna (stack trace, rutas)
- [ ] Los logs no registran datos sensibles (passwords, tokens, PAN)

### 2.5 OWASP A05 — Security Misconfiguration
- [ ] Endpoints de debug deshabilitados en producción
- [ ] Headers de seguridad presentes (CORS restrictivo, CSP, X-Frame-Options)
- [ ] Dependencias actualizadas sin CVEs críticos conocidos

### 2.6 OWASP A07 — Auth Failures
- [ ] Los tokens JWT tienen expiración razonable
- [ ] El logout invalida el token en el servidor
- [ ] Los intentos fallidos de login tienen límite y bloqueo temporal
- [ ] La recuperación de contraseña no revela si el email existe

### 2.7 OWASP A08 — Software and Data Integrity
- [ ] Los webhooks verifican la firma HMAC del emisor

## 3. CONFIABILIDAD (ISO/IEC 25010 §4.2 — Reliability)

- [ ] Los errores se capturan y manejan explícitamente (no catch vacío)
- [ ] Las operaciones críticas tienen rollback en caso de fallo
- [ ] Los timeouts están definidos en llamadas externas (HTTP, DB)
- [ ] Las conexiones DB/APIs externas tienen retry con backoff
- [ ] Los recursos (conexiones DB, file handles) se liberan siempre
- [ ] Los fallos se loguean con contexto suficiente para debug

## 4. MANTENIBILIDAD (ISO/IEC 25010 §4.6 — Maintainability)

- [ ] Las funciones tienen una sola responsabilidad (SRP — SOLID)
- [ ] Las funciones tienen menos de 50 líneas (alerta en >100)
- [ ] Los nombres de variables/funciones son descriptivos
- [ ] La lógica compleja tiene comentarios explicando el WHY
- [ ] Las funciones son puras o tienen dependencias inyectadas
- [ ] Existe cobertura de tests para lógica de negocio crítica
- [ ] Las constantes están definidas como constantes, no magic numbers

## 5. EFICIENCIA (ISO/IEC 25010 §4.3 — Performance Efficiency)

- [ ] Las consultas a DB tienen índices en campos de búsqueda frecuente
- [ ] Se evita el problema N+1 queries
- [ ] Las operaciones costosas están fuera de loops
- [ ] Todos los endpoints de lista tienen paginación (NUNCA SELECT * sin LIMIT)
- [ ] Los archivos/imágenes grandes se procesan en streams

## FORMATO DE REPORTE

## Code Review Report — [nombre del módulo]
**Norma base:** ISO/IEC 25010:2023 + OWASP Top 10:2021
**Archivos revisados:** [lista]

### Hallazgos

🔴 [ID-001] **[Título breve]**
- **Norma:** OWASP A03:2021 — Injection
- **Archivo:** src/api/users.ts:45
- **Descripción:** [qué está mal y por qué es un riesgo]
- **Evidencia:** [código específico problemático]
- **Corrección:** [cómo arreglarlo]

### Resumen
| Severidad | Cantidad |
|-----------|----------|
| 🔴 Críticos | N |
| 🟡 Importantes | N |
| 🟢 Sugerencias | N |

**Veredicto:** BLOQUEADO / APROBADO CON OBSERVACIONES / APROBADO
