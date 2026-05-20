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

## Fases y Herramientas

| Rama | Lab | Herramienta | Fase | Descripción |
|---|---|---|---|---|
| [`lab-nslookup`](../../tree/lab-nslookup) | Lab 3 | nslookup | Reconocimiento Pasivo | Resolución DNS básica: IP, NS, PTR |
| [`lab-dig`](../../tree/lab-dig) | Lab 3 | dig + whois | Reconocimiento Pasivo | Análisis DNS avanzado, DNSSEC, registro del dominio |
| [`lab-httrack`](../../tree/lab-httrack) | Lab 2 | HTTrack | Reconocimiento Pasivo | Clonación del sitio y análisis de exposición frontend |
| [`lab-maltego`](../../tree/lab-maltego) | Lab 4 | Maltego | Reconocimiento OSINT | Grafos de relaciones: IP, NS, subdominios, correos |
| [`lab-burpsuite`](../../tree/lab-burpsuite) | Lab 5 | Burp Suite | Análisis Web | Interceptación HTTP, análisis de autenticación |
| [`lab-injection`](../../tree/lab-injection) | Lab 7 | OWASP ZAP | Análisis de Vulnerabilidades | Spider, Alerts, Active Scan, HTTP Sessions |

---

## Metodología Aplicada

```
Reconocimiento Pasivo → Análisis OSINT → Clonación del sitio
        ↓                                         ↓
  nslookup / dig                              HTTrack
  whois / DNSSEC                         Exposición frontend
        ↓
   Maltego (OSINT visual)
        ↓
Análisis de Tráfico HTTP → Detección de Vulnerabilidades
        ↓                           ↓
    Burp Suite                  OWASP ZAP
  Interceptación              Spider + Active Scan
  Autenticación               Alerts + Sessions
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

- Subdominios expuestos
- IPs públicas y proveedor de hosting
- Tecnologías del servidor
- Certificados SSL y DNSSEC
- Headers HTTP de seguridad
- Cookies y tokens de sesión
- Formularios y endpoints de autenticación
- Rutas y estructura interna del sitio

---

## Riesgos Identificados

| Riesgo | Nivel | Herramienta que lo detectó |
|---|---|---|
| DNSSEC no implementado | Alto | dig |
| Identidad del registrante oculta | Medio | whois |
| Dominio joven (< 1 año) | Medio | whois |
| IP compartida (hosting compartido) | Medio | nslookup |
| Headers de seguridad HTTP | A verificar | OWASP ZAP |
| Cookies de sesión inseguras | A verificar | Burp Suite / ZAP |
| Exposición de frontend | A verificar | HTTrack |

---

## Cumplimiento Legal

Todos los análisis se realizaron en **entornos controlados y autorizados**, en cumplimiento de la **Ley 1273 de 2009** (Colombia) sobre delitos informáticos y los principios de ética en seguridad ofensiva que rigen la práctica profesional del pentesting.

> **Nota:** Los análisis activos sobre `latinoamericacomparte.com` se limitaron a consultas DNS públicas (reconocimiento pasivo). Las pruebas con Active Scan se realizaron sobre entornos controlados (DVWA, Juice Shop).

---

## Recursos

| Recurso | URL |
|---|---|
| OWASP Top 10 | https://owasp.org/www-project-top-ten/ |
| PortSwigger Academy | https://portswigger.net/web-security |
| OWASP Juice Shop | https://demo.owasp-juice.shop |
| OWASP ZAP | https://www.zaproxy.org/ |
