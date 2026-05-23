# Accessibility Checklist — WCAG 2.1 AA + Resolución 1519/2016 Colombia

> Evalúa accesibilidad funcional de código. Para auditoría visual de diseño UI, usar /web-design-guidelines.

## 1. PERCEIVABLE — Los usuarios pueden percibir el contenido

### 1.1 Alternativas de Texto (WCAG 1.1)
- [ ] Todas las imágenes tienen atributo `alt` descriptivo (no vacío, no "imagen")
- [ ] Imágenes decorativas tienen `alt=""` para que lectores de pantalla las ignoren
- [ ] Iconos sin texto visible tienen `aria-label` o `aria-labelledby`
- [ ] Gráficos complejos (charts, infografías) tienen descripción alternativa en texto
- [ ] Captchas tienen alternativa accesible (audio captcha o método alternativo)

### 1.2 Medios de Tiempo (WCAG 1.2)
- [ ] Videos tienen subtítulos (no auto-generados sin revisión humana)
- [ ] Audio grabado tiene transcripción de texto
- [ ] Videos con información visual tienen audiodescripción

### 1.3 Adaptable — Información no depende solo de presentación visual (WCAG 1.3)
- [ ] Estructura usa HTML semántico: `h1-h6` para jerarquía, no solo tamaño visual
- [ ] Listas usan `ul/ol/li`, no divs con bullets visuales
- [ ] Formularios usan `<label>` asociado a cada input (`for/id` o `aria-labelledby`)
- [ ] Orden de lectura del DOM es lógico sin depender de CSS
- [ ] Errores de formulario no se comunican SOLO por color (agregar icono + texto)
- [ ] Tablas de datos usan `<th>` con `scope` para headers de fila/columna
- [ ] La información no se transmite solo por posición espacial ("el botón de la derecha")

### 1.4 Distinguible — Fácil de ver y escuchar (WCAG 1.4)
- [ ] Contraste texto normal: mínimo **4.5:1** (AA) — verificar con webaim.org/resources/contrastchecker
- [ ] Contraste texto grande (18pt+ o 14pt+ bold): mínimo **3:1**
- [ ] Contraste de componentes UI (bordes de inputs, iconos activos): mínimo **3:1**
- [ ] La información no se transmite SOLO por color
- [ ] El texto puede ampliarse al 200% sin pérdida de contenido ni scroll horizontal
- [ ] Sin texto embebido en imágenes (excepto logos)
- [ ] Espaciado de texto puede modificarse sin romper layout (line-height, letter-spacing, word-spacing)
- [ ] Contenido de audio que reproduce automáticamente puede pausarse

---

## 2. OPERABLE — Los usuarios pueden operar la interfaz

### 2.1 Accesible por Teclado (WCAG 2.1)
- [ ] TODA funcionalidad alcanzable con teclado (Tab, Shift+Tab, Enter, Space, Arrows)
- [ ] Sin "keyboard traps" (modal que no se puede cerrar con Escape)
- [ ] Custom components interactivos tienen keyboard handlers correctos
  - Dropdown: `Arrow Up/Down` navega opciones, `Enter` selecciona, `Escape` cierra
  - Modal: `Escape` cierra, foco queda en el elemento que lo abrió al cerrar
  - Tabs: `Arrow Left/Right` navega tabs
- [ ] Skip links presentes ("Ir al contenido principal") al inicio de la página
- [ ] El orden de Tab es lógico y sigue el flujo visual de la página

### 2.2 Tiempo Suficiente (WCAG 2.2)
- [ ] Las sesiones con timeout informan al usuario y permiten extenderla
- [ ] Los contenidos que se mueven/parpadean pueden pausarse (carruseles, banners animados)
- [ ] Sin contenido que requiere acción en tiempo muy limitado (< 20 segundos)

### 2.3 Sin Convulsiones (WCAG 2.3)
- [ ] Sin contenido que parpadea más de 3 veces por segundo
- [ ] Las animaciones intensas respetan `prefers-reduced-motion` media query
- [ ] Videos con flashes intensos tienen advertencia previa

### 2.4 Navegable (WCAG 2.4)
- [ ] Cada página tiene `<title>` único y descriptivo
- [ ] El foco visible es claramente distinguible (outline visible, no `outline: none` sin reemplazo)
- [ ] Los links tienen texto descriptivo (no "click aquí", no "leer más" sin contexto)
- [ ] La página tiene landmarks: `<header>`, `<nav>`, `<main>`, `<footer>`
- [ ] Múltiples `<nav>` tienen `aria-label` que los diferencia
- [ ] Breadcrumbs presentes en flujos con múltiples pasos

