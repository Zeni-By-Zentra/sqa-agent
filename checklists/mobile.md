# Mobile Quality Checklist — OWASP Mobile Top 10:2024 + iOS HIG + Material Design 3

## 1. SEGURIDAD MÓVIL (OWASP Mobile Top 10:2024)

### M1 — Credenciales Débiles / Improper Credential Usage
- [ ] Sin credenciales hardcodeadas en el APK/IPA (strings, buildConfigs, plist, Info.plist)
- [ ] API keys no incluidas en el bundle — usar backend proxy o secrets manager
- [ ] Certificados de desarrollo no incluidos en build de producción
- [ ] Obfuscación activa en Android (ProGuard/R8), symbols ocultados en iOS (`STRIP_SWIFT_SYMBOLS`)
- [ ] Los logs de debug no imprimen tokens, passwords ni PII en producción

### M2 — Seguridad Insuficiente de Supply Chain
- [ ] Dependencias de terceros auditadas (licencias + vulnerabilidades conocidas)
- [ ] SDKs de analytics/ads revisados (qué datos recopilan exactamente)
- [ ] Firmas de APK/IPA verificadas en el pipeline de CI
- [ ] Sin SDKs de fuentes no verificadas (repositorios privados sin auditoría)

### M3 — Autenticación/Autorización Insegura
- [ ] Tokens almacenados en **Keychain** (iOS) o **Keystore** (Android) — NUNCA en SharedPreferences sin cifrar, AsyncStorage sin cifrar, o archivos en Documents
- [ ] Biométrica como 2FA adicional, no como único factor para operaciones críticas
- [ ] Las sesiones se invalidan en el servidor al hacer logout (no solo limpiar storage local)
- [ ] Deeplinks y Universal Links validan autenticación antes de ejecutar acciones
- [ ] Re-autenticación requerida para acciones sensibles (cambio de email, datos bancarios, borrar cuenta)

### M4 — Validación Insuficiente de Input/Output
- [ ] La app valida inputs en cliente Y en servidor (nunca solo cliente-side)
- [ ] Las respuestas del servidor se validan antes de renderizar (sin `eval()` de JSON no confiable)
- [ ] Los deeplinks validan origen y parámetros antes de ejecutar
- [ ] Formularios tienen límites de longitud y tipo de caracteres

### M5 — Comunicación Insegura
- [ ] TLS 1.2+ en todas las llamadas de red sin excepción
- [ ] Certificate pinning implementado para endpoints críticos (pagos, auth)
- [ ] Sin comunicación HTTP en producción (ATS activo en iOS, `cleartextTrafficPermitted=false` en Android)
- [ ] Los logs de red en modo debug no se activan en builds de producción
- [ ] HPKP o certificate pinning con backup pins para evitar lockout

### M6 — Controles de Privacidad Inadecuados
- [ ] Permisos solicitados just-in-time con justificación clara justo antes de necesitarlos
- [ ] Sin permisos innecesarios (no pedir contactos si no es esencial para el feature)
- [ ] Los datos recopilados declarados en Privacy Policy, App Store y Google Play
- [ ] Cumplimiento con App Tracking Transparency (iOS 14+) si hay tracking cross-app
- [ ] Política de privacidad accesible desde la app sin requerir login
- [ ] Cumplimiento con Ley 1581/2012 (Colombia) para datos de colombianos

### M7 — Protecciones Binarias Insuficientes
- [ ] Anti-tampering activo en apps financieras o que manejan datos muy sensibles
- [ ] Detección de jailbreak/root implementada donde el riesgo lo justifica
- [ ] Debug mode deshabilitado en builds de producción (`BuildConfig.DEBUG = false`)
- [ ] Sin logs de debug en producción (`console.log`, `Log.d`, `print()` eliminados o guard por flag)
- [ ] Compilación con optimizaciones habilitadas (minificación, tree shaking)

### M8 — Seguridad de Datos en Reposo
- [ ] Base de datos local cifrada si contiene datos sensibles (SQLCipher, Realm con cifrado)
- [ ] Archivos sensibles en directorio privado de la app, no en Documents/Downloads (accesible por otras apps)
- [ ] Sin PII en logs del dispositivo accesibles por otras apps
- [ ] `android:allowBackup="false"` para datos sensibles en Android
- [ ] `NSFileProtectionComplete` para archivos sensibles en iOS (protección cuando el device está bloqueado)

### M9 — Gestión Insegura de Credenciales
- [ ] Las sesiones tienen tiempo de expiración (no tokens infinitos)
- [ ] Refresh tokens tienen rotación (un refresh token se invalida al usarse)
- [ ] Biométrica es siempre opcional (usuarios pueden usar PIN/password como alternativa)
- [ ] Al detectar sesión comprometida, logout forzado con notificación

### M10 — Funcionalidad Insuficiente
- [ ] La app funciona correctamente en modo avión para funciones declaradas como offline
- [ ] Los errores de red tienen mensajes comprensibles (no "Error 500" o "Network Error")
- [ ] Las versiones muy antiguas pueden ser forzadas a actualizar con gracia (force update)
- [ ] Los crashes se reportan automáticamente (Firebase Crashlytics, Sentry)

---

## 2. PERFORMANCE MÓVIL (ISO/IEC 25010 §4.3)

### 2.1 Startup
- [ ] Cold start < 3 segundos en dispositivo mid-range del año target
- [ ] Warm start < 1 segundo
- [ ] Sin operaciones síncronas bloqueantes en el hilo principal (UI thread / main thread)
- [ ] Splash screen no oculta problemas de startup lento

