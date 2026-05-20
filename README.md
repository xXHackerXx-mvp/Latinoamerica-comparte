# Lab 7 — OWASP ZAP | Análisis de Seguridad Web e Inyección

## Herramienta: OWASP ZAP (Zed Attack Proxy)

OWASP ZAP es una herramienta de seguridad de código abierto desarrollada y mantenida por la fundación OWASP (Open Web Application Security Project). Actúa como un **proxy interceptor** situado entre el navegador y la aplicación objetivo, capturando, analizando y modificando el tráfico HTTP/HTTPS en tiempo real. Incluye módulos para análisis pasivo, rastreo de rutas, detección automatizada de vulnerabilidades y análisis de sesiones.

> Framework metodológico aplicado: **PIAP-LLM** (F-PIA) — Prompt Injection Analysis and Prevention.

---

## Ficha Técnica

| Campo | Valor |
|---|---|
| Desarrollador | OWASP Foundation |
| Licencia | Apache License 2.0 |
| Versión usada | ZAP 2.17.0 |
| Plataforma | Kali Linux (multiplataforma Java) |
| Entornos objetivo | DVWA (localhost:8080), Juice Shop, SAC USTA, latinoamericacomparte.com |
| Modo de uso | Proxy manual + Automated Scan |

---

## Sitio Analizado

**latinoamericacomparte.com** — plataforma orientada a programas sociales, emprendimiento y transformación empresarial en Latinoamérica.

IP pública: `64.202.187.143` — identificada en fase de reconocimiento (ver ramas `lab-nslookup` y `lab-dig`).

---

## Objetivo

Analizar la superficie de ataque web del sitio objetivo mediante:

- Rastreo de rutas y endpoints (Spider / AJAX Spider)
- Detección automatizada de vulnerabilidades (Alerts)
- Análisis de cookies y sesiones (HTTP Sessions)
- Interceptación de solicitudes HTTP/HTTPS (Proxy)
- Identificación de vectores de inyección (SQL, XSS, Path Traversal)

---

## Instalación

```bash
# Verificar si ZAP está disponible
zaproxy

# Instalar si no está presente
sudo apt update
sudo apt install zaproxy -y

# Confirmar versión instalada
zaproxy --version
# => Found Java version 21.0.9
# => Available memory: 10238 MB
```

---

## Configuración del Entorno

### Proxy Firefox → ZAP

| Campo | Valor |
|---|---|
| Ruta | Preferencias → Red → Conexión → Configuración manual |
| HTTP Proxy | 127.0.0.1 |
| Puerto ZAP | 8090 (evitar conflicto con servicios en 8080) |

> En ZAP: `Tools → Options → Local Servers/Proxies → Port: 8090`

---

## Módulos Utilizados

### Módulo 1 — History (Historial de Tráfico)

Registra de forma **pasiva** todo el tráfico HTTP interceptado: método, URL, código de respuesta, tamaño y tiempo de respuesta.

**Interpretación de códigos HTTP:**

| Código | Significado | Implicación de seguridad |
|---|---|---|
| 200 | OK | Recurso accesible — auditar contenido |
| 301 | Redirección | Verificar destino; posible fuga de información |
| 404 | No encontrado | Confirma existencia de estructura de directorios |
| 500 | Error de servidor | Expone detalles del backend (DBMS, rutas internas) |

> **Hallazgo crítico:** errores HTTP 500 revelan tecnología de base de datos, rutas internas y configuración defectuosa — facilita la fase de fingerprinting.

---

### Módulo 2 — Alerts (Vulnerabilidades Detectadas)

El panel de Alerts consolida todos los hallazgos detectados en modo pasivo y activo.

**Vulnerabilidades comunes detectadas (referencia DVWA / Juice Shop):**

| Vulnerabilidad | Riesgo | Descripción |
|---|---|---|
| Content Security Policy (CSP) Missing | Alto | Sin cabecera CSP; facilita ataques XSS |
| Cookie sin HttpOnly | Alto | Cookie de sesión robable vía JavaScript |
| Cookie sin SameSite | Medio | Permite ataques CSRF desde dominios externos |
| Anti-Clickjacking Header Missing | Medio | Sin `X-Frame-Options`; vulnerable a clickjacking |
| Server Version Expuesta | Medio | Cabecera `Server` revela versión de Apache/PHP |
| Directory Browsing | Medio | Listado de directorios activo |
| Application Error Disclosure | Alto | Mensajes de error con información interna del sistema |
| X-Content-Type-Options Missing | Bajo | Ausencia de `nosniff`; MIME sniffing posible |

