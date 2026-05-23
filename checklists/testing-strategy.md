# Testing Strategy Checklist — ISO/IEC 25010 + ISTQB Principles

## 1. PIRÁMIDE DE TESTING

### 1.1 Distribución Esperada
```
        /\
       /E2E\          5-10%  (lentos, frágiles, caros)
      /------\
     /Integr. \       20-30% (APIs, DB, servicios)
    /----------\
   / Unit Tests \     60-70% (rápidos, aislados, baratos)
  /--------------\
```
- [ ] La distribución real del proyecto se acerca a la pirámide
- [ ] Si hay más E2E que unitarios → alerta: suite frágil y lenta
- [ ] Si solo hay unitarios sin integración → alerta: falsas garantías

### 1.2 Cobertura Significativa (no solo porcentaje)
- [ ] Los branches/ramas están cubiertos (if/else, switch, ternary)
- [ ] Los edge cases están cubiertos: null, vacío, límites, errores
- [ ] La lógica de negocio crítica tiene cobertura ≥ 80%
- [ ] Los happy paths Y los unhappy paths tienen tests
- [ ] Código de manejo de errores (catch blocks) está testeado
- [ ] La cobertura de líneas NO es la única métrica (puede engañar con código sin ramas)

---

## 2. CALIDAD DE TESTS UNITARIOS

### 2.1 Principio F.I.R.S.T
- [ ] **Fast**: El suite unitario completo corre en < 30 segundos
- [ ] **Isolated**: Cada test es independiente — puede correr en cualquier orden
- [ ] **Repeatable**: Mismo resultado siempre, sin depender de tiempo/red/datos externos
- [ ] **Self-validating**: El test falla o pasa por sí solo — no requiere inspección manual
- [ ] **Timely**: Tests escritos junto al código, no semanas después

### 2.2 Estructura AAA (Arrange-Act-Assert)
- [ ] Cada test tiene sección clara de setup (Arrange)
- [ ] Una sola acción por test (Act) — si hay varios Acts, dividir el test
- [ ] Assertions específicas y descriptivas (Assert)
- [ ] Un test testea UNA SOLA COSA — si el nombre tiene "and", dividirlo

### 2.3 Anti-patrones Detectados
- [ ] Sin lógica condicional en tests (if/else/loops en tests = test no determinista)
- [ ] Sin datos de producción en tests (usar factories/fixtures con datos falsos)
- [ ] Sin `sleep()` o timers reales (mockear el tiempo con fake timers)
- [ ] Sin dependencias en orden de ejecución entre tests
- [ ] Sin tests que siempre pasan (assertions que no pueden fallar)
- [ ] Sin tests comentados — si está comentado, eliminar o arreglar

### 2.4 Calidad de Assertions
- [ ] Las assertions son específicas (`expect(result).toBe(42)`, no `expect(result).toBeTruthy()`)
- [ ] Un test tiene máximo 3-5 assertions (si tiene más, probablemente testea varias cosas)
- [ ] Los mensajes de error de assertion describen qué falló y qué se esperaba
- [ ] Las assertions de objetos verifican campos específicos, no el objeto completo si no es necesario

---

## 3. CALIDAD DE TESTS DE INTEGRACIÓN

### 3.1 Cobertura de Integración
- [ ] Cada endpoint de API tiene al menos un test de integración (happy path)
- [ ] Los flujos críticos de negocio están testeados end-to-end a nivel de API
- [ ] Los errores de DB (constraint violations, duplicados) están testeados
- [ ] La autenticación/autorización está testeada (401, 403, 404 para recursos ajenos)
- [ ] La paginación y filtros están testeados
- [ ] Los webhooks entrantes están testeados con payloads reales

### 3.2 Aislamiento de DB en Tests
- [ ] Cada test corre en transacción y hace rollback al terminar (no contamina otros tests)
- [ ] O usa base de datos de test separada que se resetea entre suites
- [ ] Los tests de integración NO corren contra la DB de producción
- [ ] Los fixtures/seeds de test son deterministas (siempre los mismos datos base)
- [ ] El estado de la DB al inicio de cada test es predecible

### 3.3 Mocks y Stubs
- [ ] Los servicios externos (APIs de terceros, email, SMS, WhatsApp) están mockeados
- [ ] Los mocks reflejan el contrato real del servicio externo (no inventados)
- [ ] No se mockea lo que se quiere testear (error común: mockear la función que se testea)
- [ ] Los mocks se resetean entre tests para no contaminar
- [ ] Los contract tests verifican que los mocks siguen siendo fieles a la API real

---

## 4. CALIDAD DE TESTS E2E

### 4.1 Scope Correcto
- [ ] Los E2E cubren los critical user journeys (registro, pago, flujo principal del producto)
- [ ] Los E2E NO testean variaciones de UI — eso va en unit/integración
- [ ] El número de E2E es el mínimo necesario (cada E2E = deuda de mantenimiento)
- [ ] Los E2E corren en un ambiente idéntico a producción (staging, no localhost)

