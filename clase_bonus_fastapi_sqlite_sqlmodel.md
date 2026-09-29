# Clase Bonus — FastAPI + SQLite con SQLModel

**Duración:** 30–45 minutos  
**Objetivo:** agregar persistencia real a nuestra API de FastAPI usando SQLite sin escribir SQL directamente.

---

# 1. El problema

Hasta ahora:

```python
productos = []
```

Los datos viven en memoria. Si detenemos el servidor, desaparecen.

Queremos:

```text
FastAPI
   ↓
SQLModel
   ↓
SQLite
   ↓
products.db
```

SQLite guarda la base de datos en un archivo. SQLModel nos permite trabajar con ella usando objetos Python.

---

# 2. ¿Qué es SQLModel?

SQLModel permite definir modelos Python que también representan datos de una base de datos.

Por debajo:

```text
SQLModel
   ↓
SQLAlchemy
   ↓
SQLite
```

El SQL sigue existiendo, pero SQLModel/SQLAlchemy lo generan por nosotros.

Instala:

```bash
pip install sqlmodel
```

---

# 3. Crear el modelo

```python
from sqlmodel import SQLModel, Field


class Product(SQLModel, table=True):
    id: int | None = Field(
        default=None,
        primary_key=True
    )
    nombre: str
    precio: float
    stock: int
```

`table=True` indica que el modelo representa una tabla.

```text
Product                  SQLite

id        ─────────────→ id
nombre    ─────────────→ nombre
precio    ─────────────→ precio
stock     ─────────────→ stock
```

`primary_key=True` indica que `id` identifica de forma única cada registro.

SQLite generará el ID cuando guardemos un producto.

---

# 4. Crear el engine

```python
from sqlmodel import create_engine


engine = create_engine(
    "sqlite:///products.db"
)
```

El engine representa nuestra configuración de acceso a la base de datos.

```text
FastAPI
   ↓
engine
   ↓
products.db
```

---

# 5. Crear las tablas

```python
SQLModel.metadata.create_all(engine)
```

Código:

```python
engine = create_engine(
    "sqlite:///products.db"
)

SQLModel.metadata.create_all(engine)
```

Al ejecutar la aplicación aparecerá:

```text
products.db
```

---

# 6. Session

Para trabajar con los datos usamos una sesión:

```python
from sqlmodel import Session
```

```python
with Session(engine) as session:
    ...
```

Una `Session` nos permite:

```text
guardar
buscar
modificar
eliminar
```

datos.

---

# 7. POST — guardar productos

Antes:

```python
productos.append(product)
```

Ahora:

```python
@app.post("/products")
def create_product(product: Product):

    with Session(engine) as session:
        session.add(product)
        session.commit()
        session.refresh(product)

        return product
```

Las operaciones:

```python
session.add(product)
```

prepara el objeto para guardarlo.

```python
session.commit()
```

confirma los cambios.

```python
session.refresh(product)
```

actualiza el objeto con valores generados por la base, como el `id`.

---

# 8. Probar POST

Ejecuta:

```bash
fastapi dev main.py
```

Abre:

```text
http://localhost:8000/docs
```

Envía:

```json
{
    "nombre": "Laptop",
    "precio": 15000,
    "stock": 5
}
```

Después:

```json
{
    "nombre": "Mouse",
    "precio": 450,
    "stock": 10
}
```

Ahora detén el servidor:

```text
Ctrl + C
```

y vuelve a iniciarlo.

Los datos permanecen almacenados en:

```text
products.db
```

---

# 9. GET — obtener todos

Importa:

```python
from sqlmodel import select
```

Endpoint:

```python
@app.get("/products")
def get_products():

    with Session(engine) as session:
        return session.exec(
            select(Product)
        ).all()
```

Antes:

```python
return productos
```

Ahora:

```python
session.exec(
    select(Product)
).all()
```

Los datos ya no vienen de una lista. Vienen de SQLite.

---

# 10. GET por ID

```python
from fastapi import HTTPException
```

```python
@app.get("/products/{product_id}")
def get_product(product_id: int):

    with Session(engine) as session:

        product = session.get(
            Product,
            product_id
        )

        if product is None:
            raise HTTPException(
                status_code=404,
                detail="Producto no encontrado"
            )

        return product
```

`session.get()` busca utilizando la primary key.

---

# Actividad

Desde `/docs`:

1. crea tres productos;
2. consulta `/products`;
3. consulta `/products/1`;
4. consulta `/products/999`;
5. reinicia el servidor;
6. comprueba que los productos siguen existiendo.

---

# 11. DELETE

Antes:

```python
productos.remove(product)
```

Ahora:

```python
@app.delete("/products/{product_id}")
def delete_product(product_id: int):

    with Session(engine) as session:

        product = session.get(
            Product,
            product_id
        )

        if product is None:
            raise HTTPException(
                status_code=404,
                detail="Producto no encontrado"
            )

        session.delete(product)
        session.commit()

        return {
            "mensaje": "Producto eliminado"
        }
```

---

# 12. CRUD con SQLModel

## CREATE