**Aplicación a latinoamericacomparte.com:**

| Elemento a verificar | Herramienta | Riesgo esperado |
|---|---|---|
| Headers de seguridad HTTP | ZAP Alerts (pasivo) | CSP, X-Frame-Options, HSTS |
| Cookies de sesión | ZAP HTTP Sessions | HttpOnly, Secure, SameSite |
| Endpoints de login | ZAP History + Alerts | Autenticación, tokens |
| Formularios del sitio | ZAP Spider + Active Scan | XSS, SQLi |

---

### Módulo 3 — Spider (Reconocimiento de Rutas)

El Spider sigue automáticamente todos los enlaces HTML para construir un mapa completo de la aplicación.

```
Estadísticas Spider — DVWA (localhost:8080):
URLs descubiertas: 92+
Estado: 100% completado
```

**Rutas de interés típicas:**

| Ruta descubierta | Relevancia |
|---|---|
| `/login.php` | Punto de autenticación |
| `/robots.txt` | Puede revelar rutas ocultas |
| `/hackable/uploads/` | Directorio de subidas |
| `/sitemap.xml` | Mapa oficial del sitio |

> **Nota:** `robots.txt` con directivas `Disallow` actúa como un mapa involuntario de rutas de interés para un atacante.

---

### Módulo 4 — Active Scan (Ataque Automatizado)

Envía solicitudes HTTP maliciosas para detectar vulnerabilidades explotables.

```
Estadísticas Active Scan — DVWA:
Total de requests enviados: 3,000+
Técnicas probadas: SQLi, XSS, Path Traversal, Headers, RCE
```

**Payloads enviados automáticamente:**

```bash
# SQL Injection
?id=1
?id=1'
?id=1 OR 1=1
?id=1' AND SLEEP(5)--

# Cross-Site Scripting (XSS)
?name=<script>alert(1)</script>
?name=<img src=x onerror=alert(1)>

# Path Traversal
?page=../../etc/passwd
?page=....//....//etc/passwd

# Headers inseguros auditados
X-Content-Type-Options: nosniff (ausente)
X-Frame-Options: DENY (ausente)
Content-Security-Policy (ausente)
```

---

### Módulo 5 — HTTP Sessions (Gestión de Sesiones)

Identifica y gestiona las cookies de sesión capturadas.

**Análisis de cookie crítica (PHPSESSID):**

| Atributo | Estado encontrado | Implicación |
|---|---|---|
| HttpOnly | ❌ No configurado | Robable vía JavaScript (XSS) |
| SameSite | ❌ No configurado | Vulnerable a CSRF |
| Secure | ❌ No configurado | Transmisible en HTTP plano |

> **Riesgo:** robar la cookie equivale a tomar control total de la sesión del usuario.

---

### Módulo 6 — AJAX Spider

Utiliza un navegador real en modo headless para ejecutar JavaScript y descubrir rutas de SPAs (Angular, React, Vue).

**Comparativa Spider vs AJAX Spider:**

| Característica | Spider Clásico | AJAX Spider |
|---|---|---|
| Motor | Análisis HTML estático | Navegador real (headless) |
| JavaScript | No ejecuta JS | Ejecuta completamente |
| Interacción | Solo sigue enlaces | Clicks, scroll, formularios |
| Uso ideal | Apps tradicionales | SPA (Angular, React, Vue) |
| Rutas dinámicas | No las detecta | Las descubre y sigue |

**Resultado en Juice Shop (SPA Angular):**

AJAX Spider descubrió endpoints de API REST invisibles para el Spider clásico:
- `/api/Products`
- `/api/Users`
- Rutas de administración ocultas

---

### Módulo 7 — WebSockets

Intercepta mensajes en canales WebSocket (comunicación bidireccional en tiempo real).

```json
// Mensaje WebSocket interceptado (ejemplo vulnerable):
{ "user": "admin", "action": "login", "token": "abc123" }

// Modificado por el atacante (sin validación del token):
{ "user": "admin", "action": "deleteAll", "token": "abc123" }
```

---

## Pruebas en Entornos Controlados

### Prueba 1 — DVWA (localhost:8080)

Entorno controlado levantado con Docker:

```bash
sudo docker run --rm -it -p 8080:80 vulnerables/web-dvwa
```

**Resultado:** vulnerabilidades SQLi, XSS, Command Injection confirmadas en nivel Low.

