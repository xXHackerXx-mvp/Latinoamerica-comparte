# Lab 4 — Maltego | Reconocimiento Visual e Inteligencia OSINT

## Herramienta: Maltego

Maltego es una plataforma de inteligencia de código abierto (OSINT) con interfaz gráfica que permite visualizar relaciones entre entidades como dominios, IPs, personas, correos electrónicos, organizaciones y servidores. Utiliza **transforms** (consultas automáticas) para recolectar y enlazar información de múltiples fuentes públicas simultáneamente, generando grafos de relaciones que revelan la infraestructura de un objetivo.

---

## Sitio Analizado

**latinoamericacomparte.com** — plataforma orientada a programas sociales, emprendimiento y transformación empresarial en Latinoamérica.

---

## Objetivo

Mapear visualmente la infraestructura pública del dominio e identificar:

- Relaciones entre dominio, IPs y servidores de nombres
- Subdominios expuestos
- Correos electrónicos asociados
- Tecnologías del servidor
- Certificados SSL
- Información pública sensible vinculada al dominio

---

## Conceptos Clave

| Término | Definición |
|---|---|
| Entity (Entidad) | Objeto en el grafo: dominio, IP, correo, persona, organización |
| Transform | Consulta automática que extrae información de una entidad |
| Graph | Visualización de relaciones entre entidades |
| OSINT | Open Source Intelligence — información obtenida de fuentes públicas |
| Footprinting | Fase inicial de recolección de información sobre un objetivo |

---

## Flujo del Laboratorio

### Paso 1 — Abrir Maltego y crear un nuevo grafo

1. Abrir Maltego Community Edition
2. Crear una nueva investigación: `File > New`
3. En la paleta de entidades buscar **Domain**
4. Arrastrar al grafo y configurar: `latinoamericacomparte.com`

---

### Paso 2 — Ejecutar Transforms sobre el dominio

Clic derecho sobre la entidad del dominio → **Run Transforms**

**Transforms relevantes ejecutados:**

| Transform | Qué descubre |
|---|---|
| To DNS Name (All) | Subdominios asociados al dominio |
| To IP Address | IP pública del dominio (64.202.187.143) |
| To Nameserver | Servidores DNS: ns05/ns06.domaincontrol.com |
| To MX Record | Servidores de correo del dominio |
| To Website | Sitio web relacionado |
| To Whois Record | Información del registrador (GoDaddy) |

---

### Paso 3 — Análisis de la IP encontrada

Al obtener la IP `64.202.187.143`, se ejecutan transforms sobre ella:

| Transform | Resultado |
|---|---|
| To Autonomous System | Sistema autónomo al que pertenece la IP |
| To Netblock | Rango de IPs del mismo bloque de red |
| To Organization | Organización dueña del bloque (GoDaddy/hosting) |
| To Location | Geolocalización aproximada del servidor |

---

### Paso 4 — Análisis de Nameservers

Sobre los nameservers `ns05.domaincontrol.com` y `ns06.domaincontrol.com`:

| Transform | Resultado |
|---|---|
| To IP Address | IPs de los servidores DNS |
| To Domain | Otros dominios que usan los mismos NS |

> Identificar otros dominios que comparten la misma infraestructura DNS permite mapear la superficie de ataque compartida.

---

### Paso 5 — Búsqueda de correos y personas

Sobre el dominio, ejecutar:

| Transform | Resultado esperado |
|---|---|
| To Email Address | Correos asociados al dominio |
| To Person | Personas vinculadas públicamente |
| To Phone Number | Teléfonos encontrados en registros públicos |

> En este caso, el registrante usa privacidad de dominio (Domains By Proxy), por lo que estos datos pueden no estar disponibles directamente.

---

## Hallazgos de Seguridad

| Elemento | Valor / Estado | Impacto |
|---|---|---|
| IP pública | 64.202.187.143 | Punto de entrada identificado |
| Proveedor DNS | GoDaddy (domaincontrol.com) | Dependencia de tercero |
| Subdominios | A verificar con transforms | Superficie de ataque adicional |
| Correos expuestos | Ocultos por privacidad de registro | Bajo riesgo de phishing directo |
| Tecnología del servidor | A identificar via transforms | Permite detectar versiones vulnerables |
| Certificado SSL | A verificar | Revela fechas y autoridad emisora |

---

## Ventajas de Maltego vs Herramientas CLI

| Criterio | Maltego | nslookup / dig |
|---|---|---|
| Visualización | Grafo interactivo | Texto plano |
| Fuentes de datos | Múltiples simultáneas | Una por consulta |
| Relaciones | Automáticas y visuales | Manuales |
| Velocidad de análisis | Alta para mapeo amplio | Alta para consultas específicas |
| Uso ideal | Reconocimiento amplio OSINT | Análisis técnico DNS preciso |

---

## Contexto: Red Team / Blue Team

| Rol | Uso |
|---|---|
| Red Team | Mapear rápidamente toda la superficie pública antes del escaneo activo |
| Blue Team | Auditar qué información de la organización es visible públicamente en OSINT |

---

## Herramientas del Laboratorio

| Herramienta | Propósito |
|---|---|
| Maltego CE | Reconocimiento OSINT visual con grafos de relaciones |
| nslookup | Consultas DNS básicas (ver rama `lab-nslookup`) |
| dig | Análisis DNS avanzado (ver rama `lab-dig`) |
