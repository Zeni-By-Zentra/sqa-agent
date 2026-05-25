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
