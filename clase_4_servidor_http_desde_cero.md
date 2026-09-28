# Clase 4 — Construyendo un servidor HTTP desde cero

**Duración:** 2 horas  
**Objetivo:** entender cómo funciona un servidor HTTP construyendo uno simple con Python, consumirlo desde otro programa, probarlo con `curl` y abrirlo desde el navegador.

---

# 1. ¿Qué vamos a construir?

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

# 2. Primer servidor HTTP

Python incluye herramientas para crear un servidor HTTP básico sin instalar librerías externas.

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
```

Crea:

```text
server.py
```

```python
from http.server import BaseHTTPRequestHandler, HTTPServer


class Handler(BaseHTTPRequestHandler):

    def do_GET(self):
        self.send_response(200)
        self.end_headers()

        self.wfile.write(
            b"Hola desde Python"
        )


server = HTTPServer(
    ("localhost", 8000),
    Handler
)

print("Servidor corriendo en http://localhost:8000")

server.serve_forever()
```

Ejecuta:

```bash
python server.py
```

El programa se queda ejecutándose porque está esperando requests.

---

# 3. `localhost` y puerto

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

# 4. Probar desde el navegador

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

# 5. Status code

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

# 6. Headers y JSON

Modifica el servidor:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer


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


server = HTTPServer(
    ("localhost", 8000),
    Handler
)

print("Servidor corriendo en http://localhost:8000")

server.serve_forever()
```

Ahora enviamos:

```text
Content-Type: application/json
```

para indicar que el body es JSON.

---

# 7. ¿Por qué `b"..."`?

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

# 8. Probar con `curl`

En otra terminal:

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

# 9. Consumir desde otro programa Python

Crea:

```text
client.py
```

```python
import requests


response = requests.get(
    "http://localhost:8000"
)

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

# 10. Convertir JSON a Python

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

# Actividad 1 — Cambiar la respuesta

Modifica el servidor para devolver:

```json
{
    "curso": "Python",
    "clase": 4,
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

# 11. Generar JSON con Python

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
body = json.dumps(data).encode()
```

Flujo:

```text
dict
 ↓
json.dumps()
 ↓
str
 ↓
.encode()
 ↓
bytes
 ↓
HTTP Response
```

---

# 12. Servidor usando `json.dumps()`

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import json


class Handler(BaseHTTPRequestHandler):

    def do_GET(self):

        data = {
            "mensaje": "Hola desde Python",
            "version": 1
        }

        body = json.dumps(data).encode()

        self.send_response(200)

        self.send_header(
            "Content-Type",
            "application/json"
        )

        self.end_headers()

        self.wfile.write(body)


server = HTTPServer(
    ("localhost", 8000),
    Handler
)

print("Servidor corriendo en http://localhost:8000")

server.serve_forever()
```

---

# 13. Crear varias rutas

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

Ejemplo:

```python
def do_GET(self):

    if self.path == "/":
        ...

    elif self.path == "/health":
        ...

    elif self.path == "/products":
        ...
```

---

# 14. Ruta `/health`

```python
if self.path == "/health":

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

# 15. Ruta `/products`

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

```python
elif self.path == "/products":
    data = productos
```

Luego:

```python
body = json.dumps(data).encode()
```

---

# 16. Evitar repetir código

Crea una función:

```python
def enviar_json(self, data, status=200):

    body = json.dumps(data).encode()

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
if self.path == "/":

    self.enviar_json({
        "mensaje": "Mi API"
    })

elif self.path == "/health":

    self.enviar_json({
        "status": "ok"
    })

elif self.path == "/products":

    self.enviar_json(productos)
```

---

# 17. Ruta no encontrada

```python
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

# 18. Servidor completo

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
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

        body = json.dumps(data).encode()

        self.send_response(status)

        self.send_header(
            "Content-Type",
            "application/json"
        )

        self.end_headers()

        self.wfile.write(body)


    def do_GET(self):

        if self.path == "/":

            self.enviar_json({
                "mensaje": "Mi primera API"
            })

        elif self.path == "/health":

            self.enviar_json({
                "status": "ok"
            })

        elif self.path == "/products":

            self.enviar_json(productos)

        else:

            self.enviar_json(
                {
                    "error": "Ruta no encontrada"
                },
                404
            )


server = HTTPServer(
    ("localhost", 8000),
    Handler
)

print("Servidor corriendo en http://localhost:8000")

server.serve_forever()
```

---

# Actividad 2 — Agregar rutas

Agrega:

```text
GET /info
GET /users
```

## `/info`

```json
{
    "nombre": "Mi API",
    "version": "1.0",
    "lenguaje": "Python"
}
```

## `/users`

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

# 19. Consumir `/products` desde Python

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

# 20. Navegador vs `curl` vs `requests`

Los tres son clientes HTTP.

```text
NAVEGADOR ─┐
CURL ──────┼──→ HTTP Server
REQUESTS ──┘
```

El servidor recibe HTTP sin importar qué cliente hizo el request.

---

# 21. ¿Qué está haciendo nuestro servidor?

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

# 22. ¿Qué pasa con POST?

También podríamos implementar:

```python
def do_POST(self):
```

Tendríamos que leer manualmente el body:

```python
length = int(
    self.headers["Content-Length"]
)

body = self.rfile.read(length)
```

Después:

```python
data = json.loads(body)
```

Y tendríamos que validar manualmente los datos.

Por ejemplo:

```text
¿Existe nombre?
¿Existe precio?
¿Precio es realmente un número?
```

Esto comienza a generar bastante código manual.

---

# 23. ¿Qué problema aparece?

Ya estamos manejando nosotros mismos:

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

# 24. Puente a FastAPI

Con nuestro servidor manual:

```python
class Handler(BaseHTTPRequestHandler):

    def do_GET(self):

        if self.path == "/products":
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
body = json.dumps(productos).encode()
```

podremos:

```python
return productos
```

FastAPI se encargará de:

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

# Ejercicio final — Mini Products API

Construye un servidor con:

```text
GET /
GET /health
GET /products
GET /users
GET /info
```

## `/`

```json
{
    "mensaje": "Bienvenido a mi API"
}
```

## `/health`

```json
{
    "status": "ok"
}
```

## `/products`

Debe devolver al menos 5 productos.

## `/users`

Debe devolver al menos 5 usuarios.

## `/info`

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

# Cliente del ejercicio

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

# Bonus

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

# Cheat Sheet

## Crear servidor

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
```

```python
server = HTTPServer(
    ("localhost", 8000),
    Handler
)

server.serve_forever()
```

## GET

```python
def do_GET(self):
    ...
```

## Ruta

```python
self.path
```

## Status

```python
self.send_response(200)
```

## Header

```python
self.send_header(
    "Content-Type",
    "application/json"
)
```

## Finalizar headers

```python
self.end_headers()
```

## Python → JSON

```python
json.dumps(data)
```

## String → bytes

```python
texto.encode()
```

## Escribir respuesta

```python
self.wfile.write(body)
```

## Cliente Python

```python
response = requests.get(
    "http://localhost:8000/products"
)
```

## Curl

```bash
curl http://localhost:8000/products
```

## Curl + headers

```bash
curl -i http://localhost:8000/products
```

## Navegador

```text
http://localhost:8000/products
```

---

# Flujo completo

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
       .encode()
          │
          ▼
      HTTP Response
          │
          ▼
Browser / curl / requests
```

---

# Siguiente clase

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
if self.path == "/products":
```

a:

```python
@app.get("/products")
```

Y de:

```python
json.dumps(...).encode()
```

a:

```python
return productos
```

Ese será nuestro punto de entrada a **FastAPI**.
