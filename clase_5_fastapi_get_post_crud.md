# Clase 5 — FastAPI: de servidor manual a API real

**Duración:** 2 horas  
**Objetivo:** reconstruir con FastAPI la API de la clase anterior y agregar GET, parámetros, POST, validación, errores y DELETE.

---

# 1. De servidor manual a FastAPI

En la clase anterior hicimos manualmente:

```text
Request → HTTPServer → do_GET() → self.path → JSON → Response
```

Ahora:

```python
@app.get("/products")
def get_products():
    return productos
```

```text
MANUAL                    FASTAPI
do_GET()                  @app.get()
self.path                 "/products"
json.dumps()              automático
.encode()                 automático
send_response()           automático
Content-Type              automático
```

> FastAPI no reemplaza HTTP. Automatiza gran parte del trabajo HTTP que hicimos manualmente.

---

# 2. Crear el proyecto

```text
products_api/
├── main.py
└── client.py
```

Instala:

```bash
pip install "fastapi[standard]"
```

---

# 3. Primera API

`main.py`:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def home():
    return {"mensaje": "Hola desde FastAPI"}
```

Ejecuta:

```bash
fastapi dev main.py
```

Abre:

```text
http://localhost:8000
```

FastAPI convierte automáticamente los datos de Python a JSON.

---

# 4. Documentación automática

Abre:

```text
http://localhost:8000/docs
```

Desde aquí podemos:

- ver endpoints;
- enviar requests;
- probar parámetros;
- enviar JSON;
- revisar responses y status codes.

---

# 5. Productos

```python
productos = [
    {"id": 1, "nombre": "Laptop", "precio": 15000, "stock": 5},
    {"id": 2, "nombre": "Mouse", "precio": 450, "stock": 12},
    {"id": 3, "nombre": "Monitor", "precio": 4200, "stock": 3}
]
```

## GET `/products`

```python
@app.get("/products")
def get_products():
    return productos
```

Prueba desde:

```text
http://localhost:8000/products
```

También:

```bash
curl http://localhost:8000/products
```

---

# 6. Path Parameters

Queremos:

```text
GET /products/1
GET /products/2
```

```python
@app.get("/products/{product_id}")
def get_product(product_id: int):

    for producto in productos:
        if producto["id"] == product_id:
            return producto

    return {"error": "Producto no encontrado"}
```

Prueba:

```text
/products/1
/products/abc
```

FastAPI utiliza:

```python
product_id: int
```

para convertir y validar el parámetro.

---

# 7. Query Parameters

Queremos:

```text
GET /products?min_price=1000
```

```python
@app.get("/products")
def get_products(min_price: float | None = None):

    if min_price is None:
        return productos

    return [
        producto
        for producto in productos
        if producto["precio"] >= min_price
    ]
```

Prueba:

```text
/products
/products?min_price=1000
/products?min_price=5000
```

---

# Actividad 1 — Filtros

Agrega:

```text
in_stock
```

Debe soportar:

```text
/products
/products?min_price=1000
/products?in_stock=true
/products?min_price=1000&in_stock=true
```

Cuando:

```text
in_stock=true
```

solo devuelve productos con:

```python
stock > 0
```

---

# 8. POST

Hasta ahora:

```text
GET
cliente ← datos ← servidor
```

Ahora:

```text
POST
cliente → datos → servidor
```

Queremos enviar:

```json
{
    "nombre": "Webcam",
    "precio": 800,
    "stock": 6
}
```

---

# 9. Modelo con Pydantic

```python
from pydantic import BaseModel


class Product(BaseModel):
    nombre: str
    precio: float
    stock: int
```

Esto describe los datos que esperamos:

```text
nombre    str
precio    float
stock     int
```

---

# 10. POST `/products`

```python
@app.post("/products")
def create_product(product: Product):

    nuevo_producto = {
        "id": len(productos) + 1,
        "nombre": product.nombre,
        "precio": product.precio,
        "stock": product.stock
    }

    productos.append(nuevo_producto)

    return nuevo_producto
```

Abre `/docs` y envía:

```json
{
    "nombre": "Webcam",
    "precio": 800,
    "stock": 6
}
```

Después ejecuta:

```text
GET /products
```

---

# 11. Validación automática

Prueba:

```json
{
    "nombre": "Webcam",
    "precio": "abc",
    "stock": 6
}
```

FastAPI/Pydantic detectará el error automáticamente.

Antes tendríamos que hacer validaciones manuales. Ahora:

```python
precio: float
```

ya describe el tipo esperado.

---

# 12. Campos opcionales

```python
class Product(BaseModel):
    nombre: str
    precio: float
    stock: int
    descripcion: str | None = None
```

```text
nombre         obligatorio
precio         obligatorio
stock          obligatorio
descripcion    opcional
```

---

# 13. HTTPException y 404

Importa:

```python
from fastapi import FastAPI, HTTPException
```

Actualiza:

```python
@app.get("/products/{product_id}")
def get_product(product_id: int):

    for producto in productos:
        if producto["id"] == product_id:
            return producto

    raise HTTPException(
        status_code=404,
        detail="Producto no encontrado"
    )
```

Prueba:

```text
GET /products/999
```

Ahora devuelve un verdadero:

```text
404 Not Found
```

---

# 14. Consumir FastAPI con `requests`

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
    print(producto["nombre"], producto["precio"])
```

El cliente no necesita saber cómo está implementado el servidor.

Solo conoce HTTP.

---

# 15. POST desde `requests`

