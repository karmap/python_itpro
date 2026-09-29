# Clase 12: Persistencia con SQLite y SQLModel

**Duración:** 2 horas

**Objetivo:** conservar los productos de nuestra API después de reiniciar, usando SQLite y SQLModel sin escribir SQL directamente. Esta clase es obligatoria antes del proyecto final.

## 1. De datos en memoria a persistencia

En la clase 11, reiniciar FastAPI restablecía la lista y el contador. Ahora los datos se guardan en un archivo:

```text
Cliente / requests
        ↓
FastAPI
        ↓
Modelo de entrada / validación
        ↓
SQLModel / Session
        ↓
SQLAlchemy
        ↓
SQLite / products.db
```

| Componente | Responsabilidad |
|---|---|
| SQLModel | Modelos de datos y trabajo con objetos persistentes. Integra Pydantic y SQLAlchemy. |
| Pydantic | Valida los datos de entrada y salida. |
| SQLAlchemy | Ejecuta consultas y maneja el acceso a la base de datos. |
| SQLite | Motor de base de datos que guarda los registros en un archivo. |

Un ORM relaciona objetos Python con registros de una tabla. SQL sigue existiendo por debajo; no necesitamos aprender su sintaxis para esta actividad.

Una **tabla** agrupa registros del mismo tipo: cada fila es un producto y cada columna es un campo, como `nombre` o `precio`. Una clave primaria identifica una fila de forma única. El modelo describe las columnas; sus instancias representan los productos concretos.

## 2. Preparar el proyecto

Reutiliza `products_api/`, su entorno y el cliente de la clase 11. Instala:

```bash
python -m pip install sqlmodel
```

Si creas un entorno nuevo, instala también `"fastapi[standard]"` y `requests`, o usa el `requirements.txt` de la raíz del curso. La versión comprobada es SQLModel 0.0.47 con Pydantic 2.13.5 y SQLAlchemy 2.0.54.

Conservaremos `/`, `/health`, filtros y `descripcion`. PUT sigue siendo opcional. La base inicia vacía: los productos de la antigua lista no se importan automáticamente. Créalos con POST para practicar la persistencia.

## 3. Separar entrada, tabla y respuesta

Usaremos tres clases pequeñas con responsabilidades diferentes:

```python
from pydantic import ConfigDict
from sqlmodel import Field, SQLModel


class ProductInput(SQLModel):
    model_config = ConfigDict(
        str_strip_whitespace=True, extra="forbid", allow_inf_nan=False
    )
    nombre: str = Field(min_length=1)
    precio: float = Field(ge=0)
    stock: int = Field(ge=0)
    descripcion: str | None = None


class Product(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    nombre: str
    precio: float
    stock: int
    descripcion: str | None = None


class ProductPublic(SQLModel):
    id: int
    nombre: str
    precio: float
    stock: int
    descripcion: str | None = None
```

`ProductInput` y `ProductPublic` no tienen `table=True`: son modelos de datos basados en Pydantic. Solo `Product` representa una tabla.

Aquí importamos `Field` desde `sqlmodel`: permite declarar reglas de datos y opciones de tabla como `primary_key`. `ConfigDict(allow_inf_nan=False)` aplica a todos los campos numéricos del modelo de entrada la restricción de valores finitos que antes pusimos en el precio.

- La entrada conserva las reglas de la clase 11 y rechaza campos extra, incluido `id`.
- La tabla usa `primary_key=True` para identificar cada registro. Antes de guardarlo, su ID puede ser `None`; SQLite genera el valor al insertar.
- La respuesta exige un ID entero porque el registro ya fue guardado.

No recibas el modelo de tabla directamente como body: separar la entrada garantiza errores 422 antes de llegar a la base de datos y evita que el cliente elija un ID. `Product.model_validate(entrada)` construirá el objeto persistente.

SQLite garantiza IDs únicos para registros existentes. Su generación automática no garantiza que un ID de un registro eliminado nunca se reutilice; no la confundas con el contador creciente de la clase anterior.

## 4. Engine, archivo y creación de tablas

Después de declarar los modelos, añade:

```python
from pathlib import Path
from sqlmodel import create_engine


db_path = Path(__file__).resolve().with_name("products.db")
engine = create_engine(f"sqlite:///{db_path}")
SQLModel.metadata.create_all(engine)
```

El engine configura el acceso. `__file__` es la ruta del módulo actual; `resolve()` la vuelve absoluta y `with_name()` cambia su nombre. Así `products.db` queda junto a `main.py`, aunque la terminal esté en otra carpeta.

