# Clase 11: FastAPI y operaciones sobre productos

**Duración:** 2 horas

**Objetivo:** reconstruir la API manual con FastAPI y agregar parámetros, creación, validación y eliminación de productos. PUT queda como práctica opcional.

## 1. De HTTP manual a FastAPI

En la clase 10 implementamos GET, elegimos rutas y construimos respuestas. Conservaremos el cliente y reemplazaremos el servidor:

```text
Cliente → Uvicorn → FastAPI → función Python → JSON → cliente
                        ↕
                  Pydantic / validación
```

| Componente | Responsabilidad |
|---|---|
| FastAPI | Framework que registra las rutas y coordina la API. |
| Starlette | Capa web y ASGI utilizada por FastAPI. |
| Uvicorn | Servidor ASGI que escucha HTTP y ejecuta la aplicación. |
| Pydantic | Modelos, conversión y validación de datos. |

ASGI es la interfaz entre el servidor y la aplicación. Basta conocer esa separación; no necesitamos implementar ASGI ni usar `async` para estos ejemplos.

| Servidor manual | Con FastAPI |
|---|---|
| `do_GET()` e `if` por ruta | `@app.get("/products")` |
| `json.dumps()` y `.encode()` | Devolver datos Python |
| `send_response(404)` | Lanzar `HTTPException(status_code=404, ...)` |
| Leer el body y comprobar campos | Modelo Pydantic de entrada |
| Construir documentación | `/docs` automático |

FastAPI abstrae el trabajo manual que acabamos de practicar. Nosotros seguimos eligiendo qué hace la ruta y qué código HTTP corresponde.

## 2. Preparar el proyecto

```text
products_api/
├── main.py
└── client.py
```

Reutiliza el entorno de la clase 08 o crea y activa uno nuevo. Desde `products_api/`:

```bash
python -m pip install "fastapi[standard]" requests
```

`fastapi[standard]` instala también el comando `fastapi` y Uvicorn. `requests` se usa en el cliente. El archivo `requirements.txt` del repositorio fija las versiones comprobadas para el curso completo; instala con `python -m pip install -r requirements.txt` desde la raíz si deseas reproducir ese entorno.

## 3. Primera aplicación y decoradores

Crea `main.py`:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def home():
    return {"mensaje": "Hola desde FastAPI"}
```

Un decorador es una línea con `@` que aplica comportamiento a una función. Aquí `@app.get("/")` registra `home()` para peticiones GET a `/`. FastAPI llama a esa función cuando llega la petición.

```bash
fastapi dev main.py
```

Abre `http://localhost:8000/` y `http://localhost:8000/docs`. Uvicorn ejecuta la aplicación; el modo `dev` recarga los cambios automáticamente. Detén el servidor manual de la clase 10 antes de usar el mismo puerto.

Desde `/docs` podemos probar los endpoints, sus parámetros, cuerpos y respuestas.

## 4. GET y parámetros

FastAPI decide de dónde tomar cada parámetro: un nombre que aparece en la ruta, como `{product_id}`, viene de esa ruta; un parámetro simple que no está en la ruta viene de la query; un modelo Pydantic, como `ProductInput`, se recibe en el cuerpo JSON. Sus anotaciones y valores por defecto establecen tipos y si se puede omitir.

Añade esta lista debajo de `app = FastAPI()`:

```python
productos = [
    {"id": 1, "nombre": "Laptop", "precio": 15000, "stock": 5,
     "descripcion": None},
    {"id": 2, "nombre": "Mouse", "precio": 450, "stock": 12,
     "descripcion": None},
    {"id": 3, "nombre": "Monitor", "precio": 4200, "stock": 3,
     "descripcion": None}
]
```

Una ruta GET puede recibir parámetros de query. Añade una única versión de `/products`:

```python
@app.get("/products")
def get_products(min_price: float | None = None, in_stock: bool = False):
    resultado = productos
    if min_price is not None:
        resultado = [p for p in resultado if p["precio"] >= min_price]
    if in_stock:
        resultado = [p for p in resultado if p["stock"] > 0]
    return resultado
```

Prueba estas URL en `/docs` o en el navegador:

```text
/products
/products?min_price=1000
/products?in_stock=true
/products?min_price=1000&in_stock=true
```

`None` significa que no se envió un precio mínimo. `in_stock=false`, o la ausencia de ese parámetro, conserva los productos con stock cero. Los filtros se combinan.

Si modificas una ruta existente, reemplaza su función y decorador. Registrar dos veces el mismo método y ruta no actualiza la primera versión.

### Parámetro de ruta y 404

