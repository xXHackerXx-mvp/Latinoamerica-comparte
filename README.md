# Lab 2 — HTTrack | Clonación de Sitio y Análisis de Exposición

## Herramienta: HTTrack Website Copier

HTTrack es una herramienta de código abierto que permite descargar un sitio web completo (espejo/mirror) al sistema local, replicando su estructura de directorios, HTML, CSS, JavaScript e imágenes. En ciberseguridad se usa en la fase de reconocimiento para analizar el código fuente del frontend sin interactuar con el servidor de producción.

---

## Sitio Analizado

**latinoamericacomparte.com** — plataforma orientada a programas sociales, emprendimiento y transformación empresarial en Latinoamérica.

El sitio fue previamente desplegado en Netlify como entorno de pruebas controlado para el laboratorio.

---

## Objetivo

Clonar el sitio objetivo y analizar su código fuente en busca de:

- Rutas de API expuestas en el frontend
- Tokens o claves hardcodeadas
- Llamadas HTTP externas
- Configuraciones inseguras visibles en el cliente

---

## Verificación e Instalación

```bash
httrack --version
```

Si no está instalado:

```bash
sudo apt update
sudo apt install httrack -y
```

---

## Comando de Clonación

```bash
httrack "https://latinoamericacomparte.com/" \
        -O "$HOME/lab_httrack_latinoamerica" \
        "+*.latinoamericacomparte.com/*" \
        "+*.css" "+*.js" "+*.png" "+*.jpg" "+*.jpeg" "+*.gif" \
        -v
```

**Explicación de parámetros:**

| Parámetro | Función |
|---|---|
| `"https://latinoamericacomparte.com/"` | URL objetivo a clonar |
| `-O "$HOME/lab_httrack_latinoamerica"` | Carpeta destino del espejo local |
| `"+*.latinoamericacomparte.com/*"` | Incluye todos los recursos del dominio |
| `"+*.css" "+*.js"` | Incluye hojas de estilo y scripts |
| `"+*.png" "+*.jpg" "+*.gif"` | Incluye imágenes |
| `-v` | Modo verbose: muestra el proceso en tiempo real |

---

## Verificación del Clon

```bash
cd ~/lab_httrack_latinoamerica
ls -la
```

```bash
cd latinoamericacomparte.com
ls -la
```

---

## Servidor Local para Análisis

```bash
cd ~/lab_httrack_latinoamerica
python3 -m http.server 8080 --bind 127.0.0.1
```

Acceder desde el navegador en: `http://127.0.0.1:8080`

---

## Análisis Técnico (Fase White Hat)

Desde el directorio del clon, ejecutar búsquedas en el código fuente:

```bash
grep -r "api" .
grep -r "fetch" .
grep -r "token" .
grep -r "key" .
grep -r "http" .
```

**Interpretación de resultados:**

| Patrón buscado | ¿Qué podría significar? | Riesgo |
|---|---|---|
| `api` | Uso de API externa o interna | Expone endpoints |
| `fetch` | Llamadas HTTP desde el frontend | Revela arquitectura |
| `token` | Token de autenticación hardcodeado | Alto — riesgo de robo |
| `key` | Clave de API embebida en el código | Alto — acceso no autorizado |
| `http` | Llamadas a recursos externos | Dependencias de terceros |

---

## Hallazgos de Seguridad (Blue Team)

### Buenas prácticas que se deben verificar:

| Practica | Correcto | Incorrecto |
|---|---|---|
| Secretos en frontend | `fetch("/api/users")` | `const SECRET_KEY = "abc123"` |
| Variables de entorno | `process.env.API_KEY` (backend) | API key visible en JS |
| Configuración CORS | Solo dominio propio permitido | `Access-Control-Allow-Origin: *` |
| Tokens de sesión | Dinámicos y con expiración | Tokens fijos hardcodeados |
| Rate limiting | 429 Too Many Requests activo | Sin límite de solicitudes |

### Rate Limiting

Técnica del backend que limita cuántas solicitudes puede hacer un usuario o IP en un periodo de tiempo. Protege contra:

- Fuerza bruta de credenciales
- Scraping masivo automatizado
- DDoS básicos
- Abuso de APIs
- Enumeración de usuarios

---

## Resumen de Hallazgos

| Elemento | Estado observado | Impacto |
|---|---|---|
| Código frontend accesible | Sí (sitio estático clonable) | Expone estructura y llamadas API |
| Secretos en JS | A verificar con grep | Potencialmente alto |
| Llamadas HTTP externas | A verificar con grep | Dependencia de terceros |
| Configuración CORS | A verificar | Puede permitir acceso no autorizado |

---

## Contexto: Red Team / Blue Team

| Rol | Uso |
|---|---|
| Red Team | Encontrar endpoints, tokens y rutas ocultas en el frontend |
| Blue Team | Auditar que no existan secretos ni configuraciones inseguras expuestas al cliente |

---

## Herramientas del Laboratorio

| Herramienta | Propósito |
|---|---|
| `httrack` | Clonar el sitio web completo para análisis offline |
| `grep` | Buscar patrones sensibles en el código fuente clonado |
| `python3 -m http.server` | Servir el clon localmente para inspección en navegador |
