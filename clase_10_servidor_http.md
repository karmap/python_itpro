# Clase 10: Servidor HTTP desde cero

**Duración:** 2 horas


> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.
**Objetivo:** entender cómo funciona un servidor HTTP construyendo uno simple con Python, consumirlo desde otro programa, probarlo con `curl` y abrirlo desde el navegador.

---

## 1. ¿Qué vamos a construir?

Hasta ahora:

```text
Nuestro programa
      ↓
requests
      ↓
API externa
```

Ahora invertimos el flujo:

```text
Cliente
  │
  │ HTTP Request
  ▼
Nuestro servidor Python
  │
  │ HTTP Response
  ▼
Cliente
```

Lo probaremos desde:

```text
Navegador
curl
requests
   │
   └────────────→ Servidor Python
                    │
                    └→ JSON
```

La idea principal:

```text
cliente
   ↓
request
   ↓
servidor
   ↓
ruta
   ↓
response
   ↓
cliente
```

---

## 2. Primer servidor HTTP

### Clases y métodos que vamos a usar

`class Handler(BaseHTTPRequestHandler)` define una clase que hereda el comportamiento HTTP de otra. Un método es una función dentro de la clase; `self` representa la instancia que atiende la petición. La biblioteca crea esa instancia y llama a `do_GET()` cuando recibe GET. No necesitamos desarrollar una clase completa desde cero.

Una clase describe un tipo de objeto y una instancia es un objeto concreto de ese tipo. Heredar permite reutilizar sus métodos y reemplazar uno, como `do_GET()`, con nuestro comportamiento. `HTTPServer(("localhost", 8000), Handler)` crea el objeto servidor: recibe una tupla con dirección y puerto, y la clase que usará para atender peticiones. Pasamos `Handler` sin paréntesis porque la biblioteca creará las instancias.

El cuerpo se transmite como `bytes`, una secuencia de datos binarios. `b"Hola"` es un literal de bytes para texto ASCII; una cadena con acentos se convierte con `.encode("utf-8")`. La operación inversa es `.decode("utf-8")`. El JSON es texto antes de esa conversión.

Los métodos `enviar_json()` y `do_GET()` de los fragmentos siguientes deben ir dentro de `Handler`. Cada versión completa de `server.py` reemplaza la anterior. Reinicia con Ctrl+C y `python server.py` después de cambiar el código.

Este servidor es una demostración local para entender HTTP; no lo publicaremos como servidor de producción.

Python incluye herramientas para crear un servidor HTTP básico sin instalar librerías externas.

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlsplit
```

Crea:

```text
server.py
```

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlsplit


class Handler(BaseHTTPRequestHandler):

    def do_GET(self):
        self.send_response(200)
        self.end_headers()

        self.wfile.write(
            b"Hola desde Python"
        )


def main():
    server = HTTPServer(("localhost", 8000), Handler)
    print("Servidor corriendo en http://localhost:8000")
    try:
        server.serve_forever()
    except KeyboardInterrupt:
        print("\nServidor detenido")
    finally:
        server.server_close()


if __name__ == "__main__":
    main()
```

Ejecuta:

```bash
python server.py
```

La respuesta se construye en este orden: `send_response(200)` establece el estado; `send_header()` añade cabeceras si las hay; `end_headers()` termina esa sección; `wfile.write()` escribe el cuerpo como bytes. `wfile` es el flujo de salida al cliente; `rfile` será el flujo de entrada para leer un cuerpo POST.

`serve_forever()` mantiene el servidor escuchando. Ctrl+C genera `KeyboardInterrupt`; `finally` llama a `server_close()` para liberar el puerto, siguiendo el patrón de la clase 06.

El programa se queda ejecutándose porque está esperando requests.

---

## 3. `localhost` y puerto

```text
localhost
```

significa:

> esta misma computadora

También:

```text
127.0.0.1
```

Nuestro servidor escucha en:

```text
localhost:8000
```

El `8000` es el puerto.

```text
Computadora
│
├── :3000
├── :5000
├── :8000 ← nuestro servidor
└── :8080
```