### 2.5 Modalidades de Entrada (WCAG 2.5)
- [ ] Los gestos complejos (swipe, pinch) tienen alternativa con un solo punto de contacto
- [ ] Las acciones activadas por movimiento del dispositivo tienen alternativa por UI
- [ ] Los targets táctiles tienen mínimo 44x44px de área de activación

---

## 3. UNDERSTANDABLE — Los usuarios entienden el contenido

### 3.1 Legibilidad (WCAG 3.1)
- [ ] El idioma de la página declarado (`lang="es"` en `<html>`)
- [ ] Los cambios de idioma dentro de la página están marcados (`lang` en el elemento)

### 3.2 Predecible (WCAG 3.2)
- [ ] Al enfocar un elemento no cambia automáticamente el contexto (no auto-submit on focus)
- [ ] Al cambiar un `<select>` no navega automáticamente sin aviso
- [ ] La navegación es consistente entre páginas del mismo sitio
- [ ] Los componentes idénticos tienen labels idénticos en toda la app

### 3.3 Asistencia en Errores (WCAG 3.3)
- [ ] Los campos con error se identifican específicamente (no solo "hay un error")
- [ ] La descripción del error sugiere cómo corregirlo
- [ ] El mensaje de error está vinculado al input con `aria-describedby`
- [ ] Los formularios con consecuencias importantes (pagos, borrar datos) piden confirmación
- [ ] Los datos de formularios largos pueden revisarse antes de confirmar

---

## 4. ROBUST — Compatible con tecnologías asistivas (WCAG 4.1)

- [ ] HTML válido sin tags no cerrados ni atributos duplicados (validar: validator.w3.org)
- [ ] Los roles ARIA son correctos y no sobreescriben semántica HTML nativa innecesariamente
- [ ] Los componentes custom tienen nombre, rol y valor accesible (`aria-label`, `role`, `aria-valuenow`)
- [ ] Los modales usan `aria-modal="true"`, gestionan foco al abrir y devuelven foco al cerrar
- [ ] Los mensajes de estado (loading, success, error) usan `aria-live` para anunciarse
  - `aria-live="polite"` para notificaciones no urgentes
  - `aria-live="assertive"` solo para errores críticos
- [ ] Los acordeones usan `aria-expanded` correctamente (true/false)
- [ ] Los sliders/inputs range usan `aria-valuenow`, `aria-valuemin`, `aria-valuemax`
- [ ] Los checkboxes y radios indeterminados usan `aria-checked="mixed"`

---

## 5. HERRAMIENTAS DE TESTING

### Testing Automático (detecta ~30-40% de issues de accesibilidad)
- [ ] axe-core integrado en tests E2E (`@axe-core/playwright` o `cypress-axe`)
- [ ] Lighthouse accessibility score ≥ 90 en páginas principales
- [ ] Sin errores en WAVE (wave.webaim.org) en flujos críticos
- [ ] HTML válido según W3C Validator (validator.w3.org)

### Testing Manual (detecta el 60-70% restante — NO omitir)
- [ ] Navegación completa por teclado sin mouse en flujos críticos (registro, login, pago)
- [ ] Zoom 200% en navegador: sin overflow horizontal, sin contenido cortado
- [ ] Testeado con VoiceOver (Mac/iOS) o NVDA (Windows) en flujos principales
- [ ] Testeado con `prefers-reduced-motion: reduce` activo
- [ ] Testeado con modo de alto contraste del sistema operativo
- [ ] Testeado con solo teclado por alguien que no conoce el flujo de la app

---

## FORMATO DE REPORTE ACCESSIBILITY

```
## Accessibility Report — [Aplicación/Página]
**Estándar:** WCAG 2.1 AA
**Herramientas:** [axe-core / Lighthouse / WAVE / manual]
**Páginas revisadas:** [lista]
**Lighthouse Score:** [N/100]

### Hallazgos

🔴 [A11Y-001] **Formulario sin labels asociados**
- **WCAG:** 1.3.1 Info and Relationships (Level A)
- **Elemento:** `<input type="email" placeholder="Email">`
- **Impacto:** Lectores de pantalla no anuncian qué campo es — usuarios ciegos no pueden completar el formulario
- **Corrección:** `<label for="email">Email</label><input id="email" ...>`

### Puntuación por Principio
| Principio | Estado |
|-----------|--------|
| 1. Perceivable | OK/WARN/FAIL |
| 2. Operable | OK/WARN/FAIL |
| 3. Understandable | OK/WARN/FAIL |
| 4. Robust | OK/WARN/FAIL |

**Cumplimiento:** WCAG 2.1 AA CUMPLE / CUMPLE PARCIALMENTE / NO CUMPLE
**Riesgo legal:** ALTO (Colombia Res. 1519/2016 para entidades públicas) / BAJO (privado)
```