```python
session.add(product)
session.commit()
```

## READ — todos

```python
session.exec(
    select(Product)
).all()
```

## READ — por ID

```python
session.get(
    Product,
    product_id
)
```

## DELETE

```python
session.delete(product)
session.commit()
```

UPDATE puede agregarse después.

---

# 13. Código completo

```python
from fastapi import FastAPI, HTTPException

from sqlmodel import (
    SQLModel,
    Field,
    Session,
    create_engine,
    select
)


app = FastAPI()


class Product(SQLModel, table=True):
    id: int | None = Field(
        default=None,
        primary_key=True
    )
    nombre: str
    precio: float
    stock: int


engine = create_engine(
    "sqlite:///products.db"
)

SQLModel.metadata.create_all(engine)


@app.get("/")
def home():
    return {
        "mensaje": "Products API"
    }


@app.get("/products")
def get_products():

    with Session(engine) as session:
        return session.exec(
            select(Product)
        ).all()


@app.get("/products/{product_id}")
def get_product(product_id: int):

    with Session(engine) as session:

        product = session.get(
            Product,
            product_id
        )

        if product is None:
            raise HTTPException(
                status_code=404,
                detail="Producto no encontrado"
            )

        return product


@app.post("/products")
def create_product(product: Product):

    with Session(engine) as session:
        session.add(product)
        session.commit()
        session.refresh(product)

        return product


@app.delete("/products/{product_id}")
def delete_product(product_id: int):

    with Session(engine) as session:

        product = session.get(
            Product,
            product_id
        )

        if product is None:
            raise HTTPException(
                status_code=404,
                detail="Producto no encontrado"
            )

        session.delete(product)
        session.commit()

        return {
            "mensaje": "Producto eliminado"
        }
```

---

# 14. ¿Dónde está el SQL?

Nosotros escribimos:

```python
select(Product)
```

Por debajo se genera SQL equivalente conceptualmente a:

```sql
SELECT * FROM product;
```

Cuando hacemos:

```python
session.add(product)
```

por debajo terminará ejecutándose un `INSERT`.

El SQL no desapareció.

```text
Nuestro Python
     ↓
SQLModel
     ↓
SQLAlchemy
     ↓
SQL
     ↓
SQLite
```

---

# 15. ¿Qué es un ORM?

ORM significa:

```text
Object Relational Mapping
```

La idea:

```text
Objeto Python
      ↕
Registro de base de datos
```

Por ejemplo:

```python
product.nombre
```

corresponde a una columna `nombre`.

Nuestro modelo:

```python
Product
```

representa una tabla.

---

# 16. Evolución de nuestra API

Primero:

```text
HTTPServer
   ↓
JSON manual
   ↓
list[dict]
```

Después:

```text
FastAPI
   ↓
Pydantic
   ↓
list
```

Ahora:

```text
FastAPI
   ↓
SQLModel
   ↓
SQLAlchemy
   ↓
SQLite
```

Una aplicación más grande podría utilizar:

```text
FastAPI
   ↓
Pydantic / SQLAlchemy
   ↓
PostgreSQL
```

Pero ya estamos aprendiendo los conceptos fundamentales:

```text
modelo
primary key
session
query
persistencia
```

---

# Ejercicio final

Construye una API persistente con:

```text
GET       /products
GET       /products/{id}
POST      /products
DELETE    /products/{id}
```

Modelo:

```python
class Product(SQLModel, table=True):
    id: int | None = Field(
        default=None,
        primary_key=True
    )
    nombre: str
    precio: float
    stock: int
```

Requisitos:

1. No utilizar una lista global.
2. POST guarda en SQLite.
3. GET lee desde SQLite.
4. GET por ID devuelve 404 si no existe.
5. DELETE elimina de SQLite.
6. Los datos deben seguir existiendo después de reiniciar el servidor.

Prueba todo desde:

```text
http://localhost:8000/docs
```

---

# Cheat Sheet

## Modelo

```python
class Product(SQLModel, table=True):
    id: int | None = Field(
        default=None,
        primary_key=True
    )
    nombre: str
    precio: float
    stock: int
```

## Engine

```python
engine = create_engine(
    "sqlite:///products.db"
)
```

## Crear tablas

```python
SQLModel.metadata.create_all(engine)
```

## Session

```python
with Session(engine) as session:
    ...
```

## Guardar

```python
session.add(product)
session.commit()
session.refresh(product)
```

## Obtener todos

```python
session.exec(
    select(Product)
).all()
```

## Obtener por ID

```python
session.get(
    Product,
    product_id
)
```

## Eliminar

```python
session.delete(product)
session.commit()
```

---

# Idea principal

Antes:

```python
productos.append(product)
```

Ahora:

```python
session.add(product)
session.commit()
```

Antes:

```python
return productos
```

Ahora:

```python
session.exec(
    select(Product)
).all()
```

La diferencia más importante:

```text
ANTES

detener servidor
      ↓
datos desaparecen
```

```text
AHORA

detener servidor
      ↓
products.db
      ↓
datos permanecen
```

Nuestra API ahora tiene **persistencia real**.