### 4.2 Anti-flakiness
- [ ] Sin hardcoded waits (`sleep(2000)`) — usar `waitForElement`, `waitForResponse`
- [ ] Sin dependencia en orden de elementos del DOM que puede cambiar
- [ ] Selectores por `data-testid`, no por clases CSS o XPath frágil
- [ ] Los tests limpian sus datos de prueba al finalizar
- [ ] Reintentos automáticos configurados para flakiness de red (máx 2 reintentos)
- [ ] Los E2E tienen timeout global razonable (ej: 30s por test, no 5 minutos)

### 4.3 Reporte de Flaky Tests
- [ ] Existe mecanismo para detectar tests flaky (correr N veces y comparar)
- [ ] Los tests flaky están tracked y tienen owner asignado para arreglo
- [ ] Los tests flaky NO bloquean el pipeline mientras se arreglan (se quarantina temporalmente)

---

## 5. TESTING DE SEGURIDAD

- [ ] Tests de autorización: usuario A no puede acceder a recursos de usuario B
- [ ] Tests de rate limiting: el endpoint bloquea tras N intentos
- [ ] Tests con inputs maliciosos: SQL injection strings, XSS payloads, payloads oversized
- [ ] Tests de tokens expirados/inválidos retornan 401
- [ ] Tests de roles: admin endpoints rechazan usuarios sin rol
- [ ] Tests de boundary: IDs de otros usuarios en URL retornan 403/404
- [ ] SAST en CI: SonarQube, Semgrep, o CodeQL (al menos uno activo)
- [ ] Dependency scan en CI: `npm audit --audit-level=high`, Snyk, o Dependabot

---

## 6. TESTING DE PERFORMANCE

- [ ] Baseline de performance definido y medido (P50, P95, P99 de latencias clave)
- [ ] Tests de carga para endpoints críticos: k6, Artillery, o Locust
- [ ] El build/pipeline alerta si P95 supera el threshold definido
- [ ] Tests de stress para conocer el punto de quiebre del sistema
- [ ] Tests de memory leak para procesos long-running (workers, cron jobs)
- [ ] DB queries analizadas bajo carga con datos realistas (no solo 10 filas de dev)

---

## 7. CI/CD Y AUTOMATIZACIÓN

- [ ] Todos los tests corren automáticamente en cada PR/push
- [ ] El pipeline falla y bloquea merge si algún test falla
- [ ] Tests unitarios e integración corren en < 5 minutos en CI
- [ ] E2E corren en pipeline separado (no bloquean el ciclo rápido de feedback)
- [ ] Reporte de cobertura generado y visible en cada PR (diff de cobertura)
- [ ] Los test results se archivan para comparación histórica (detectar degradación)
- [ ] Ambientes de test son efímeros y se crean/destruyen por pipeline (no compartidos)

---

## 8. EVALUACIÓN REAL: MUTATION TESTING

Para saber si los tests REALMENTE detectan bugs (no solo cubren líneas):

- [ ] Mutation testing ejecutado: Stryker (JS/TS), PITest (Java), mutmut (Python)
- [ ] **Mutation score < 40%** → tests que no detectan bugs reales — ALARMA 🔴
- [ ] **Mutation score 40-60%** → tests básicos, mejora necesaria 🟡
- [ ] **Mutation score 60-80%** → tests razonables 🟢
- [ ] **Mutation score > 80%** → suite sólido ✅

---

## FORMATO DE REPORTE TESTING STRATEGY

```
## Testing Strategy Report — [Proyecto]
**Stack:** [Jest + Playwright / Pytest + Selenium / Vitest + etc.]
**Cobertura actual:** [X% líneas / Y% branches]
**Tests totales:** N (Unit: N | Integration: N | E2E: N)
**Mutation score:** X% (si disponible)

### Distribución de Pirámide
| Nivel | Actual | Objetivo | Estado |
|-------|--------|----------|--------|
| Unit | X% | 60-70% | OK/WARN/FAIL |
| Integration | X% | 20-30% | OK/WARN/FAIL |
| E2E | X% | 5-10% | OK/WARN/FAIL |

### Hallazgos

🔴 [TEST-001] **[Título]**
- **Área:** Unitarios / Integración / E2E / Seguridad / Performance
- **Descripción:** [qué falta o está mal]
- **Impacto:** [qué bugs puede no detectar]
- **Corrección:** [qué test escribir o cómo arreglar]

### Veredicto
**Confianza en el suite:** ALTA / MEDIA / BAJA / INEXISTENTE
**Riesgo de regresión en producción:** ALTO / MEDIO / BAJO
**Acción prioritaria:** [qué hacer primero]
```