`sqlite:///` identifica el tipo de base y va seguido de la ruta del archivo; no es una URL HTTP. `SQLModel.metadata` reúne las definiciones de las clases con `table=True`; por eso se declaran antes de llamar a `create_all(engine)`.

`create_all()` crea las tablas que aún no existen; conserva los registros al reiniciar. Para esta demostración se ejecuta al cargar el módulo, después de definir las tablas. No actualiza columnas de una tabla existente. Si cambias el esquema, prueba con una base nueva para el ejercicio o conserva la anterior y usa otro nombre de archivo; no borres una base con datos que quieras guardar.

## 5. Session y transacciones

```python
from sqlmodel import Session

with Session(engine) as session:
    ...
```

Una sesión agrupa nuestro trabajo con la base. `with` la cierra al terminar, también si ocurre una excepción. Para confirmar escrituras necesitamos `commit()`; salir del `with` por sí solo no las confirma.

Una **transacción** agrupa cambios que se confirman juntos con `commit()` o se descartan con `rollback()`. `add()` y `delete()` preparan el trabajo; el cierre no sustituye la confirmación. Esta sesión de base de datos no es una sesión de usuario ni una cookie HTTP.

| Operación | Efecto |
|---|---|
| `session.add(objeto)` | Prepara un registro nuevo o modificado. |
| `session.commit()` | Confirma la transacción. |
| `session.refresh(objeto)` | Recarga valores guardados, como el ID. |
| `session.get(Product, id)` | Busca por clave primaria; devuelve objeto o `None`. |
| `session.delete(objeto)` | Prepara la eliminación, que se confirma con `commit()`. |

Si falla una operación, no anuncies éxito. En estos endpoints dejamos propagar los errores inesperados y cerramos la sesión; el proyecto final mostrará un mensaje comprensible en el cliente. Si quisieras continuar trabajando con la misma sesión tras un error de escritura, tendrías que usar `session.rollback()` antes de reutilizarla.

## 6. POST: de entrada validada a registro

En el archivo de trabajo debe existir `app = FastAPI()` e importarse `FastAPI`; el código completo aparece más abajo. Reemplaza la ruta POST anterior:

```python
@app.post("/products", response_model=ProductPublic, status_code=201)
def create_product(product: ProductInput):
    db_product = Product.model_validate(product)
    with Session(engine) as session:
        session.add(db_product)
        session.commit()
        session.refresh(db_product)
        return db_product
```

`response_model` describe y valida la respuesta; no crea tablas. `refresh()` obtiene el ID generado antes de enviar el registro al cliente.

Envía desde `/docs`:

```json
{
    "nombre": "Laptop",
    "precio": 15000,
    "stock": 5,
    "descripcion": "Equipo de trabajo"
}
```

Prueba `{}`, precio `"abc"`, valores negativos y un `id` enviado en el body. Todos deben devolver 422 y no guardar registros.

## 7. GET con filtros

Importa `select` desde `sqlmodel` y reemplaza GET `/products`:

```python
@app.get("/products", response_model=list[ProductPublic])
def get_products(min_price: float | None = None, in_stock: bool = False):
    statement = select(Product)
    if min_price is not None:
        statement = statement.where(Product.precio >= min_price)
    if in_stock:
        statement = statement.where(Product.stock > 0)
    with Session(engine) as session:
        return session.exec(statement).all()
```

`select(Product)` prepara una consulta de registros. `where()` añade condiciones; las expresiones sobre los campos del modelo se convierten en condiciones de consulta. `session.exec()` la ejecuta y `.all()` obtiene los resultados antes de cerrar la sesión.

Prueba los mismos filtros de la clase 11. Ya no recorremos una lista global: la base de datos filtra los productos. No asumiremos un orden específico sin indicarlo; si necesitas ordenar, usa `statement.order_by(Product.id)`.

## 8. GET por ID y DELETE

Actualiza el import a `from fastapi import FastAPI, HTTPException`.

```python
@app.get("/products/{product_id}", response_model=ProductPublic)
def get_product(product_id: int):
    with Session(engine) as session:
        product = session.get(Product, product_id)
        if product is None:
            raise HTTPException(status_code=404, detail="Producto no encontrado")
        return product


@app.delete("/products/{product_id}")
def delete_product(product_id: int):
    with Session(engine) as session:
        product = session.get(Product, product_id)
        if product is None:
            raise HTTPException(status_code=404, detail="Producto no encontrado")
        session.delete(product)
        session.commit()
        return {"mensaje": "Producto eliminado"}
```