---

## 4. Probar desde el navegador

Abre:

```text
http://localhost:8000
```

El navegador hace:

```text
GET /
```

El servidor ejecuta:

```python
def do_GET(self):
    ...
```

y responde:

```text
200 OK
```

con:

```text
Hola desde Python
```

Flujo:

```text
Browser
   │
   │ GET /
   ▼
server.py
   │
   │ 200 OK
   │ Hola desde Python
   ▼
Browser
```

---

## 5. Código de estado

```python
self.send_response(200)
```

envía:

```text
200 OK
```

Otros códigos comunes:

```text
200 OK
201 Created
400 Bad Request
404 Not Found
500 Internal Server Error
```

---

## 6. Headers (cabeceras) y JSON

Modifica el servidor:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlsplit


class Handler(BaseHTTPRequestHandler):

    def do_GET(self):
        self.send_response(200)

        self.send_header(
            "Content-Type",
            "application/json"
        )

        self.end_headers()

        self.wfile.write(
            b'{"mensaje": "Hola desde Python"}'
        )


def main():
    server = HTTPServer(("localhost", 8000), Handler)
    print("Servidor corriendo en http://localhost:8000")
    try:
        server.serve_forever()
    except KeyboardInterrupt:
        print("\nServidor detenido")
    finally:
        server.server_close()


if __name__ == "__main__":
    main()
```

Ahora enviamos:

```text
Content-Type: application/json
```

para indicar que el body es JSON.

---

## 7. ¿Por qué `b"..."`?

```python
b"Hola"
```

es `bytes`.

```python
"Hola"
```

es `str`.

Compruébalo:

```python
print(type("Hola"))
print(type(b"Hola"))
```

Resultado:

```text
<class 'str'>
<class 'bytes'>
```

`wfile.write()` necesita bytes.

---

## 8. Probar con `curl`

En otra terminal:

Los comandos `curl` mostrados usan Bash/zsh. En Windows utiliza `curl.exe` si `curl` es un alias de PowerShell.

```bash
curl http://localhost:8000
```

Respuesta:

```json
{"mensaje": "Hola desde Python"}
```

Para ver también headers:

```bash
curl -i http://localhost:8000
```

Resultado aproximado:

```text
HTTP/1.0 200 OK
Server: BaseHTTP/0.6 Python/3.x
Date: ...
Content-Type: application/json

{"mensaje": "Hola desde Python"}
```

Esto deja visibles:

```text
STATUS
HEADERS
BODY
```

---

## 9. Consumir desde otro programa Python

Crea:

```text
client.py
```

```python
import requests


response = requests.get(
    "http://localhost:8000",
    timeout=10
)

response.raise_for_status()
print(response.status_code)
print(response.text)
```

Ejecuta mientras `server.py` sigue corriendo:

```bash
python client.py
```

Salida:

```text
200
{"mensaje": "Hola desde Python"}
```

---

## 10. Convertir JSON a Python

```python
data = response.json()

print(data)
print(type(data))
```

Resultado:

```python
{
    "mensaje": "Hola desde Python"
}
```

Tipo:

```text
<class 'dict'>
```

Tenemos dos programas hablando:

```text
client.py                          server.py

requests.get() ────────────────→ do_GET()

               ←───────────────
                    JSON
```

---

## Actividad 1: Cambiar la respuesta

Modifica el servidor para devolver:

```json
{
    "curso": "Python",
    "clase": 10,
    "tema": "HTTP Server",
    "activo": true
}
```

Pruébalo desde:

1. navegador;
2. `curl`;
3. `client.py`.

Desde `client.py`, imprime únicamente:

```text
Python
HTTP Server
```

---

## 11. Recordatorio de JSON y conversión a bytes

Ya usamos `json.dumps()` en la clase 08. La novedad es codificar su texto en bytes para escribirlo en la respuesta HTTP.

No queremos construir JSON a mano:

```python
b'{"mensaje": "Hola"}'
```

Usamos:

```python
import json
```

Ejemplo:

```python
data = {
    "mensaje": "Hola desde Python",
    "version": 1
}

