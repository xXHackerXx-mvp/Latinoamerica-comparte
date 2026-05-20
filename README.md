# Lab 5 — Burp Suite | Interceptación y Análisis de Tráfico HTTP

## Herramienta: Burp Suite Community Edition

Burp Suite es una plataforma profesional para pruebas de seguridad en aplicaciones web. Funciona como un **proxy de interceptación** que se sitúa entre el navegador y el servidor, permitiendo capturar, analizar y modificar el tráfico HTTP/HTTPS en tiempo real. Es la herramienta estándar de la industria para pentesting de aplicaciones web y evaluación de vulnerabilidades OWASP Top 10.

**Desarrollado por:** PortSwigger — empresa líder en seguridad de aplicaciones web.

---

## Sitio Analizado

**latinoamericacomparte.com** — plataforma orientada a programas sociales, emprendimiento y transformación empresarial en Latinoamérica.

Pruebas realizadas también sobre entornos controlados: OWASP Juice Shop y PortSwigger Web Security Academy.

---

## Objetivo

Interceptar y analizar el tráfico HTTP del sitio para identificar:

- Estructura de solicitudes de autenticación
- Parámetros del login (endpoint, método, formato)
- Cookies de sesión y su configuración de seguridad
- Headers HTTP y sus implicaciones de seguridad
- Comportamiento del servidor ante credenciales incorrectas

---

## Módulos Utilizados

| Módulo | Función |
|---|---|
| **Proxy** | Intercepta tráfico HTTP entre navegador y servidor |
| **HTTP History** | Registra todas las solicitudes y respuestas capturadas |
| **Intercept** | Pausa y permite modificar una solicitud antes de enviarla |
| **Repeater** | Reenvía solicitudes modificadas para analizar respuestas |
| **Inspector** | Decodifica y muestra parámetros, cookies y headers |

---

## Conceptos Clave

| Concepto | Explicación |
|---|---|
| Proxy | Intermediario entre el cliente (navegador) y el servidor |
| Endpoint | Ruta del servidor que recibe la solicitud (ej: `/rest/user/login`) |
| Payload | Datos enviados en el cuerpo de la petición |
| HTTP Request | Solicitud enviada al servidor (método + headers + body) |
| HTTP Response | Respuesta del servidor (código de estado + headers + body) |

---

## Flujo del Laboratorio

### Paso 1 — Configuración inicial

1. Abrir Burp Suite → **Temporary project in memory**
2. Seleccionar **Use Burp defaults**
3. Ir a `Proxy > Intercept`
4. Hacer clic en **Open browser** (navegador integrado de Burp)

---

### Paso 2 — Activar Interceptación

En Burp Suite: `Proxy > Intercept > Intercept ON`

---

### Prueba 1 — Análisis del Login de latinoamericacomparte.com

#### Procedimiento:
1. Navegar al formulario de login del sitio
2. Ingresar credenciales de prueba (sin enviar aún)
3. Activar Intercept ON en Burp Suite
4. Hacer clic en el botón de inicio de sesión
5. Capturar la solicitud en Burp Suite

#### Solicitud interceptada (estructura esperada):

```http
POST /[endpoint-login] HTTP/2
Host: latinoamericacomparte.com
Content-Type: application/json

{
  "email": "test@test.com",
  "password": "testpassword"
}
```

#### Análisis:

| Campo | Valor | Significado |
|---|---|---|
| Método | POST | Envío de datos de autenticación |
| Content-Type | application/json | Formato JSON para las credenciales |
| Endpoint | `/[ruta-login]` | Ruta que maneja la autenticación |

---

### Paso 3 — Enviar al Repeater

Clic derecho sobre la solicitud interceptada → **Send to Repeater** (`Ctrl+R`)

En el Repeater se puede:
- Modificar credenciales y reenviar
- Analizar la respuesta del servidor
- Probar distintos payloads sin usar el navegador

---

### Prueba 2 — OWASP Juice Shop (Entorno Controlado)

