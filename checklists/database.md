# Database Quality Checklist — ISO/IEC 25010 + PostgreSQL Best Practices

## 1. DISEÑO DE ESQUEMA (ISO/IEC 25010 §4.6 — Maintainability)

### 1.1 Normalización
- [ ] Las tablas están en 3NF mínimo (sin dependencias transitivas)
- [ ] Datos no repetidos innecesariamente entre tablas (DRY aplicado a datos)
- [ ] Las columnas almacenan un único valor atómico (sin JSON embedido para datos relacionales)
- [ ] Excepciones a la normalización están justificadas por performance con comentario en schema

### 1.2 Nombrado y Convenciones
- [ ] Tablas en snake_case y plural (`users`, no `User`, no `tbl_user`)
- [ ] Columnas en snake_case descriptivo (`created_at`, no `createdAt`, no `dt_cre`)
- [ ] PKs nombradas consistentemente (`id` o `{tabla}_id`)
- [ ] FKs reflejan la tabla que referencian (`{tabla}_id`)
- [ ] Booleanos con prefijo `is_` o `has_` (`is_active`, `has_subscription`)
- [ ] Timestamps siempre `TIMESTAMPTZ`, nunca `TIMESTAMP` sin zona horaria

### 1.3 Tipos de Datos
- [ ] UUIDs para PKs expuestas (no integers secuenciales — OWASP A01)
- [ ] `DECIMAL`/`NUMERIC` para dinero (NUNCA `FLOAT` — error de precisión)
- [ ] `TEXT` para strings de longitud variable (PostgreSQL no penaliza TEXT vs VARCHAR)
- [ ] `JSONB` (no `JSON`) para datos semiestructurados que se consultan
- [ ] `ENUM` de PostgreSQL o tabla de lookup para categorías fijas
- [ ] Columnas `NOT NULL` donde el dato es siempre requerido

### 1.4 Integridad Referencial
- [ ] Todas las FKs tienen constraint `FOREIGN KEY` definido
- [ ] `ON DELETE` especificado explícitamente (CASCADE, RESTRICT, SET NULL — nunca default)
- [ ] `CHECK` constraints para validaciones de dominio (`status IN ('active','trial','cancelled')`)
- [ ] `UNIQUE` constraints en columnas que deben ser únicas (email, slug, etc.)

---

## 2. ÍNDICES Y PERFORMANCE (ISO/IEC 25010 §4.3 — Performance Efficiency)

### 2.1 Índices Necesarios
- [ ] Índice en toda FK (PostgreSQL NO los crea automáticamente)
- [ ] Índice en columnas usadas en `WHERE` frecuentes (email, status, created_at)
- [ ] Índice en columnas usadas en `ORDER BY` para paginación
- [ ] Índices compuestos cuando dos columnas siempre se filtran juntas
- [ ] Índices parciales para subsets frecuentes (`WHERE is_active = true`)

### 2.2 Índices Problemáticos a Evitar
- [ ] Sin índices en columnas de baja cardinalidad como booleanos solos (no ayudan)
- [ ] Sin índices duplicados (mismo set de columnas en distinto orden)
- [ ] Revisado que índices no usados no existan (`pg_stat_user_indexes` para auditar)

### 2.3 Análisis de Queries
- [ ] Las queries críticas ejecutadas con `EXPLAIN ANALYZE` (no Seq Scan en tablas grandes)
- [ ] Problema N+1 identificado y resuelto (JOINs o carga eager)
- [ ] Paginación con `LIMIT+OFFSET` vs cursor-based para datasets >10k filas
- [ ] Queries con JOIN en más de 4 tablas revisadas (posible desnormalización estratégica)

### 2.4 Connection Pool
- [ ] Pool configurado con `max_connections` apropiado para el servidor
- [ ] Pool min/max definido según carga esperada
- [ ] Timeout de conexión configurado (no espera infinita)
- [ ] Conexiones se liberan siempre en `finally` (no connection leaks)

---

## 3. MIGRACIONES (ISO/IEC 25010 §4.6 — Maintainability)

### 3.1 Calidad de Migraciones
- [ ] Cada migración es atómica (todo o nada, envuelta en transacción)
- [ ] Migraciones son reversibles (existe `down` migration para cada `up`)
- [ ] Las migraciones no contienen lógica de negocio (solo cambios de schema)
- [ ] Datos de seed separados de las migraciones de schema
- [ ] Migraciones testeadas en staging antes de producción
- [ ] Nombre de archivo con timestamp y descripción (`20240115_add_users_email_index`)

