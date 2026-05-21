# Laboratorio de Ciberseguridad — latinoamericacomparte.com

## Sitio Analizado

**[Latinoamérica Comparte](https://latinoamericacomparte.com/)** — plataforma orientada a programas sociales, emprendimiento, productividad y transformación empresarial en Latinoamérica.

---

## Objetivo General

Realizar un entorno controlado de reconocimiento, análisis de infraestructura y pruebas de seguridad web sobre el dominio `latinoamericacomparte.com`, aplicando herramientas especializadas de ciberseguridad en cada fase del proceso, bajo los principios del pentesting ético y la metodología **PIAP-LLM (F-PIA Framework)**.

---

## Infraestructura Identificada

| Elemento | Valor |
|---|---|
| Dominio | latinoamericacomparte.com |
| IP pública | 64.202.187.143 |
| Registrador | GoDaddy.com, LLC |
| Servidores DNS | ns05.domaincontrol.com / ns06.domaincontrol.com |
| DNSSEC | No implementado (unsigned) |
| Creación del dominio | 2025-08-06 |
| Expiración | 2026-08-06 |
| Registrante | Oculto (Domains By Proxy, LLC) |

---

## Ramas y Herramientas

| Rama | Lab | Herramienta | Fase | Descripción |
|---|---|---|---|---|
| [`lab-dig`](../../tree/lab-dig) | Lab 3 | nslookup + dig + whois | Reconocimiento Pasivo | Resolución DNS, análisis DNSSEC, SOA, TTL y registro del dominio |
| [`lab-httrack`](../../tree/lab-httrack) | Lab 2 | HTTrack Website Copier | Reconocimiento Pasivo | Clonación del sitio y análisis de exposición frontend con grep |
| [`lab-maltego`](../../tree/lab-maltego) | Lab 4 | Maltego CE | Reconocimiento OSINT | Grafos de relaciones: IP, NS, subdominios, correos, Whois |
| [`lab-burpsuite`](../../tree/lab-burpsuite) | Lab 5 | Burp Suite Community | Análisis Web | Interceptación HTTP/HTTPS, análisis de autenticación y cookies |
| [`lab-injection`](../../tree/lab-injection) | Lab 7 | OWASP ZAP 2.17.0 | Análisis de Vulnerabilidades | Spider, AJAX Spider, Active Scan, HTTP Sessions, Alerts |

---

## Metodología Aplicada

```
Reconocimiento Pasivo ──────────────────────── Clonación del sitio
        │                                               │
  nslookup / dig / whois                           HTTrack
  IP · NS · PTR · DNSSEC · SOA               Exposición frontend
        │                                       grep: api|token|key
        ▼
   Maltego CE (OSINT visual)
   Transforms: To IP · To NS · To MX
   To Whois · To Email Address
        │
        ▼
Análisis de Tráfico HTTP ──────────── Detección de Vulnerabilidades
        │                                           │
    Burp Suite                                  OWASP ZAP
  Proxy 127.0.0.1:8080                   Spider + Active Scan
  Intercept · Repeater                   Alerts + HTTP Sessions
  POST auth · Cookies                    CSP · Cookies · Banner
        │                                           │
        └──────────────── Pruebas de Inyección ─────┘
                          SQL Injection (PortSwigger)
                          XSS Almacenado (Juice Shop)
```

---

## Marco Metodológico: F-PIA (PIAP-LLM)

Este laboratorio sigue el **Framework PIAP-LLM** — Prompt Injection Analysis and Prevention for Large Language Models, que define cuatro fases:

| Fase | Descripción |
|---|---|
| **Prevención** | Diseño seguro, separación de contexto, validadores |
| **Identificación** | Detección de entradas maliciosas, clasificación de ataques |
| **Análisis** | Evaluación de comportamiento, medición de impacto |
| **Mitigación** | Filtros, refuerzo de instrucciones, monitoreo continuo |

---

## Qué se Analizó

- Subdominios expuestos y registros DNS públicos
- IPs públicas, proveedor de hosting y configuración DNSSEC
- Tecnologías del servidor y certificados SSL
- Headers HTTP de seguridad (CSP, HSTS, X-Frame-Options)
- Cookies y tokens de sesión (HttpOnly, SameSite, Secure)
- Formularios y endpoints de autenticación
- Rutas y estructura interna del sitio (frontend clonado)
- Grafos de relaciones OSINT (IP, NS, correos, Whois)
- Vulnerabilidades de inyección (SQLi, XSS) en entornos controlados

---

## Riesgos Identificados

| Riesgo | Nivel | Herramienta | Estado |
|---|---|---|---|
| Content-Security-Policy ausente | Alto | OWASP ZAP | Confirmado |
| Cookies sin HttpOnly / SameSite / Secure | Alto | OWASP ZAP · Burp Suite | Confirmado |
| SQL Injection — login bypass | Crítico | Burp Suite (PortSwigger lab) | Confirmado |
| XSS Almacenado — robo de cookies | Alto | Burp Suite (Juice Shop) | Confirmado |
| DNSSEC no implementado | Alto | dig | Confirmado |
| Server Banner expuesto | Medio | OWASP ZAP | Confirmado |
| Endpoints API y tokens expuestos en JS | Medio | HTTrack + grep | A revisar |
| Identidad del registrante oculta | Medio | whois | Confirmado |
| Dominio joven (< 1 año) | Medio | whois | Confirmado |
| IP compartida (hosting compartido) | Medio | nslookup | Confirmado |
| Infraestructura DNS mapeada públicamente | Bajo | Maltego CE | Confirmado |

---

## Documentación

El análisis completo se encuentra registrado en el documento **F-PIA Framework**, que incluye:

- **4 tablas HPTesting** (una por herramienta): OWASP ZAP, HTTrack, Maltego CE y Burp Suite
- **2 tablas LabPInjection**: SQL Injection (login bypass) y XSS Almacenado
- Metodología PIAP-LLM aplicada a cada prueba
- Hallazgos, impacto, severidad y recomendaciones de mitigación

---

## Cumplimiento Legal

Todos los análisis se realizaron en **entornos controlados y autorizados**, en cumplimiento de la **Ley 1273 de 2009** (Colombia) sobre delitos informáticos y los principios de ética en seguridad ofensiva que rigen la práctica profesional del pentesting.

> **Nota:** Los análisis activos sobre `latinoamericacomparte.com` se limitaron a consultas DNS públicas (reconocimiento pasivo). Las pruebas con Active Scan se realizaron sobre entornos controlados (DVWA, OWASP Juice Shop, PortSwigger Academy).

---

## Recursos

| Recurso | URL |
|---|---|
| OWASP Top 10 | https://owasp.org/www-project-top-ten/ |
| PortSwigger Academy | https://portswigger.net/web-security |
| OWASP Juice Shop | https://demo.owasp-juice.shop |
| OWASP ZAP | https://www.zaproxy.org/ |
| HTTrack Website Copier | https://www.httrack.com/ |
| Maltego CE | https://www.maltego.com/ |