**URL:** `https://demo.owasp-juice.shop`

#### Solicitud interceptada:

```http
POST /rest/user/login HTTP/2
Host: demo.owasp-juice.shop
Content-Type: application/json

{
  "email": "test@test.com",
  "password": "testpassword"
}
```

#### Respuesta del servidor (credenciales incorrectas):
```json
{
  "status": "401 Unauthorized",
  "error": "Invalid email or password."
}
```

---

### Prueba 3 — PortSwigger Web Security Academy

**Lab:** SQL Injection — Login Bypass

**URL:** `https://portswigger.net/web-security/sql-injection/lab-login-bypass`

#### Solicitud interceptada:

```http
POST /login HTTP/2
Host: [lab-id].web-security-academy.net
Content-Type: multipart/form-data

username=administrator'--&password=testusta2026
```

#### Técnica aplicada:

Modificar el parámetro `username` con el valor `administrator'--` (SQL Injection):

| Payload | Efecto |
|---|---|
| `administrator'--` | Comenta el resto de la consulta SQL, omitiendo verificación de contraseña |
| `' OR 1=1--` | Retorna verdadero en cualquier consulta SQL |

#### Respuesta: `302 Found` → redirección al panel de administrador.

---

## Comparativa de Formatos de Login

| Aplicación | Formato del Body | Endpoint | Método HTTP |
|---|---|---|---|
| Juice Shop | JSON | `/rest/user/login` | POST |
| PortSwigger | multipart/form-data | `/login` | POST |
| SAC USTA | JSON codificado en Base64 | `/sgacampus/...` | POST |
| latinoamericacomparte.com | A verificar con Burp | A identificar | POST |

---

## Hallazgos de Seguridad a Verificar

| Elemento | Qué buscar | Riesgo si falla |
|---|---|---|
| Cookies de sesión | Flags `HttpOnly` y `Secure` | Session Hijacking |
| Headers de seguridad | `Content-Security-Policy`, `X-Frame-Options` | XSS, Clickjacking |
| Tokens CSRF | Presencia en formularios | Cross-Site Request Forgery |
| Exposición de datos en respuesta | Datos sensibles en JSON de error | Fuga de información |
| Código de estado en fallo de login | `401` o `403` (no `200`) | Enumeración de usuarios |

---

## Cuestionario de Análisis

1. ¿Cuál es el endpoint exacto que maneja el login del sitio?
2. ¿Qué formato usa el body de la solicitud? (JSON / form-data / Base64)
3. ¿Qué método HTTP se utiliza para la autenticación?
4. ¿Qué código de estado retorna el servidor ante credenciales incorrectas?
5. ¿Qué cookies se crean al iniciar sesión? ¿Tienen los flags `HttpOnly` y `Secure`?
6. ¿Existen headers de seguridad en la respuesta?

---

## Contexto: Red Team / Blue Team

| Rol | Uso |
|---|---|
| Red Team | Interceptar solicitudes para identificar endpoints, parámetros y mecanismos de autenticación vulnerables |
| Blue Team | Verificar que las solicitudes estén correctamente protegidas con HTTPS, CSRF tokens y cookies seguras |

---

## Recursos de Práctica

| Entorno | URL | Propósito |
|---|---|---|
| OWASP Juice Shop | https://demo.owasp-juice.shop | App vulnerable intencional |
| PortSwigger Academy | https://portswigger.net/web-security | Labs guiados de seguridad web |
| SQL Injection Lab | https://portswigger.net/web-security/sql-injection/lab-login-bypass | Lab: bypass de login con SQLi |

---

## Herramientas del Laboratorio

| Herramienta | Propósito |
|---|---|
| Burp Suite | Interceptar, analizar y modificar tráfico HTTP/HTTPS |
| Repeater | Reenviar solicitudes modificadas y analizar respuestas |
| Inspector | Decodificar parámetros, cookies y headers en tiempo real |