body = json.dumps(data)

print(body)
```

Todavía es `str`.

El servidor necesita bytes:

```python
body = json.dumps(data).encode("utf-8")
```

Flujo:

```text
dict
 ↓
json.dumps()
 ↓
str
 ↓
.encode("utf-8")
 ↓
bytes
 ↓
HTTP Response
```

---

## 12. Servidor usando `json.dumps()`

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlsplit
import json


class Handler(BaseHTTPRequestHandler):

    def do_GET(self):

        data = {
            "mensaje": "Hola desde Python",
            "version": 1
        }

        body = json.dumps(data).encode("utf-8")

        self.send_response(200)

        self.send_header(
            "Content-Type",
            "application/json"
        )

        self.end_headers()

        self.wfile.write(body)


def main():
    server = HTTPServer(("localhost", 8000), Handler)
    print("Servidor corriendo en http://localhost:8000")
    try:
        server.serve_forever()
    except KeyboardInterrupt:
        print("\nServidor detenido")
    finally:
        server.server_close()


if __name__ == "__main__":
    main()
```

---

## 13. Crear varias rutas

Queremos:

```text
GET /
GET /health
GET /products
```

Podemos revisar:

```python
self.path
```

`self.path` contiene también la query. Con `urlsplit(self.path).path` obtenemos solo la ruta; así `/products?min_price=1000` sigue llegando a `/products`, aunque esta API manual todavía no aplica filtros. FastAPI hará ese filtrado en la clase 11.

Ejemplo:

```python
def do_GET(self):

    if urlsplit(self.path).path == "/":
        ...

    elif urlsplit(self.path).path == "/health":
        ...

    elif urlsplit(self.path).path == "/products":
        ...
```

---

## 14. Ruta `/health`

```python
if urlsplit(self.path).path == "/health":

    data = {
        "status": "ok"
    }
```

Prueba:

```text
http://localhost:8000/health
```

o:

```bash
curl http://localhost:8000/health
```

Resultado:

```json
{
    "status": "ok"
}
```

---

## 15. Ruta `/products`

```python
productos = [
    {
        "id": 1,
        "nombre": "Laptop",
        "precio": 15000
    },
    {
        "id": 2,
        "nombre": "Mouse",
        "precio": 450
    },
    {
        "id": 3,
        "nombre": "Monitor",
        "precio": 4200
    }
]
```

Para `/products`:

Fragmento dentro de `do_GET()` (requiere los `if` anteriores):

```text
elif urlsplit(self.path).path == "/products":
    data = productos
```

Luego:

```python
body = json.dumps(data).encode("utf-8")
```

---

## 16. Evitar repetir código

Crea un método dentro de `Handler`:

```python
def enviar_json(self, data, status=200):

    body = json.dumps(data).encode("utf-8")

    self.send_response(status)

    self.send_header(
        "Content-Type",
        "application/json"
    )

    self.end_headers()

    self.wfile.write(body)
```

Entonces:

```python
if urlsplit(self.path).path == "/":

    self.enviar_json({
        "mensaje": "Mi API"
    })

elif urlsplit(self.path).path == "/health":

    self.enviar_json({
        "status": "ok"
    })

elif urlsplit(self.path).path == "/products":

    self.enviar_json(productos)
```

---

## 17. Ruta no encontrada

Fragmento final dentro de `do_GET()`:

```text
else:

    self.enviar_json(
        {
            "error": "Ruta no encontrada"
        },
        404
    )
```

Prueba:

```bash
curl -i http://localhost:8000/abc
```

Debe responder:

```text
404 Not Found
```

y:

```json
{
    "error": "Ruta no encontrada"
}
```

---

## 18. Servidor completo

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlsplit
import json


productos = [
    {
        "id": 1,
        "nombre": "Laptop",
        "precio": 15000
    },
    {
        "id": 2,
        "nombre": "Mouse",
        "precio": 450
    },
    {
        "id": 3,
        "nombre": "Monitor",
        "precio": 4200
    }
]