```python
import requests


producto = {
    "nombre": "Teclado",
    "precio": 900,
    "stock": 8
}

response = requests.post(
    "http://localhost:8000/products",
    json=producto,
    timeout=10
)

response.raise_for_status()

print(response.json())
```

Flujo:

```text
client.py
   │
   │ POST + JSON
   ▼
FastAPI
   │
   ▼
Pydantic
   │
   ▼
validación
   │
   ▼
productos.append()
   │
   ▼
JSON Response
```

---

# Actividad 2 — Cliente POST

Crea un programa que solicite:

```text
Nombre:
Precio:
Stock:
```

Ejemplo:

```text
Nombre: Bocinas
Precio: 1300
Stock: 4
```

Después debe hacer:

```text
POST /products
```

y mostrar el producto creado.

---

# 16. DELETE

Ya tenemos:

```text
GET     leer
POST    crear
```

Agregamos:

```text
DELETE  eliminar
```

```python
@app.delete("/products/{product_id}")
def delete_product(product_id: int):

    for producto in productos:

        if producto["id"] == product_id:
            productos.remove(producto)

            return {
                "mensaje": "Producto eliminado"
            }

    raise HTTPException(
        status_code=404,
        detail="Producto no encontrado"
    )
```

Prueba desde `/docs`:

```text
DELETE /products/2
```

Después:

```text
GET /products
```

---

# 17. CRUD

```text
CREATE    POST
READ      GET
UPDATE    PUT / PATCH
DELETE    DELETE
```

Ya tenemos:

```text
POST      CREATE
GET       READ
DELETE    DELETE
```

---

# 18. Bonus — PUT

Si sobra tiempo:

```python
@app.put("/products/{product_id}")
def update_product(product_id: int, product: Product):

    for i, actual in enumerate(productos):

        if actual["id"] == product_id:

            actualizado = {
                "id": product_id,
                "nombre": product.nombre,
                "precio": product.precio,
                "stock": product.stock,
                "descripcion": product.descripcion
            }

            productos[i] = actualizado

            return actualizado

    raise HTTPException(
        status_code=404,
        detail="Producto no encontrado"
    )
```

Ya tenemos CRUD completo:

```text
CREATE    POST
READ      GET
UPDATE    PUT
DELETE    DELETE
```

---

# Ejercicio final — Products API

Crea:

```text
products_api/
├── main.py
└── client.py
```

Endpoints:

```text
GET      /
GET      /health
GET      /products
GET      /products/{id}
POST     /products
DELETE   /products/{id}
```

Bonus:

```text
PUT      /products/{id}
```

## Modelo

```python
class Product(BaseModel):
    nombre: str
    precio: float
    stock: int
    descripcion: str | None = None
```

## `/`

```json
{
    "mensaje": "Products API"
}
```

## `/health`

```json
{
    "status": "ok"
}
```

## `/products`

Debe devolver todos los productos y permitir:

```text
/products?min_price=1000
```

Bonus:

```text
/products?in_stock=true
```

## `/products/{id}`

Devuelve el producto o:

```text
404 Not Found
```

## POST `/products`

Ejemplo:

```json
{
    "nombre": "Webcam",
    "precio": 800,
    "stock": 6,
    "descripcion": "Webcam Full HD"
}
```

## DELETE `/products/{id}`

Elimina el producto o devuelve 404.

---

# Cliente final

`client.py` debe:

1. consultar `/health`;
2. consultar `/products`;
3. mostrar los productos;
4. crear un producto con POST;
5. volver a consultar `/products`.

Usa:

```python
timeout=10
```

```python
response.raise_for_status()
```

y maneja:

```python
requests.RequestException
```

---

# Cheat Sheet

## Crear app

```python
from fastapi import FastAPI

app = FastAPI()
```

## Ejecutar

```bash
fastapi dev main.py
```

## Docs

```text
http://localhost:8000/docs
```

## GET

```python
@app.get("/products")
def get_products():
    return productos
```

## Path parameter

```python
@app.get("/products/{product_id}")
def get_product(product_id: int):
    ...
```

## Query parameter

```python
@app.get("/products")
def get_products(min_price: float | None = None):
    ...
```

## Modelo

```python
from pydantic import BaseModel

class Product(BaseModel):
    nombre: str
    precio: float
    stock: int
```

## POST

```python
@app.post("/products")
def create_product(product: Product):
    ...
```

## DELETE

```python
@app.delete("/products/{product_id}")
def delete_product(product_id: int):
    ...
```

## 404

```python
raise HTTPException(
    status_code=404,
    detail="Producto no encontrado"
)
```

## GET con requests

```python
requests.get(
    "http://localhost:8000/products"
)
```

## POST con requests

```python
requests.post(
    "http://localhost:8000/products",
    json=producto
)
```

---

# Comparación final

```text
SERVIDOR MANUAL                 FASTAPI

HTTPServer                      FastAPI()
do_GET()                        @app.get()
do_POST()                       @app.post()
self.path                       rutas
json.loads()                    Pydantic
json.dumps()                    automático
.encode()                       automático
send_response()                 automático
headers                         automático
validación manual               type hints + Pydantic
404 manual                      HTTPException
sin documentación               /docs
```

---

# Idea principal

Clase anterior:

```text
HTTP → ruta → Python → JSON → response
```

Ahora:

```text
cliente
   ↓
HTTP
   ↓
FastAPI
   ↓
endpoint
   ↓
Pydantic / validación
   ↓
lógica Python
   ↓
JSON response
```

FastAPI se encarga del trabajo repetitivo y nosotros nos concentramos en:

```text
rutas
datos
validación
lógica de la aplicación
```

Con GET, POST y DELETE ya tenemos las bases para construir una API real.
