# Performance Assessment — ISO/IEC 25010

## Normas Aplicadas
- ISO/IEC 25010:2023 §4.2.1 — Performance Efficiency
- Google Web Vitals 2024
- RAIL Model (Response, Animation, Idle, Load)

## Métricas Web (Google Web Vitals)
| ID | Métrica | Criterio Bueno | Criterio Pobre | Severidad |
|----|---------|----------------|----------------|-----------|
| PERF-001 | LCP (Largest Contentful Paint) | <= 2.5s | > 4.0s | 🔴 Crítico |
| PERF-002 | INP (Interaction to Next Paint) | <= 200ms | > 500ms | 🔴 Crítico |
| PERF-003 | CLS (Cumulative Layout Shift) | <= 0.1 | > 0.25 | 🟡 Importante |
| PERF-004 | TTFB (Time to First Byte) | <= 800ms | > 1800ms | 🟡 Importante |
| PERF-005 | FCP (First Contentful Paint) | <= 1.8s | > 3.0s | 🟡 Importante |

## Métricas Backend
| ID | Métrica | Criterio | Severidad |
|----|---------|----------|-----------|
| PERF-006 | Tiempo de respuesta API p95 | <= 200ms | 🔴 Crítico |
| PERF-007 | Tiempo de respuesta API p99 | <= 500ms | 🟡 Importante |
| PERF-008 | Conexiones a BD por request | <= 3 | 🟡 Importante |
| PERF-009 | N+1 queries detectadas | 0 | 🔴 Crítico |
| PERF-010 | Índices faltantes | 0 | 🔴 Crítico |
| PERF-011 | Connection pooling configurado | Sí | 🟡 Importante |
| PERF-012 | Cache implementado (Redis/Memcached) | Recomendado | 🟢 Sugerencia |
| PERF-013 | Compresión gzip/brotli | Habilitada | 🟢 Sugerencia |

## Métricas Móviles
| ID | Métrica | Criterio | Severidad |
|----|---------|----------|-----------|
| PERF-014 | Cold start time | <= 2s | 🔴 Crítico |
| PERF-015 | Warm start time | <= 1s | 🟡 Importante |
| PERF-016 | Consumo de batería por hora | <= 5% | 🟡 Importante |
| PERF-017 | Tamaño de la app | <= 50 MB (Android), <= 100 MB (iOS) | 🟢 Sugerencia |

## Métricas de Base de Datos
| ID | Métrica | Criterio | Severidad |
|----|---------|----------|-----------|
| PERF-018 | Tiempo de query más lenta (p99) | <= 1s | 🔴 Crítico |
| PERF-019 | Uso de índices en queries frecuentes | EXPLAIN ANALYZE = Index Scan | 🔴 Crítico |
| PERF-020 | Tamaño de tabla sin particionado | <= 10M registros | 🟡 Importante |
| PERF-021 | Vacuuming/Autovacuum configurado | Sí (PostgreSQL) | 🟡 Importante |

---

## PRESUPUESTO DE PERFORMANCE (Performance Budget)

> Acordar el presupuesto en `--plan` y verificarlo en cada deploy. Un presupuesto no medido es una aspiración.

| Recurso | Presupuesto | Severidad si excede |
|---------|-------------|---------------------|
| JS inicial (gzipped) | ≤ 170 KB | 🔴 |
| CSS inicial (gzipped) | ≤ 60 KB | 🟡 |
| Peso total de la página (sobre el fold) | ≤ 1 MB | 🟡 |
| Imagen individual | ≤ 200 KB (WebP/AVIF) | 🟡 |
| Fuentes web | ≤ 2 familias, ≤ 100 KB, `font-display: swap` + preload | 🟡 |
| Requests en el critical path | ≤ 50 | 🟢 |
| Time to Interactive (4G, mid-tier mobile) | ≤ 3.5 s | 🔴 |

---

## AUDITORÍA WEB (modo --pagespeed)

### Frontend / Next.js / React-Vite
- [ ] Bundle analizado (`@next/bundle-analyzer` / `rollup-plugin-visualizer`); sin dependencias pesadas innecesarias (moment→date-fns, lodash completo→imports puntuales)
- [ ] Code splitting por ruta; `dynamic()`/`React.lazy` para componentes pesados below-the-fold
- [ ] `next/image` (o equivalente) con `width`/`height` para evitar CLS; formatos WebP/AVIF; lazy loading
- [ ] Fuentes con `next/font` o `font-display: swap` + `preload` de la fuente del LCP
- [ ] Sin render-blocking: CSS crítico inline, JS no crítico con `defer`/`async`
- [ ] Server Components / SSG / ISR usados donde aplica (no enviar JS que no se necesita)
- [ ] Sin `useEffect` fetch en cascada que retrase el contenido principal
- [ ] Third-party scripts (analytics, chat widget) con `next/script strategy="lazyOnload"` o diferidos
- [ ] `prefers-reduced-motion` respetado (también ayuda a INP)

### Backend / API
- [ ] **N+1 queries = 0** (auditar loops que consultan la BD por iteración)
- [ ] Índices presentes en columnas de WHERE/JOIN/ORDER frecuentes (`EXPLAIN ANALYZE` = Index Scan, no Seq Scan)
- [ ] Connection pooling configurado con límites
- [ ] Respuestas paginadas (cursor) para listas grandes; no devolver miles de filas
- [ ] Cache (Redis/in-memory) para datos costosos y poco cambiantes
- [ ] `statement_timeout` para evitar queries que cuelgan el pool
- [ ] Compresión gzip/brotli activa en Nginx para JSON/text

### CDN / caching / red
- [ ] Cloudflare cacheando assets estáticos; assets con hash → `Cache-Control: immutable`
- [ ] HTML/API autenticada con `no-store` (no cachear datos privados)
- [ ] HTTP/2 o HTTP/3 activo
- [ ] Imágenes servidas vía CDN con transformación/resize cuando aplica

### Medición (con URL)
- [ ] Medición real con WebFetch: status, tiempo total, tamaño — 3 repeticiones, promedio + desviación
- [ ] Recomendar complementar con **PageSpeed Insights** (datos de campo CrUX) y **Lighthouse** (laboratorio)
- [ ] Separar SIEMPRE en el reporte: dato de **laboratorio** vs **campo** vs **inferido**. Nunca presentar inferencia como medición.
