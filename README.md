# Lab 3 — dig | DNS Footprinting Pasivo + WHOIS

## Herramienta: dig (Domain Information Groper) + whois

`dig` es la herramienta avanzada de consulta DNS. A diferencia de `nslookup`, muestra TTL, la ruta completa de delegación jerárquica, DNSSEC y registros TXT. Se complementa con `whois` para obtener datos del registro del dominio.

---

## Sitio Analizado

**latinoamericacomparte.com** — plataforma orientada a programas sociales, emprendimiento y transformación empresarial en Latinoamérica.

---

## Objetivo

Realizar un análisis DNS avanzado sobre el dominio para identificar:

- IP pública y TTL
- Jerarquía completa de resolución DNS (Root → TLD → dominio)
- Estado de DNSSEC (firmado o sin firmar)
- Información del registrador y estado del dominio (whois)

---

## Comandos Ejecutados y Análisis

### Paso 1 — Consulta básica (Registro A)

```bash
dig latinoamericacomparte.com
```

**Salida:**
```
;; ANSWER SECTION:
latinoamericacomparte.com. 159  IN      A       64.202.187.143

;; Query time: 7 msec
;; SERVER: 192.168.1.1#53(192.168.1.1) (UDP)
;; WHEN: Wed May 20 16:58:26 EDT 2026
```

**Análisis:**
| Campo | Valor | Significado |
|---|---|---|
| A record | 64.202.187.143 | IP pública del servidor |
| TTL | 159 | Segundos restantes en caché al momento de la consulta |
| Query time | 7 ms | Tiempo de respuesta (respuesta desde caché local) |
| status | NOERROR | Consulta exitosa, dominio existe |
| flags: ad | presente | DNSSEC validado por el recursor (solo en .com, no en el dominio) |

---

### Paso 2 — Trazado jerárquico completo

```bash
dig -4 +trace latinoamericacomparte.com
```

**Ruta de resolución:**
```
. (Root) → com. (TLD) → latinoamericacomparte.com. (dominio)
```

**Delegaciones identificadas:**

| Nivel | Servidor | Función |
|---|---|---|
| Root (.) | a-m.root-servers.net | Nivel más alto del DNS global |
| TLD (.com) | a-m.gtld-servers.net | Gestiona todos los dominios .com |
| Dominio | ns05.domaincontrol.com | Servidor autoritativo de GoDaddy |
| Dominio | ns06.domaincontrol.com | Servidor autoritativo secundario |

**Respuesta autoritativa final:**
```
latinoamericacomparte.com. 3600 IN  A  64.202.187.143
```

> El trazado confirma que GoDaddy (`ns05/ns06.domaincontrol.com`) es la autoridad final. TTL de 3600s en la zona autoritativa.

---

### Paso 3 — Verificación DNSSEC (DNSKEY)

```bash
dig DNSKEY latinoamericacomparte.com
```

**Salida:**
```
;; ANSWER: 0
;; AUTHORITY SECTION:
latinoamericacomparte.com. 300  IN  SOA  ns05.domaincontrol.com. dns.jomax.net.
```

**Análisis:** No se encontraron claves DNSKEY. El dominio **no tiene DNSSEC implementado**.

---

### Paso 4 — Verificación DS (Delegation Signer)

```bash
dig DS latinoamericacomparte.com
```

**Salida:**
```
;; ANSWER: 0
;; AUTHORITY SECTION:
com.  145  IN  SOA  a.gtld-servers.net. nstld.verisign-grs.com.
```

**Análisis:** No existe registro DS en la zona `.com`. Confirma que el dominio **no está firmado con DNSSEC**.

| Resultado | Significado | Riesgo |
|---|---|---|
| Sin DS | No hay enlace de confianza TLD → dominio | Alto: vulnerable a Cache Poisoning |
| Sin DNSKEY | No hay claves criptográficas publicadas | Alto: respuestas DNS no verificables |
| Sin RRSIG | Registros no firmados | Sin integridad de respuestas |

---

### Paso 5 — WHOIS (Información del Registrador)

```bash
whois latinoamericacomparte.com
```

**Hallazgos:**

| Campo | Valor |
|---|---|
| Registrador | GoDaddy.com, LLC |
| Fecha de creación | 2025-08-06 |
| Fecha de expiración | 2026-08-06 |
| DNSSEC | unsigned |
| Registrante | Registration Private / Domains By Proxy, LLC |
| Estado | clientDeleteProhibited, clientRenewProhibited, clientTransferProhibited, clientUpdateProhibited |
| SOA serial | 2025110701 |

**Análisis del Registro:**

- **Dominio joven:** creado en agosto 2025, vigente solo 1 año → riesgo de no renovación.
- **Identidad oculta:** el registrante usa un servicio de privacidad (Domains By Proxy), lo que impide identificar al propietario real.
- **Estados client*Prohibited:** protegen el dominio contra transferencias no autorizadas (buena práctica de seguridad registral).
- **DNSSEC unsigned:** la ausencia de firma digital expone el dominio a ataques de envenenamiento de caché DNS.

---

## Resumen de Hallazgos de Seguridad

| Elemento | Estado | Impacto |
|---|---|---|
| IP pública | 64.202.187.143 | Punto de entrada identificado |
| Proveedor DNS | GoDaddy (domaincontrol.com) | Dependencia de infraestructura de tercero |
| DNSSEC | **No implementado** | Vulnerable a Cache Poisoning |
| Identidad registrante | Oculta (proxy) | No se puede identificar propietario |
| Antigüedad del dominio | < 1 año | Riesgo de no renovación |
| TTL autoritativo | 3600s | Caché de 1 hora — cambios de IP lentos en propagarse |

---

## Interpretación de Términos dig

| Término | Significado |
|---|---|
| `status: NOERROR` | Consulta exitosa |
| `ANSWER SECTION` | Contiene el resultado solicitado |
| `AUTHORITY SECTION` | Servidor responsable de la zona |
| `flags: qr rd ra` | Query Response, Recursion Desired, Recursion Available |
| `+trace` | Muestra cada paso de la cadena de delegación DNS |
| `DNSKEY` | Clave pública DNSSEC del dominio |
| `DS` | Delegation Signer — enlace de confianza TLD → dominio |

---

## Herramientas del Laboratorio

| Herramienta | Propósito |
|---|---|
| `dig` | Análisis DNS avanzado: TTL, DNSSEC, trazado jerárquico |
| `whois` | Información del registrador y estado del dominio |
| `nslookup` | Consultas DNS básicas (ver rama `lab-nslookup`) |
