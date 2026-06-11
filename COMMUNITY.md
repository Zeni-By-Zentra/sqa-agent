# Comunidad SQA Agent — Guía de Contribución

Una comunidad abierta de desarrolladores colombianos y globales que creen que el software de calidad no es lujo, es responsabilidad.

## Código de Conducta

- Respeto mutuo ante todo — no hay preguntas "estúpidas"
- Críticas al código, nunca a la persona
- El español y el inglés son igualmente bienvenidos
- Ayudamos a quienes están aprendiendo — este proyecto nació en ese espíritu

## Formas de contribuir

### 1. Usar el agente y reportar feedback
La contribución más valiosa es usar SQA Agent en proyectos reales y contarnos:
- ¿Qué encontró que no debería haber encontrado? (falso positivo)
- ¿Qué no encontró que debería haber encontrado? (falso negativo)
- ¿Qué norma falta? ¿Qué checklist necesita el proyecto?

Abre un [issue](https://github.com/Zeni-By-Zentra/sqa-agent/issues/new) con la etiqueta `feedback`.

### 2. Corregir o mejorar un checklist existente

1. Fork del repo
2. Edita el archivo en `checklists/`
3. Sigue el formato de tabla: `| ID | Verificación | Norma | Severidad |`
4. Cada ítem debe citar una norma específica con sección (ej: `OWASP A03:2021`, `ISO 27001 A.9.4.2`)
5. Abre un PR con descripción: qué cambiaste y por qué

### 3. Agregar un checklist nuevo

1. Crea `checklists/tu-checklist.md` con esta estructura mínima:
   ```markdown
   # [Nombre] Assessment — [Norma principal]
   
   ## Activación
   Este checklist se activa cuando el usuario menciona:
   - "palabra clave 1"
   - "palabra clave 2"
   
   ## Normas Aplicadas
   - [Norma 1:Año — Descripción]
   
   ## [Categoría 1]
   | ID | Verificación | Norma | Severidad |
   |----|-------------|-------|-----------|
   | XX-001 | ... | ... | 🔴/🟡/🟢 |
   ```
2. Agrega la URL de fetch en `SKILL.md` en la tabla de checklists
3. Agrega la señal de detección en la tabla de tipos de análisis de `SKILL.md`
4. Actualiza `README.md` con el nuevo checklist
5. Agrega entrada en `CHANGELOG.md`

### 4. Reportar un falso positivo

Un falso positivo ocurre cuando el agente reporta un hallazgo que NO es un problema real en tu contexto.

Abre un issue con:
- Etiqueta: `false-positive`
- Qué hallazgo generó (ID + título)
- Por qué no aplica en tu caso
- El contexto de tu proyecto (tipo, stack, normas aplicables)

Usamos estos reportes para refinar los checklists y agregar condiciones de contexto.

### 5. Traducir documentación

Si quieres traducir checklists al inglés u otro idioma, abre un issue primero para coordinarlo.

## Entorno de desarrollo

```bash
# Clonar el repo
git clone https://github.com/Zeni-By-Zentra/sqa-agent.git
cd sqa-agent

# Instalar como skill de Claude Code
mkdir -p ~/.claude/skills
ln -s $(pwd) ~/.claude/skills/sqa-agent

# Probar el skill
# En Claude Code: /sqa-agent --quick src/tuArchivo.ts
```

## Template de Pull Request

```markdown
## Descripción
[Qué cambia y por qué]

## Tipo de cambio
- [ ] Nuevo checklist
- [ ] Corrección de checklist existente
- [ ] Nueva norma/regulación
- [ ] Bug fix en SKILL.md
- [ ] Documentación

## Normas referenciadas
- [Norma 1 con sección]
- [Norma 2 con sección]

## Checklist
- [ ] Seguí el formato de tabla estándar
- [ ] Cada ítem cita una norma específica
- [ ] Actualicé CHANGELOG.md
- [ ] Actualicé README.md si agregué checklist nuevo
```

## Badge de cumplimiento

Muestra que tu proyecto fue auditado con SQA Agent:

```markdown
[![SQA Agent Auditado](https://img.shields.io/badge/SQA%20Agent-v6.0%20Auditado-brightgreen)](https://github.com/Zeni-By-Zentra/sqa-agent)
```

Para obtener badge de nivel (Oro/Plata/Bronce), usa `sqa-agent --community` en tu proyecto.

## Reconocimientos

- **SENA Colombia** — Por el programa "Procesos para Software de Calidad" que inspiró este proyecto
- **OWASP Foundation** — Top 10 y ASVS como referencia base de seguridad
- **ISO/IEC** — Estándares internacionales de calidad (25010, 27001, 25012)
- **Comunidad dev colombiana** — Por el uso real en proyectos y el feedback que mejora cada versión
- **Contribuidores** — Ver [Contributors](https://github.com/Zeni-By-Zentra/sqa-agent/graphs/contributors)

## Contacto

- **Issues y PRs:** [github.com/Zeni-By-Zentra/sqa-agent](https://github.com/Zeni-By-Zentra/sqa-agent)
- **Email mantenedor:** jhonatan@webzentra.com
- **Web:** [webzentra.com](https://webzentra.com)
