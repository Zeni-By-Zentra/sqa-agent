# API Quality Checklist — REST Best Practices + ISO/IEC 25010

## 1. DISEÑO DE ENDPOINTS (Richardson REST Maturity Model)

- [ ] URLs usan sustantivos en plural, no verbos (/users no /getUsers)
- [ ] Recursos anidados reflejan relaciones (/users/{id}/orders)
- [ ] Sin extensiones de archivo en URLs
- [ ] Palabras separadas con guion (/user-profiles no /user_profiles)
- [ ] Versión en URL o header (/api/v1/ o Accept header)
- [ ] GET es idempotente sin side effects
- [ ] POST crea recurso
- [ ] PUT reemplaza recurso completo (idempotente)
- [ ] PATCH actualiza parcialmente
- [ ] DELETE es idempotente (204 si existe, 404 si no)

## 2. CÓDIGOS DE ESTADO HTTP

- [ ] 200 OK con body
- [ ] 201 Created con Location header
- [ ] 204 No Content para DELETE/PUT sin body
- [ ] 400 Bad Request con detalle del error
- [ ] 401 Unauthorized (no autenticado)
- [ ] 403 Forbidden (autenticado sin permisos)
- [ ] 404 Not Found
- [ ] 409 Conflict para estado inconsistente
- [ ] 422 Unprocessable Entity para validación semántica
- [ ] 429 Too Many Requests con Retry-After header
- [ ] 500 sin stack trace expuesto

## 3. REQUEST/RESPONSE

- [ ] Content-Type: application/json en requests con body
- [ ] Validación de campos requeridos con mensajes descriptivos
- [ ] Validación de tipos (número, email, UUID, fecha ISO 8601)
- [ ] Estructura consistente en todos los endpoints (data + meta o error + message)
- [ ] Fechas en ISO 8601 (2024-01-15T10:30:00Z)
- [ ] IDs como UUID strings, no integers secuenciales
- [ ] Paginación con page, per_page, total, total_pages
- [ ] NUNCA arrays en nivel raíz del response

## 4. RENDIMIENTO (ISO/IEC 25010 §4.3)

- [ ] Todos los endpoints de lista tienen paginación obligatoria
- [ ] per_page máximo configurado
- [ ] Filtros disponibles con índices en DB
- [ ] Cache-Control headers apropiados
- [ ] ETags o Last-Modified para recursos estáticos
- [ ] P95 < 200ms para lectura simple
- [ ] P95 < 500ms para lógica de negocio
- [ ] Timeouts configurados y documentados

## 5. DOCUMENTACIÓN

- [ ] OpenAPI/Swagger actualizado y sincronizado
- [ ] Ejemplos de request/response en cada endpoint
- [ ] Errores posibles documentados por endpoint
- [ ] Guía de autenticación con ejemplos funcionales
- [ ] Changelog con versiones y breaking changes
- [ ] Colección Postman/Insomnia disponible

## 6. SEGURIDAD API

- [ ] Rate limiting por endpoint
- [ ] CORS para orígenes específicos
- [ ] Tokens de autorización en header (nunca en URL)
- [ ] Webhooks verifican firma HMAC
- [ ] GraphQL: depth limiting, query complexity, introspection deshabilitado en prod

## FORMATO DE REPORTE

## API Quality Report — [nombre de la API]
**Estándar:** REST Maturity Level [1-3], ISO/IEC 25010:2023
**Endpoints revisados:** [lista]

### Hallazgos por Categoría

#### Diseño de Endpoints
🔴/🟡/🟢 [hallazgo + norma]

#### Seguridad
🔴/🟡/🟢 [hallazgo + norma]

#### Rendimiento
🔴/🟡/🟢 [hallazgo + norma]

### Nivel de Madurez REST: Richardson Level [0/1/2/3]
### Veredicto: BLOQUEADO / APROBADO CON OBSERVACIONES / APROBADO
