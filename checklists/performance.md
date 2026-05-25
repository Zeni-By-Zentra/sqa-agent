# Performance Checklist — SQA Agent v4.0

> Normas: ISO/IEC 25010:2023 §4.2.1 (Performance Efficiency) · Web Vitals (Google) · RAIL Model · HTTP/2 best practices
> Aplica análisis estático siempre. Aplica medición dinámica solo si se proporciona URL.

---

## 🔴 Crítico

### [PERF-001] N+1 queries en bucles
- **Condición:** Hay llamadas a DB dentro de `for`, `forEach`, `map`, o `.then()` anidado
- **Norma:** ISO/IEC 25010:2023 §4.2.1 — Time behaviour
- **Impacto:** O(n) queries → latencia exponencial con volumen
- **Corrección:** Usar JOIN, subconsulta o batch load. En ORMs: eager loading (`include`/`with`)
- **[AUTO-FIXABLE]:** No — requiere reestructurar la lógica de consulta

### [PERF-002] Falta de índices en columnas de filtrado frecuente
- **Condición:** Columnas usadas en `WHERE`, `JOIN ON`, `ORDER BY` sin índice declarado
- **Norma:** ISO/IEC 25010:2023 §4.2.1 · PostgreSQL/MySQL index best practices
- **Impacto:** Full table scan en cada query → lentitud crítica con >10k filas
- **Corrección:** `CREATE INDEX CONCURRENTLY idx_table_col ON table(col);`
- **[AUTO-FIXABLE]:** No — requiere migración de DB

### [PERF-003] Operaciones síncronas bloqueantes en event loop (Node.js)
- **Condición:** `fs.readFileSync`, `execSync`, `JSON.parse` de archivos grandes, criptografía síncrona en request handler
- **Norma:** Node.js Event Loop best practices · ISO/IEC 25010:2023 §4.2.2 (Resource utilisation)
- **Impacto:** Bloquea el event loop — toda latencia de un request afecta a todos los usuarios simultáneos
- **Corrección:** Reemplazar con versiones async (`fs.readFile`, `exec`, Worker threads para CPU-bound)
- **[AUTO-FIXABLE]:** Parcial — renombrar sync→async es mecánico, pero requiere manejar Promise correctamente

### [PERF-004] Respuesta sin caché en endpoints de alta frecuencia
- **Condición:** Endpoints consultados >100 veces/min que calculan lo mismo retornan sin headers de caché
- **Norma:** HTTP/1.1 RFC 7234 · Web Vitals TTFB
- **Impacto:** Carga innecesaria en DB y server
- **Corrección:** Añadir `Cache-Control: public, max-age=60` o implementar caché en memoria (Redis/LRU)
- **[AUTO-FIXABLE]:** Parcial — añadir headers es automático, la lógica de TTL es manual

### [PERF-005] LCP > 2.5s o FID > 100ms (si se mide dinámicamente)
- **Condición:** Largest Contentful Paint supera 2.5s o First Input Delay supera 100ms
- **Norma:** Google Core Web Vitals 2024 · ISO/IEC 25010:2023 §4.2.1
- **Impacto:** Penalización SEO + alta tasa de abandono (53% de usuarios abandona si >3s)
- **Corrección:** Lazy loading, compresión de imágenes, eliminación de render-blocking resources

---

## 🟡 Importante

### [PERF-006] Imágenes sin formato WebP/AVIF o sin lazy loading
- **Condición:** Tags `<img>` o `next/image` sin `loading="lazy"`, sin WebP, o sin `width/height` declarados
- **Norma:** Web Vitals CLS · HTTP Archive Web Almanac
- **Impacto:** CLS (Cumulative Layout Shift) + payload innecesario en conexiones lentas
- **Corrección:** `<img loading="lazy" decoding="async">` + convertir a WebP. En Next.js usar `<Image>` de `next/image`.
- **[AUTO-FIXABLE]:** Sí — añadir atributos `loading="lazy"` y `decoding="async"` en imgs sin ellos

### [PERF-007] Bundle JavaScript > 250KB sin code splitting
- **Condición:** Bundle principal supera 250KB gzipped sin chunks separados
- **Norma:** Web Vitals TBT · Google Lighthouse performance budget
- **Impacto:** TTI (Time to Interactive) elevado en conexiones 4G
- **Corrección:** Dynamic imports (`import()`), React.lazy, Next.js automatic code splitting