Actualiza el import a `from fastapi import FastAPI, HTTPException` y agrega:

```python
@app.get("/products/{product_id}")
def get_product(product_id: int):
    for producto in productos:
        if producto["id"] == product_id:
            return producto
    raise HTTPException(status_code=404, detail="Producto no encontrado")
```

Prueba `/products/1`, `/products/999` y `/products/abc`. El primero devuelve 200, el segundo 404 y el tercero 422 porque `abc` no se puede interpretar como entero. Devolver `{"error": "..."}` por sí solo no cambia el código HTTP; el valor por defecto sería 200.

## Actividad 1: Filtros y errores

1. Agrega un producto con stock cero.
2. Comprueba los filtros por separado y combinados.
3. Verifica que el filtrado no cambie la lista original.
4. Prueba un ID existente, uno ausente y un ID no numérico.

## 5. Modelo de entrada y reglas de negocio

Añade estos imports y el modelo antes de las rutas POST:

```python
from pydantic import BaseModel, ConfigDict, Field


class ProductInput(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True, extra="forbid")
    nombre: str = Field(min_length=1)
    precio: float = Field(ge=0, allow_inf_nan=False)
    stock: int = Field(ge=0)
    descripcion: str | None = None
```

`class ProductInput(BaseModel)` define un modelo que hereda la validación de Pydantic. Los campos se declaran con los type hints de la clase 07.

- `Field(min_length=1)` impide un nombre vacío.
- `ge=0` significa mayor o igual que cero.
- `allow_inf_nan=False` exige un precio finito.
- `str_strip_whitespace=True` retira espacios exteriores; un nombre de solo espacios también queda inválido.
- `extra="forbid"` rechaza campos no definidos, incluido un `id` enviado por el cliente.
- `descripcion: str | None = None` acepta texto o `None` y permite omitir el campo gracias a su valor por defecto.

Pydantic puede convertir ciertos valores compatibles: un precio `"800"` puede convertirse a número; `"abc"` falla. Un type hint describe el tipo, pero no establece por sí solo reglas como precio no negativo.

También puedes crear el modelo directamente para entender qué recibe la función POST:

```python
entrada = ProductInput(nombre="Webcam", precio=800, stock=6)
print(entrada.nombre)
print(entrada.model_dump())
```

`entrada` es una instancia con atributos, como `.nombre`; `model_dump()` crea un diccionario con claves. FastAPI construye y valida esa instancia antes de llamar al endpoint. Si la validación falla, responde 422 y no ejecuta su cuerpo.

## 6. POST y generación de IDs

Añade `from itertools import count` a los imports y `ids = count(start=4)` junto a los datos iniciales.

`count()` crea un contador y `next(ids)` toma su siguiente número. Cada creación consume uno; eliminar productos no reduce el contador. No usamos `len(productos) + 1`, que podría repetir un ID después de un DELETE.

```python
@app.post("/products", status_code=201)
def create_product(product: ProductInput):
    nuevo_producto = {"id": next(ids), **product.model_dump()}
    productos.append(nuevo_producto)
    return nuevo_producto
```

`model_dump()` convierte el modelo validado en un diccionario. `**` incorpora sus campos, incluida `descripcion`. El código 201 indica que se creó un registro.

Desde `/docs`, envía:

```json
{
    "nombre": "Webcam",
    "precio": 800,
    "stock": 6,
    "descripcion": "Webcam Full HD"
}
```

Después consulta `/products`. Prueba también nombres vacíos, precios negativos, stock negativo, precio `"abc"` y un cuerpo sin `nombre`. FastAPI debe responder 422 antes de ejecutar la función de creación.

Este ejemplo mantiene datos y contador en memoria dentro de un único proceso. Al reiniciar o recargar la aplicación se restauran los datos iniciales. En la clase 12 persistiremos los productos y dejaremos los IDs a SQLite.

## 7. DELETE

```python
@app.delete("/products/{product_id}")
def delete_product(product_id: int):
    for producto in productos:
        if producto["id"] == product_id:
            productos.remove(producto)
            return {"mensaje": "Producto eliminado"}
    raise HTTPException(status_code=404, detail="Producto no encontrado")
```

Una eliminación exitosa devuelve 200 con un mensaje JSON. Si el ID ya no existe, devuelve 404. No uses 204 si vas a devolver un body: ese código significa que la respuesta no tiene contenido.

## 8. Código completo de `main.py`

Reemplaza el contenido del archivo por esta versión consolidada; no lo añadas debajo de las rutas anteriores.