class Handler(BaseHTTPRequestHandler):

    def enviar_json(
        self,
        data,
        status=200
    ):

        body = json.dumps(data).encode("utf-8")

        self.send_response(status)

        self.send_header(
            "Content-Type",
            "application/json"
        )

        self.end_headers()

        self.wfile.write(body)


    def do_GET(self):

        if urlsplit(self.path).path == "/":

            self.enviar_json({
                "mensaje": "Mi primera API"
            })

        elif urlsplit(self.path).path == "/health":

            self.enviar_json({
                "status": "ok"
            })

        elif urlsplit(self.path).path == "/products":

            self.enviar_json(productos)

        else:

            self.enviar_json(
                {
                    "error": "Ruta no encontrada"
                },
                404
            )


def main():
    server = HTTPServer(("localhost", 8000), Handler)
    print("Servidor corriendo en http://localhost:8000")
    try:
        server.serve_forever()
    except KeyboardInterrupt:
        print("\nServidor detenido")
    finally:
        server.server_close()


if __name__ == "__main__":
    main()
```

---

## Actividad 2: Agregar rutas

Agrega:

```text
GET /info
GET /users
```

### `/info`

```json
{
    "nombre": "Mi API",
    "version": "1.0",
    "lenguaje": "Python"
}
```

### `/users`

```json
[
    {
        "id": 1,
        "nombre": "Ana"
    },
    {
        "id": 2,
        "nombre": "Luis"
    },
    {
        "id": 3,
        "nombre": "Carlos"
    }
]
```

Prueba ambas rutas desde:

```text
browser
curl
requests
```

---

## 19. Consumir `/products` desde Python

`client.py`:

```python
import requests


response = requests.get(
    "http://localhost:8000/products",
    timeout=10
)

response.raise_for_status()

productos = response.json()


for producto in productos:

    print(
        producto["nombre"],
        producto["precio"]
    )
```

Resultado:

```text
Laptop 15000
Mouse 450
Monitor 4200
```

Antes:

```text
requests → servidor externo
```

Ahora:

```text
requests → nuestro servidor
```

---

## 20. Navegador vs `curl` vs `requests`

Los tres son clientes HTTP.

```text
NAVEGADOR ─┐
CURL ──────┼──→ HTTP Server
REQUESTS ──┘
```

El servidor recibe HTTP sin importar qué cliente hizo el request.

---

## 21. ¿Qué está haciendo nuestro servidor?

```text
Recibir request
      ↓
Detectar método HTTP
      ↓
Leer ruta
      ↓
Elegir código Python
      ↓
Crear respuesta
      ↓
Serializar JSON
      ↓
Enviar status
      ↓
Enviar headers
      ↓
Enviar body
```

```python
def do_GET(self):
    ...
```

corresponde al método:

```text
GET
```

Y:

```python
self.path
```

nos permite identificar la ruta.

---

## 22. ¿Qué pasa con POST?

También podríamos implementar:

```python
def do_POST(self):
    ...
```

Tendríamos que leer manualmente el body:

```python
length = int(self.headers.get("Content-Length", "0"))

body = self.rfile.read(length)
```

Después:

```python
data = json.loads(body)
```

Este fragmento es conceptual: para un POST completo hay que manejar un tamaño ausente o inválido, JSON mal formado y campos incorrectos antes de guardar datos. No agregaremos ese endpoint manual; en la clase 11 veremos cómo FastAPI y Pydantic resuelven gran parte del trabajo.

Por ejemplo:

```text
¿Existe nombre?
¿Existe precio?
¿Precio es realmente un número?
```

Esto comienza a generar bastante código manual.

---

## 23. ¿Qué problema aparece?

Ya manejamos rutas, respuestas y errores. También vimos qué trabajo haría falta para implementar POST:

```text
routes
status codes
headers
JSON serialization
body parsing
validation
error handling
```

Con 2 o 3 rutas todavía es manejable.

Con:

```text
20 rutas
50 rutas
100 rutas
```

se vuelve mucho más difícil.

---

## 24. Puente a FastAPI

Con nuestro servidor manual:

```python
class Handler(BaseHTTPRequestHandler):

    def do_GET(self):

        if urlsplit(self.path).path == "/products":
            ...