### [PERF-008] Consultas sin LIMIT en paginación
- **Condición:** Endpoints que devuelven listas sin `LIMIT`/`OFFSET` o cursor-based pagination
- **Norma:** ISO/IEC 25010:2023 §4.2.2
- **Impacto:** Una tabla grande puede saturar memoria y red con una sola request
- **Corrección:** Añadir `LIMIT $1 OFFSET $2` con valor máximo configurable (ej: 100)
- **[AUTO-FIXABLE]:** No — requiere diseño del esquema de paginación

### [PERF-009] Queries sin timeout declarado
- **Condición:** Conexiones a DB sin `statement_timeout` o queries sin `AbortController`/timeout en fetch
- **Norma:** ISO/IEC 25010:2023 §4.2.1 — Time behaviour
- **Impacto:** Una query lenta puede mantener conexiones abiertas indefinidamente, agotando el pool
- **Corrección:** Configurar `statement_timeout` en PostgreSQL · `AbortController` en fetch externas
- **[AUTO-FIXABLE]:** Parcial — añadir AbortController es mecánico si el patrón ya está en el codebase

### [PERF-010] Falta de connection pooling
- **Condición:** Se crea una nueva conexión DB por request en lugar de reutilizar un pool
- **Norma:** ISO/IEC 25010:2023 §4.2.2 (Resource utilisation)
- **Impacto:** Latencia de setup de conexión en cada request (~5-30ms) + límite de conexiones de PostgreSQL
- **Corrección:** Usar `pg.Pool` (node-postgres), `PrismaClient` singleton, o `Drizzle` con pool configurado

### [PERF-011] Assets estáticos sin compresión gzip/brotli
- **Condición:** Respuestas de texto (JS, CSS, HTML, JSON) sin `Content-Encoding: gzip` o `br`
- **Norma:** HTTP/2 best practices · Web Almanac 2023
- **Impacto:** Transferencia de datos 3-10x mayor
- **Corrección:** Activar gzip/brotli en nginx/CDN. En Next.js: automático si se usa `next start` con Vercel/Nginx

---

## 🟢 Sugerencia

### [PERF-012] Falta de memoización en cálculos costosos repetidos
- **Condición:** Funciones puras costosas (transformaciones, cálculos matemáticos) llamadas repetidamente con los mismos parámetros
- **Norma:** ISO/IEC 25010:2023 §4.2.1
- **Corrección:** `useMemo`, `useCallback` (React) · Memoize con Map como caché (Node.js)

### [PERF-013] Falta de índices compuestos para queries multi-columna
- **Condición:** Queries frecuentes con `WHERE col_a = X AND col_b = Y` y solo índices individuales
- **Norma:** PostgreSQL documentation — composite indexes
- **Corrección:** `CREATE INDEX CONCURRENTLY idx_table_a_b ON table(col_a, col_b);`

### [PERF-014] Re-renders innecesarios en componentes React
- **Condición:** Componentes que se re-renderizan cuando sus props no cambian (verificable con React DevTools Profiler)
- **Norma:** React performance best practices
- **Corrección:** `React.memo`, dividir estado, colocación del estado cerca del consumidor

---

## Medición dinámica (solo si se proporciona URL)

Si el input incluye una URL válida, ejecutar 3 mediciones HTTP con WebFetch e informar:

```
## Medición de respuesta real — [URL]

| Medición | Tiempo total | Status | Tamaño |
|----------|-------------|--------|--------|
| #1       | XXXms       | 200    | XXkB   |
| #2       | XXXms       | 200    | XXkB   |
| #3       | XXXms       | 200    | XXkB   |
| **Promedio** | **XXXms** | — | **XXkB** |

**Baseline:** < 200ms excelente · 200-500ms aceptable · 500ms-1s mejorar · > 1s crítico
**Headers de caché presentes:** [Sí / No — Cache-Control: ...]
**Compresión activa:** [Sí/No — Content-Encoding: ...]
```

> Nota: WebFetch mide el tiempo de respuesta total desde la perspectiva del agente, no del usuario final. Para métricas de usuario real usar Lighthouse, WebPageTest o Real User Monitoring (RUM).