```python
from itertools import count

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, ConfigDict, Field


class ProductInput(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True, extra="forbid")
    nombre: str = Field(min_length=1)
    precio: float = Field(ge=0, allow_inf_nan=False)
    stock: int = Field(ge=0)
    descripcion: str | None = None


app = FastAPI()
productos = [
    {"id": 1, "nombre": "Laptop", "precio": 15000, "stock": 5,
     "descripcion": None},
    {"id": 2, "nombre": "Mouse", "precio": 450, "stock": 12,
     "descripcion": None},
    {"id": 3, "nombre": "Monitor", "precio": 4200, "stock": 3,
     "descripcion": None}
]
ids = count(start=4)


@app.get("/")
def home():
    return {"mensaje": "Products API"}


@app.get("/health")
def health():
    return {"status": "ok"}


@app.get("/products")
def get_products(min_price: float | None = None, in_stock: bool = False):
    resultado = productos
    if min_price is not None:
        resultado = [p for p in resultado if p["precio"] >= min_price]
    if in_stock:
        resultado = [p for p in resultado if p["stock"] > 0]
    return resultado


@app.get("/products/{product_id}")
def get_product(product_id: int):
    for producto in productos:
        if producto["id"] == product_id:
            return producto
    raise HTTPException(status_code=404, detail="Producto no encontrado")


@app.post("/products", status_code=201)
def create_product(product: ProductInput):
    nuevo_producto = {"id": next(ids), **product.model_dump()}
    productos.append(nuevo_producto)
    return nuevo_producto


@app.delete("/products/{product_id}")
def delete_product(product_id: int):
    for producto in productos:
        if producto["id"] == product_id:
            productos.remove(producto)
            return {"mensaje": "Producto eliminado"}
    raise HTTPException(status_code=404, detail="Producto no encontrado")
```

## 9. Cliente con requests

Crea `client.py`. El cliente solo conoce HTTP, sin importar qué biblioteca usa el servidor:

```python
import requests

BASE_URL = "http://localhost:8000"


def main() -> None:
    try:
        response = requests.get(f"{BASE_URL}/health", timeout=10)
        response.raise_for_status()
        print("Estado:", response.json()["status"])

        response = requests.get(f"{BASE_URL}/products", timeout=10)
        response.raise_for_status()
        print("Productos:", response.json())

        nombre = input("Nombre: ").strip()
        precio = float(input("Precio: "))
        stock = int(input("Stock: "))
        descripcion = input("Descripción (opcional): ").strip() or None
        producto = {
            "nombre": nombre, "precio": precio, "stock": stock,
            "descripcion": descripcion
        }
        response = requests.post(
            f"{BASE_URL}/products", json=producto, timeout=10
        )
        response.raise_for_status()
        print("Creado:", response.json())

        response = requests.get(f"{BASE_URL}/products", timeout=10)
        response.raise_for_status()
        print("Lista actualizada:", response.json())
    except requests.exceptions.JSONDecodeError:
        print("La API no devolvió JSON válido")
    except requests.HTTPError as error:
        print("La API rechazó la petición:", error.response.status_code)
        print(error.response.text)
    except requests.RequestException as error:
        print("No fue posible conectar con la API:", error)
    except ValueError:
        print("Precio y stock deben ser números válidos")


if __name__ == "__main__":
    main()
```

Con el servidor activo en una terminal, ejecuta `python client.py` en otra. El cliente valida la conversión de números para explicar errores locales; el servidor sigue siendo responsable de validar cualquier petición.

`texto.strip() or None` devuelve el texto si queda contenido o `None` si queda vacío. `or` devuelve uno de sus operandos, no siempre un booleano; así convertimos una descripción vacía en ausencia de descripción.

## Actividad 2: Cliente POST y DELETE

1. Crea un producto y conserva el ID de la respuesta.
2. Consulta ese ID con `requests.get()`.
3. Elimínalo con `requests.delete()` y consulta de nuevo para observar 404.
4. Elimina un producto inicial que no sea el último y crea dos productos más. Comprueba que todos los IDs existentes sean distintos.
5. Reinicia el servidor y observa que los cambios en memoria desaparecen.

Usa siempre `timeout`, `raise_for_status()` y el manejo de `RequestException`.

## 10. CRUD y PUT opcional

| Operación | Método | Estado en esta clase |
|---|---|---|
| Crear | POST | Implementado |
| Leer | GET | Implementado |
| Actualizar | PUT o PATCH | PUT como bonus |
| Eliminar | DELETE | Implementado |

El núcleo cubre creación, lectura y eliminación. El CRUD completo incluye también actualización.

Añade este endpoint al `main.py` completo si sobra tiempo:

```python
@app.put("/products/{product_id}")
def update_product(product_id: int, product: ProductInput):
    for i, actual in enumerate(productos):
        if actual["id"] == product_id:
            actualizado = {"id": product_id, **product.model_dump()}
            productos[i] = actualizado
            return actualizado
    raise HTTPException(status_code=404, detail="Producto no encontrado")
```

PUT reemplaza los campos del producto conservando su ID. Como `descripcion` tiene un valor por defecto, omitirla en PUT la reemplaza por `None`. PATCH serviría para cambios parciales y queda fuera de esta clase.

## 11. Validar respuestas de IA con Pydantic

En la clase 09 comprobamos claves y tipos manualmente. Pydantic permite expresar ese contrato como un modelo. `Literal` viene de `typing` y limita un campo a valores concretos:

```python
from typing import Literal
from pydantic import BaseModel, ConfigDict, Field, StrictBool, ValidationError


class ReviewAnalysis(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True)
    sentimiento: Literal["positivo", "neutral", "negativo", "mixto"]
    categoria: Literal["producto", "envio", "soporte", "cobro", "otro"]
    urgente: StrictBool
    resumen: str = Field(min_length=1)


contenido = '''
{"sentimiento":"negativo","categoria":"cobro",
 "urgente":true,"resumen":"Cobro duplicado"}
'''
try:
    analisis = ReviewAnalysis.model_validate_json(contenido)
    print(analisis.model_dump())
except ValidationError as error:
    print("La IA devolvió datos inválidos:", error)
```

`StrictBool` exige un booleano real; no acepta la cadena `"true"`. `model_validate_json()` decodifica y valida el texto JSON, y `model_dump()` devuelve un diccionario. Para datos ya decodificados, usa `ReviewAnalysis.model_validate(datos)`.

En una API que consulte IA, una entrada inválida del usuario corresponde a 422. Si el servicio externo devuelve JSON inválido o un esquema incorrecto, podemos responder 502; si agota la espera, 504. La configuración del modelo retira espacios exteriores; el mínimo de longitud también rechaza un resumen de solo espacios.

Estas reglas preparan el proyecto final. Ejemplo de modelo de entrada que reúne lo aprendido:

```python
from typing import Literal
from pydantic import BaseModel, ConfigDict, Field


class TripInput(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True, extra="forbid")
    city: str = Field(min_length=1, max_length=100)
    days: int = Field(ge=1, le=14)
    interest: Literal["comida", "historia", "naturaleza", "vida nocturna"]
```

`le=14` pone un máximo de 14 días, una decisión sencilla de este proyecto. El resultado generado usa otro modelo. Guarda ambos modelos en `models.py` para reutilizarlos desde la API del proyecto:

```python
from typing import Literal
from pydantic import BaseModel, ConfigDict, Field


class TripResult(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True, extra="forbid")
    summary: str = Field(min_length=1)
    must_do: list[str] = Field(min_length=1)
    budget_level: Literal["bajo", "medio", "alto"]


def validar_plan(contenido: str) -> TripResult:
    resultado = TripResult.model_validate_json(contenido)
    for actividad in resultado.must_do:
        if not actividad.strip():
            raise ValueError("Cada actividad debe contener texto")
    return resultado
```

En una lista, `min_length=1` exige al menos un elemento; no comprueba que cada cadena tenga contenido. El recorrido cubre esa regla. `model_validate_json()` puede lanzar `ValidationError`; nuestra regla puede lanzar `ValueError`. El servidor debe responder 502 ante esos resultados inválidos de IA y terminar sin guardar. `502` indica una respuesta inválida del servicio externo; `504` indica que ese servicio agotó la espera.


## Ejercicio final: Products API

Construye los endpoints del código completo y un cliente que consulte `/health`, liste productos, cree uno, lo consulte y lo elimine. Conserva la descripción, valida errores y comprueba IDs después de DELETE.

PUT es opcional. La persistencia se añade en la siguiente clase.

## Cheat sheet

| Tarea | Sintaxis |
|---|---|
| Ejecutar | `fastapi dev main.py` |
| Documentación | `http://localhost:8000/docs` |
| Query opcional | `min_price: float | None = None` |
| Validar cuerpo | `product: ProductInput` |
| Crear | `@app.post("/products", status_code=201)` |
| Error | `raise HTTPException(status_code=404, detail="...")` |
| Modelo a diccionario | `product.model_dump()` |
| Texto JSON a modelo | `ReviewAnalysis.model_validate_json(contenido)` |

Siguiente paso: sustituir la lista y el contador en memoria por registros e IDs de SQLite mediante SQLModel.

---

[Índice del curso](README.md)