Conservamos la semántica anterior: GET ausente y DELETE ausente devuelven 404; un DELETE exitoso devuelve 200 con un mensaje.

## 9. Código completo de `main.py`

Reemplaza el archivo anterior con esta versión. No conserves la lista global, el contador ni rutas duplicadas.

```python
from pathlib import Path

from fastapi import FastAPI, HTTPException
from pydantic import ConfigDict
from sqlmodel import Field, Session, SQLModel, create_engine, select


class ProductInput(SQLModel):
    model_config = ConfigDict(
        str_strip_whitespace=True, extra="forbid", allow_inf_nan=False
    )
    nombre: str = Field(min_length=1)
    precio: float = Field(ge=0)
    stock: int = Field(ge=0)
    descripcion: str | None = None


class Product(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    nombre: str
    precio: float
    stock: int
    descripcion: str | None = None


class ProductPublic(SQLModel):
    id: int
    nombre: str
    precio: float
    stock: int
    descripcion: str | None = None


db_path = Path(__file__).resolve().with_name("products.db")
engine = create_engine(f"sqlite:///{db_path}")
SQLModel.metadata.create_all(engine)
app = FastAPI()


@app.get("/")
def home():
    return {"mensaje": "Products API"}


@app.get("/health")
def health():
    return {"status": "ok"}


@app.get("/products", response_model=list[ProductPublic])
def get_products(min_price: float | None = None, in_stock: bool = False):
    statement = select(Product)
    if min_price is not None:
        statement = statement.where(Product.precio >= min_price)
    if in_stock:
        statement = statement.where(Product.stock > 0)
    with Session(engine) as session:
        return session.exec(statement).all()


@app.get("/products/{product_id}", response_model=ProductPublic)
def get_product(product_id: int):
    with Session(engine) as session:
        product = session.get(Product, product_id)
        if product is None:
            raise HTTPException(status_code=404, detail="Producto no encontrado")
        return product


@app.post("/products", response_model=ProductPublic, status_code=201)
def create_product(product: ProductInput):
    db_product = Product.model_validate(product)
    with Session(engine) as session:
        session.add(db_product)
        session.commit()
        session.refresh(db_product)
        return db_product


@app.delete("/products/{product_id}")
def delete_product(product_id: int):
    with Session(engine) as session:
        product = session.get(Product, product_id)
        if product is None:
            raise HTTPException(status_code=404, detail="Producto no encontrado")
        session.delete(product)
        session.commit()
        return {"mensaje": "Producto eliminado"}
```

Ejecuta `fastapi dev main.py` y reutiliza `client.py` de la clase 11 en otra terminal.

## Actividad: Comprobar persistencia

1. Crea tres productos, incluido uno con stock cero y otro con descripción.
2. Conserva sus IDs de las respuestas POST; no supongas que empiezan en 1 si ya hay datos.
3. Consulta todos, filtra por precio y stock, y consulta un ID real.
4. Prueba un ID ausente y comprueba 404.
5. Detén el servidor con Ctrl+C y vuelve a iniciarlo desde el mismo archivo.
6. Comprueba que registros, IDs y descripciones siguen disponibles.
7. Elimina un registro, reinicia y verifica que sigue ausente.
8. Envía un body inválido y comprueba 422, sin un producto adicional.

## 10. PUT opcional: actualización persistente

Si implementaste PUT en la clase 11, reemplázalo con esta versión:

```python
@app.put("/products/{product_id}", response_model=ProductPublic)
def update_product(product_id: int, product: ProductInput):
    with Session(engine) as session:
        db_product = session.get(Product, product_id)
        if db_product is None:
            raise HTTPException(status_code=404, detail="Producto no encontrado")
        db_product.nombre = product.nombre
        db_product.precio = product.precio
        db_product.stock = product.stock
        db_product.descripcion = product.descripcion
        session.add(db_product)
        session.commit()
        session.refresh(db_product)
        return db_product
```

PUT sustituye todos los campos modificables y conserva el ID. `descripcion` se vuelve `None` si se omite. La actualización también requiere `commit()` para sobrevivir al reinicio. Sin este bonus, la clase cubre creación, lectura y eliminación; todavía no es CRUD completo.

## 11. Guardar estructuras de IA como JSON

El proyecto final necesita persistir listas como `must_do`. Para evitar relaciones entre tablas en este curso, podemos guardar un resultado completo ya validado como texto JSON en una columna `str`.