### 3.2 Migraciones Peligrosas (requieren plan especial)
- [ ] Agregar columna `NOT NULL` en tabla grande → primero nullable, llenar datos, luego NOT NULL
- [ ] Renombrar columna → alias temporal para zero-downtime
- [ ] Eliminar columna → primero dejar de usar en código, luego migración
- [ ] Cambiar tipo de columna → nueva columna + migración de datos + eliminar vieja
- [ ] Agregar índice en tabla grande → `CREATE INDEX CONCURRENTLY` (no bloquea)

### 3.3 Control de Versiones
- [ ] Migraciones en control de versiones junto al código
- [ ] Nunca modificar una migración ya ejecutada en producción
- [ ] Estado de migraciones verificable (tabla de tracking)

---

## 4. SEGURIDAD DE DATOS (OWASP + ISO/IEC 27001 + Ley 1581/2012 Colombia)

### 4.1 Datos Sensibles
- [ ] PII identificada: email, teléfono, documento, dirección (Ley 1581/2012 Colombia)
- [ ] Passwords NUNCA en texto plano — bcrypt/argon2 hash SIEMPRE
- [ ] Tokens y API keys hasheados o cifrados en DB (no texto plano)
- [ ] Datos de tarjetas NO almacenados (PCI DSS — delegar a Wompi/Stripe)
- [ ] Campos sensibles identificados con comentario en schema o documentación

### 4.2 Control de Acceso a DB
- [ ] La app conecta con usuario de mínimo privilegio (no superuser en producción)
- [ ] Usuario de app tiene solo SELECT/INSERT/UPDATE/DELETE — no DROP/CREATE
- [ ] Credenciales de DB en variables de entorno (nunca hardcodeadas)
- [ ] Conexión via SSL/TLS si la DB es remota
- [ ] Acceso directo a DB de producción restringido por IP/VPN

### 4.3 SQL Injection
- [ ] 100% de queries usan parámetros preparados (`$1`, `$2`) o ORM con escape automático
- [ ] CERO concatenación de strings del usuario en queries
- [ ] Búsquedas con `LIKE` usan parámetros, no interpolación (`LIKE $1` con valor `'%texto%'`)

### 4.4 Auditoría
- [ ] Tablas críticas tienen `created_at` y `updated_at TIMESTAMPTZ` con `DEFAULT NOW()`
- [ ] Cambios en datos financieros o de permisos tienen audit log (quién, qué, cuándo)
- [ ] Soft delete (`deleted_at`) para datos que requieren trazabilidad histórica

---

## 5. BACKUP Y RECUPERACIÓN (ISO/IEC 27001 A.12.3)

- [ ] Backups automáticos configurados (frecuencia apropiada a criticidad)
- [ ] Backups probados periódicamente con restore real (no solo backup sin verificar)
- [ ] Backups almacenados en ubicación separada al servidor principal
- [ ] Backups cifrados si contienen datos personales
- [ ] RTO y RPO definidos y documentados
- [ ] Procedimiento de disaster recovery documentado

---

## FORMATO DE REPORTE DATABASE REVIEW

```
## Database Quality Report — [Proyecto]
**Motor:** PostgreSQL [versión]
**Tablas revisadas:** [lista o "todas"]
**Norma:** ISO/IEC 25010:2023, OWASP, ISO 27001:2022, Ley 1581/2012

### Hallazgos

🔴 [DB-001] **[Título]**
- **Categoría:** Schema / Índices / Migraciones / Seguridad / Backup
- **Tabla/Columna:** users.password
- **Descripción:** [problema y riesgo]
- **Corrección:** [SQL o solución concreta]

### Resumen
| Área | Estado |
|------|--------|
| Diseño de esquema | OK/WARN/FAIL |
| Índices y performance | OK/WARN/FAIL |
| Migraciones | OK/WARN/FAIL |
| Seguridad de datos | OK/WARN/FAIL |
| Backup y recuperación | OK/WARN/FAIL |

**Riesgo de pérdida de datos:** ALTO / MEDIO / BAJO
**Riesgo de performance en escala:** ALTO / MEDIO / BAJO
```
