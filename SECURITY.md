# Política de Seguridad — SQA Agent

## Reporte de Vulnerabilidades

Si encuentras una vulnerabilidad en SQA Agent, por favor:

1. **NO** abras un issue público
2. Envía un email a: jhonatan@webzentra.com
3. Incluye: descripción, pasos para reproducir, impacto potencial
4. Responderemos en máximo 48 horas

## Versiones Soportadas

| Versión | Soportada |
|---------|-----------|
| v6.0.x (actual) | ✅ Sí — soporte completo |
| v5.x | ⚠️ Solo parches de seguridad críticos |
| v4.x | ❌ No |
| < v4.0 | ❌ No |

## Divulgación Responsable

1. Recibimos el reporte
2. Confirmamos la vulnerabilidad (máx. 5 días hábiles)
3. Desarrollamos y testeamos el fix
4. Publicamos la nueva versión con el fix
5. 30 días después, publicamos los detalles técnicos

## Alcance

SQA Agent es un agente de texto (prompts + checklists Markdown). Las vulnerabilidades de seguridad relevantes incluyen:

- Instrucciones en los checklists que induzcan al agente a ejecutar acciones destructivas
- Prompt injection embebido en los checklists o SKILL.md
- Credenciales o datos sensibles hardcodeados en cualquier archivo del repositorio
- Referencias a herramientas o comandos que puedan causar daño en sistemas de usuarios

No están en scope: vulnerabilidades del modelo de lenguaje subyacente (reportar a Anthropic), vulnerabilidades del CLI de Claude Code (reportar a Anthropic).