---

### Prueba 2 — OWASP Juice Shop

**URL:** `https://demo.owasp-juice.shop`

**Solicitud de login interceptada con Burp Suite:**

```http
POST /rest/user/login HTTP/2
Host: demo.owasp-juice.shop
Content-Type: application/json

{
  "email": "test@test.com",
  "password": "testpassword"
}
```

**Respuesta del servidor (credenciales incorrectas):**

```json
{
  "status": "401 Unauthorized",
  "error": "Invalid email or password."
}
```

| Elemento | Valor | Análisis |
|---|---|---|
| Método | POST | Envío de credenciales |
| Formato | JSON | Estructura del body |
| Endpoint | `/rest/user/login` | Ruta de autenticación expuesta |
| Respuesta error | 401 + mensaje | No enumera si el usuario existe |

**AJAX Spider — Juice Shop:**

- Estado del escaneo: `Attack complete`
- Spider utilizado: AJAX Spider (SPA Angular)
- Alertas: Múltiples (SQLi, XSS, Path Traversal, Headers)
- Rutas descubiertas: `/api/Products`, `/api/Users`

---

### Prueba 3 — SAC USTA (sac.usantotomas.edu.co)

Escaneo **pasivo observacional** sin Active Scan — cumplimiento de la Ley 1273 de 2009 (Colombia).

> Cualquier prueba activa sobre sistemas de terceros sin autorización explícita constituye un delito informático. Este ejercicio se limitó a observar el tráfico generado por la propia navegación.

---

## Resumen de Módulos ZAP

| Módulo | Tipo | Función principal |
|---|---|---|
| History | Pasivo | Registra todo el tráfico HTTP capturado |
| Alerts | Pasivo/Activo | Consolida vulnerabilidades detectadas |
| Spider | Pasivo | Rastreo de rutas HTML estáticas |
| AJAX Spider | Pasivo | Rastreo de SPAs con JavaScript real |
| Active Scan | Activo | Ataque automatizado con payloads |
| HTTP Sessions | Pasivo | Gestión y análisis de cookies de sesión |
| WebSockets | Pasivo/Activo | Interceptación de mensajes en tiempo real |

---

## Generación del Reporte ZAP

```bash
# Desde la interfaz gráfica:
Report → Generate Report → Formato HTML / PDF → Guardar

# El reporte incluye:
# - Resumen ejecutivo con conteo de alertas por nivel
# - Detalle técnico de cada vulnerabilidad
# - URL afectada + método HTTP
# - Solución recomendada + referencia CWE/OWASP
```

---

## Metodología Aplicada (F-PIA Framework)

Este laboratorio sigue la metodología del **Framework PIAP-LLM** (Prompt Injection Analysis and Prevention):

| Fase | Herramienta | Acción |
|---|---|---|
| Reconocimiento pasivo | Spider + History + HTTP Sessions | Mapa de la aplicación |
| Análisis de superficie | Alerts + AJAX Spider | Vulnerabilidades detectadas |
| Explotación controlada | Active Scan + WebSockets | Payloads de prueba |
| Documentación | Generate Report | Reporte estructurado |

---

## Hallazgos de Seguridad — Aplicación a latinoamericacomparte.com

| Elemento | Acción ZAP | Riesgo potencial |
|---|---|---|
| Headers de seguridad | Alerts pasivo | CSP, HSTS, X-Frame-Options ausentes |
| Formularios del sitio | Spider + Active Scan | XSS, SQLi en inputs |
| Cookies de sesión | HTTP Sessions | Falta de HttpOnly / Secure / SameSite |
| Rutas de API | AJAX Spider | Endpoints REST expuestos |
| Redirecciones | History | Posible fuga de información en 301/302 |

---

## Contexto: Red Team / Blue Team

| Rol | Uso de OWASP ZAP |
|---|---|
| Red Team | Mapear endpoints, detectar vectores de inyección, capturar sesiones |
| Blue Team | Auditar headers, validar cookies, verificar CSP y controles de autenticación |

---

## Recursos

| Recurso | URL |
|---|---|
| OWASP ZAP | https://www.zaproxy.org/ |
| OWASP Top 10 | https://owasp.org/www-project-top-ten/ |
| OWASP Juice Shop | https://owasp.org/www-project-juice-shop/ |
| PortSwigger Academy | https://portswigger.net/web-security |
| Ley 1273 (Colombia) | Delitos informáticos — marco legal del pentesting ético |