```

Con FastAPI veremos:

```python
@app.get("/products")
def get_products():
    return productos
```

En lugar de:

```python
body = json.dumps(productos).encode("utf-8")
```

podremos:

```python
return productos
```

FastAPI automatiza estas tareas, usando Pydantic para los modelos y la validación. Nosotros elegimos los códigos HTTP de creación o error:

```text
routing
JSON
headers
validation
status codes
documentation
```

Además tendremos:

```text
/docs
```

con documentación automática.

---

## Ejercicio final: Mini Products API

Construye un servidor con:

```text
GET /
GET /health
GET /products
GET /users
GET /info
```

### `/`

```json
{
    "mensaje": "Bienvenido a mi API"
}
```

### `/health`

```json
{
    "status": "ok"
}
```

### `/products`

Debe devolver al menos 5 productos.

### `/users`

Debe devolver al menos 5 usuarios.

### `/info`

```json
{
    "nombre": "Products API",
    "version": "1.0",
    "lenguaje": "Python"
}
```

Cualquier otra ruta debe responder:

```text
404
```

y:

```json
{
    "error": "Ruta no encontrada"
}
```

---

## Cliente del ejercicio

Crea también:

```text
client.py
```

Debe consultar:

```text
/products
```

y mostrar:

```text
PRODUCTOS

1 - Laptop - $15000
2 - Mouse - $450
3 - Monitor - $4200
...
```

Además:

- usar `timeout`;
- usar `raise_for_status()`;
- manejar `requests.RequestException`.

---

## Práctica opcional

Haz que `client.py` consulte primero:

```text
/health
```

Si recibe:

```json
{
    "status": "ok"
}
```

entonces consulta:

```text
/products
```

Si el servidor no está disponible:

```text
No fue posible conectar con la API.
```

---

## Cheat sheet

### Crear servidor

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import urlsplit
```

```python
server = HTTPServer(
    ("localhost", 8000),
    Handler
)

server.serve_forever()
```

### GET

```python
def do_GET(self):
    ...
```

### Ruta

```python
self.path
```

### Status

```python
self.send_response(200)
```

### Header

```python
self.send_header(
    "Content-Type",
    "application/json"
)
```

### Finalizar headers

```python
self.end_headers()
```

### Python → JSON

```python
json.dumps(data)
```

### String → bytes

```python
texto.encode("utf-8")
```

### Escribir respuesta

```python
self.wfile.write(body)
```

### Cliente Python

```python
response = requests.get(
    "http://localhost:8000/products"
)
```

### Curl

```bash
curl http://localhost:8000/products
```

### Curl + headers

```bash
curl -i http://localhost:8000/products
```

### Navegador

```text
http://localhost:8000/products
```

---

## Flujo completo

```text
Browser / curl / requests
          │
          │ HTTP GET
          ▼
     localhost:8000
          │
          ▼
       Handler
          │
          ▼
       do_GET()
          │
          ▼
       self.path
          │
          ▼
        route
          │
          ▼
      Python data
          │
          ▼
     json.dumps()
          │
          ▼
       .encode("utf-8")
          │
          ▼
      HTTP Response
          │
          ▼
Browser / curl / requests
```

---

## Siguiente clase

Hoy construimos manualmente:

```text
Servidor HTTP
Routes
JSON responses
404
Cliente
```

La siguiente clase reemplazará gran parte de esto con:

```python
from fastapi import FastAPI
```

Pasaremos de:

```python
if urlsplit(self.path).path == "/products":
    ...
```

a:

```python
@app.get("/products")
def get_products():
    return productos
```

Y de:

```python
json.dumps(...).encode("utf-8")
```

a:

```python
return productos
```

Ese será nuestro punto de entrada a **FastAPI**.

---

[Índice del curso](README.md)