### 2.2 Rendering y Listas
- [ ] Las listas largas usan virtualización (`FlatList`/`RecyclerView`/`LazyColumn`) — nunca `ScrollView` con items dinámicos
- [ ] Las animaciones corren a 60fps en dispositivos target (sin dropped frames medibles)
- [ ] Las imágenes se cargan progresivamente y se cachean localmente (no se re-descargan en cada scroll)
- [ ] Sin re-renders innecesarios en componentes de lista (usar `memo`, `useCallback`, `key` correctos)

### 2.3 Consumo de Recursos
- [ ] Sin memory leaks detectados (Android Profiler / Xcode Instruments / Flipper)
- [ ] Uso de batería no excesivo (no hay wakelock innecesario, background fetch optimizado)
- [ ] Tamaño del bundle razonable: Android APK < 50MB, iOS < 100MB (splits/slices recomendados para apps grandes)
- [ ] Los assets (imágenes, fuentes) están optimizados (WebP para imágenes, subsets de fuentes)

### 2.4 Red
- [ ] Requests de red tienen timeout configurado (no esperan infinitamente)
- [ ] Retry automático con backoff exponencial para errores transitorios de red
- [ ] Caché de respuestas para datos que no cambian frecuentemente
- [ ] Paginación implementada para listas con muchos items (infinite scroll o páginas)
- [ ] Sin polling innecesario (usar WebSockets o push notifications en su lugar)

---

## 3. EXPERIENCIA DE USUARIO MÓVIL

### 3.1 Touch y Gestos
- [ ] Targets táctiles mínimo **44x44pt** (iOS HIG) / **48x48dp** (Material Design 3)
- [ ] Espaciado suficiente entre elementos interactivos (mínimo 8dp/pt entre targets)
- [ ] Feedback visual/táctil inmediato (< 100ms) a la interacción del usuario
- [ ] Los gestos nativos de la plataforma no están interceptados (swipe-back en iOS, back gesture en Android)
- [ ] Haptic feedback usada apropiadamente (no en exceso, coherente con la acción)

### 3.2 Orientación y Tamaños de Pantalla
- [ ] La app funciona en portrait y landscape (o deshabilita landscape con justificación clara)
- [ ] Testeado en pantallas pequeñas (iPhone SE / Android compact) y grandes (iPhone Pro Max / tablet)
- [ ] La UI se adapta a Dynamic Type / Font Scale del sistema operativo
- [ ] Soporte correcto para dark mode y light mode
- [ ] Funciona correctamente con notch/Dynamic Island / punch-hole en dispositivos modernos
- [ ] Safe areas respetadas (no contenido bajo el home indicator o detrás de la status bar)

### 3.3 Estado y Recuperación
- [ ] La app maneja interrupciones correctamente (llamada entrante, notificación, cambio de app)
- [ ] El estado se guarda al pasar a background y se restaura al volver
- [ ] Los formularios no pierden datos al rotar o al interrumpirse
- [ ] Los deep links restauran el estado correcto de la app
- [ ] El back button (Android) o swipe-back (iOS) tiene comportamiento coherente y esperado

### 3.4 Onboarding y Permisos
- [ ] Los permisos se piden en contexto (no todos al iniciar la app por primera vez)
- [ ] Si el usuario rechaza un permiso, la app sigue funcionando (degraded gracefully)
- [ ] El onboarding es skippable y los pasos son mínimos
- [ ] La app no requiere crear cuenta para explorar (si aplica al tipo de app)

---

## 4. CALIDAD PARA APP STORES

- [ ] Metadata completa y actualizada: descripción, keywords, screenshots, preview video
- [ ] Screenshots reflejan la UI actual (no versiones antiguas)
- [ ] La app cumple las políticas de privacidad del store (App Store Review Guidelines / Google Play Policy)
- [ ] Sin APIs privadas de iOS (`_` prefix, clases no documentadas)
- [ ] Permisos declarados en `Info.plist`/`AndroidManifest.xml` con descripción real del uso
- [ ] El flujo de compra in-app funciona correctamente en sandbox antes del release
- [ ] App Rating prompt implementado correctamente (usando `SKStoreReviewController` en iOS, no implementación propia)
- [ ] Los crashes de release candidate < 1% (testeado con TestFlight / Firebase App Distribution)

---

## FORMATO DE REPORTE MOBILE REVIEW

```
## Mobile Quality Report — [App Name]
**Plataforma:** iOS / Android / React Native / Flutter / [versión]
**Dispositivos testeados:** [lista]
**Norma:** OWASP Mobile Top 10:2024, iOS HIG, Material Design 3

### Seguridad (OWASP)
🔴/🟡/🟢 [M-ID] [hallazgo + referencia OWASP]

### Performance
🔴/🟡/🟢 [hallazgo + métrica medida]

### UX Móvil
🔴/🟡/🟢 [hallazgo + plataforma afectada]

### Puntuación
| Categoría | Estado |
|-----------|--------|
| Seguridad OWASP M1-M10 | OK/WARN/FAIL |
| Performance | OK/WARN/FAIL |
| UX y plataforma | OK/WARN/FAIL |
| App Store readiness | OK/WARN/FAIL |
| Privacidad / Cumplimiento | OK/WARN/FAIL |

**Veredicto:** LISTO PARA STORE / REQUIERE CORRECCIONES ANTES DEL RELEASE / BLOQUEADO
```
