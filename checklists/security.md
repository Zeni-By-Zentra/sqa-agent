# Security Audit Checklist — OWASP + ISO/IEC 27001:2022

## Normativa Colombiana de Seguridad

| Norma | Descripción | Requisito |
|-------|-------------|-----------|
| Ley 1581/2012 | Habeas Data / Protección de datos personales | Cifrado, control de acceso, auditoría |
| Circular 007 SIC/2018 | Instrucciones para operadores de datos | Medidas de seguridad técnicas y administrativas |
| Decreto 1377/2013 | Reglamentación Ley 1581 | Política de tratamiento de datos |
| Ley 1273/2009 | Delitos informáticos en Colombia | Penaliza acceso no autorizado, daño informático |
| Resolución 1519/2016 | Estándares para entidades públicas | Seguridad en publicación de información |

## 1. GESTIÓN DE AUTENTICACIÓN

- [ ] MFA para cuentas privilegiadas (ISO 27001 A.9.4.2)
- [ ] Passwords mínimo 12 caracteres (NIST SP 800-63B)
- [ ] Bloqueo tras 5-10 intentos fallidos (OWASP ASVS 2.2.1)
- [ ] Backoff exponencial o CAPTCHA en bloqueos
- [ ] Notificación de logins desde nuevos dispositivos
- [ ] Tokens con CSPRNG, longitud mínima 128 bits
- [ ] Invalidación de sesión en logout (servidor, no solo cliente)
- [ ] Timeout de sesión inactiva (15-30 min apps sensibles)
- [ ] Regeneración de session ID tras login exitoso

## 2. DATOS SENSIBLES (ISO 27001 A.8.2 + Ley 1581 Colombia)

- [ ] PII identificada y documentada
- [ ] Datos sensibles cifrados en reposo (AES-256 mínimo)
- [ ] Backups cifrados
- [ ] Política de retención y eliminación de datos
- [ ] TLS 1.2+ en todos los endpoints
- [ ] HSTS habilitado con includeSubDomains
- [ ] Sin datos sensibles en URLs (aparecen en logs)
- [ ] Passwords con bcrypt (cost≥12), argon2id, o scrypt
- [ ] API keys en variables de entorno o secrets manager
- [ ] CERO secrets en código fuente o commits
- [ ] Rotación periódica de credenciales (90 días para privilegiados)

## 3. SEGURIDAD DE API (OWASP API Security Top 10:2023)

- [ ] Cada request verifica acceso al objeto específico (API1)
- [ ] Tokens con expiración corta (access: 15min-1hr) (API2)
- [ ] Refresh tokens con rotación y revocación (API2)
- [ ] Endpoints no retornan más campos de los necesarios (API3)
- [ ] Rate limiting por IP y por usuario autenticado (API4)
- [ ] Límites en tamaño de payload (API4)
- [ ] CORS configurado para orígenes específicos, no wildcard (API8)
- [ ] Headers de seguridad presentes (API8)
- [ ] Swagger/docs deshabilitados o protegidos en producción (API8)

## 4. INFRAESTRUCTURA (ISO 27001 A.12)

- [ ] OS actualizado sin CVEs críticos
- [ ] SSH solo con llave pública, root login deshabilitado
- [ ] Firewall con mínimo privilegio
- [ ] Fail2ban activo
- [ ] Dependencias auditadas (npm audit, trivy)
- [ ] Sin dependencias con CVSS >= 7.0
- [ ] Logs de autenticación con timestamp e IP
- [ ] Logs de cambios en datos críticos
- [ ] Alertas para anomalías
- [ ] Retención de logs mínimo 90 días
- [ ] Logs sin passwords, tokens ni PII sin enmascarar

## 5. GESTIÓN DE INCIDENTES (ISO 27001 A.16)

- [ ] Procedimiento documentado de respuesta a incidentes
- [ ] Canal de reporte de vulnerabilidades (security@dominio)
- [ ] Contactos de emergencia actualizados
- [ ] Plan de recuperación testeado periódicamente

## FORMATO DE REPORTE

## Security Audit Report — [Aplicación]
**Estándares:** OWASP Top 10:2021, OWASP API Security:2023, ISO/IEC 27001:2022
**Alcance:** [endpoints/módulos revisados]

### Hallazgos Críticos 🔴
[Riesgos que requieren acción inmediata]

### Hallazgos Importantes 🟡
[Riesgos para resolver en el próximo sprint]

### Sugerencias 🟢

### Puntuación de Seguridad
| Categoría | Estado |
|-----------|--------|
| Autenticación | OK/WARN/FAIL |
| Autorización | OK/WARN/FAIL |
| Datos en tránsito | OK/WARN/FAIL |
| Datos en reposo | OK/WARN/FAIL |
| Gestión de secretos | OK/WARN/FAIL |
| Logging/Monitoreo | OK/WARN/FAIL |
| Dependencias | OK/WARN/FAIL |

**Nivel de riesgo global:** CRÍTICO / ALTO / MEDIO / BAJO