No declares simplemente una columna `list[str]` en el modelo de tabla: necesita una estrategia de almacenamiento. Usaremos serialización, que ya conocemos.

Esta demostración es un archivo independiente, `json_storage.py`, con su propia tabla y archivo de base de datos. Ejecútalo después de activar el mismo entorno:

```python
import json
from pathlib import Path

from sqlmodel import Field, Session, SQLModel, create_engine


class SavedPlan(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    city: str
    days: int
    interest: str
    result_json: str


def main() -> None:
    db_path = Path(__file__).resolve().with_name("json_demo.db")
    engine = create_engine(f"sqlite:///{db_path}")
    SQLModel.metadata.create_all(engine)

    # Resultado fijo para practicar almacenamiento; la API real lo validará
    # con un modelo Pydantic antes de crear el registro.
    resultado = {
        "summary": "Cuatro días para explorar Tokio",
        "must_do": ["Visitar un mercado", "Probar ramen"],
        "budget_level": "medio"
    }
    registro = SavedPlan(
        city="Tokio", days=4, interest="comida",
        result_json=json.dumps(resultado, ensure_ascii=False)
    )
    with Session(engine) as session:
        session.add(registro)
        session.commit()
        session.refresh(registro)
        plan_id = registro.id
        print("ID guardado:", plan_id)

    with Session(engine) as session:
        guardado = session.get(SavedPlan, plan_id)
        if guardado is None:
            raise ValueError("No se encontró el plan guardado")
        reconstruido = json.loads(guardado.result_json)
        print(reconstruido["must_do"])


if __name__ == "__main__":
    main()
```

### Conectar generación, validación y guardado

La demostración anterior usa datos fijos. Al integrar OpenRouter, reutiliza `validar_plan()` de `models.py` (clase 11). El texto se obtiene de `response.json()["choices"][0]["message"]["content"]`, como en la clase 09. Este fragmento va después de comprobar el estado HTTP y extraer `contenido`; requiere `entrada` de tipo `TripInput`, los imports, `SavedPlan` y `engine` de tu aplicación:

```python
from fastapi import HTTPException
from models import validar_plan


try:
    resultado = validar_plan(contenido)
except ValueError as error:
    # ValidationError de Pydantic también deriva de ValueError.
    raise HTTPException(
        status_code=502, detail="La IA devolvió un plan inválido"
    ) from error

registro = SavedPlan(
    city=entrada.city,
    days=entrada.days,
    interest=entrada.interest,
    result_json=json.dumps(resultado.model_dump(), ensure_ascii=False)
)
with Session(engine) as session:
    session.add(registro)
    session.commit()
    session.refresh(registro)
```

Si la validación falla, `raise` termina este flujo antes de crear el registro. `raise ... from error` conserva la causa original para depurar. El JSON guardado proviene del diccionario validado, no directamente del texto del modelo. Captura `requests.Timeout` alrededor de la petición para responder 504 y `requests.RequestException` para responder 502; una respuesta sin `choices` o sin contenido también debe terminar con 502 antes de guardar.

En `/history/{id}`, decodifica `result_json` y devuelve el resultado como objeto JSON; no expongas una cadena con JSON escapado como si fuera el objeto original. La columna guarda consulta y resultado juntos mediante los campos del registro. El listado del historial puede mostrar solo ID, ciudad, días e interés.

La API key sigue únicamente en el entorno del servidor. No la guardes en ninguna tabla, respuesta o archivo del proyecto. Valida la salida de OpenRouter antes de abrir la sesión para guardar y evita mantener una transacción abierta mientras esperas la respuesta de IA.

## Ejercicio final: Products API persistente

Usa los endpoints de esta clase, conserva las funciones del cliente y verifica filtros, descripción, validación, eliminación y reinicio. PUT es opcional; las operaciones de historial del proyecto final solo requieren GET, POST y DELETE.

## Cheat sheet

```python
# Fragmentos de consulta: requieren los modelos y engine anteriores.
with Session(engine) as session:
    product = Product.model_validate({"nombre": "Mouse", "precio": 450, "stock": 3})
    session.add(product)
    session.commit()
    session.refresh(product)
    product_id = product.id

with Session(engine) as session:
    product = session.get(Product, product_id)
    products = session.exec(select(Product)).all()
    if product is not None:
        session.delete(product)
        session.commit()
```

Siguiente paso: AI Trip Planner, integrando la consola, FastAPI, validación de entrada y salida de IA, y un historial persistente. Su documento se incorporará al índice cuando esté disponible.

---

[Índice del curso](README.md)
