# Clase 08: JSON y entornos virtuales

**Duración:** 1 a 2 horas


> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

### Objetivos

-   Entender la relación entre JSON y Python.
-   Convertir JSON a objetos de Python y viceversa.
-   Leer y guardar archivos JSON.
-   Crear un entorno virtual con `venv`.
-   Instalar dependencias con `pip`.
-   Separar persistencia de datos y lógica.

---

## 1. JSON

JSON es un formato de texto utilizado para almacenar e intercambiar
datos.

```json
{
    "nombre": "Laptop",
    "precio": 15000,
    "stock": 5,
    "disponible": true
}
```

#### JSON vs Python

| JSON | Python |
|---|---|
| object | `dict` |
| array | `list` |
| string | `str` |
| number | `int` / `float` |
| true | `True` |
| false | `False` |
| null | `None` |

```python
import json
```

## 2. JSON string → Python

Las comillas triples permiten escribir una cadena en varias líneas. El contenido sigue siendo texto hasta que `json.loads()` lo convierte. **Deserializar** es reconstruir datos a partir del texto; **serializar** es producir ese texto desde los datos de Python.

```python
texto = '''
{
    "nombre": "Laptop",
    "precio": 15000,
    "stock": 5,
    "disponible": true
}
'''

producto = json.loads(texto)

print(producto)
print(producto["nombre"])
print(type(producto))
```

`producto` ahora es un `dict`.

## 3. Python → JSON string

`indent=4` presenta el JSON con cuatro espacios de sangría; solo cambia su legibilidad. `json.dumps()` devuelve una cadena y no escribe archivos.

```python
producto = {
    "nombre": "Laptop",
    "precio": 15000,
    "stock": 5
}

texto = json.dumps(producto, indent=4)
print(texto)
```

## 4. Leer un archivo JSON

`productos.json`:

```json
[
    {"nombre": "Laptop", "precio": 15000, "stock": 5},
    {"nombre": "Mouse", "precio": 450, "stock": 12},
    {"nombre": "Monitor", "precio": 4200, "stock": 3},
    {"nombre": "Webcam", "precio": 800, "stock": 0}
]
```

Leerlo:

```python
with open("productos.json", encoding="utf-8") as archivo:
    productos = json.load(archivo)

for producto in productos:
    print(producto["nombre"])
```

## 5. Guardar un archivo JSON

`ensure_ascii=False` conserva caracteres como `é` en el texto, en lugar de escribirlos como secuencias escapadas. `encoding="utf-8"` sigue siendo necesario para indicar cómo se escribe ese texto en el archivo.

```python
with open("productos.json", "w", encoding="utf-8") as archivo:
    json.dump(productos, archivo, indent=4, ensure_ascii=False)
```

#### Resumen

```text
loads()   JSON string → Python
dumps()   Python → JSON string

load()    JSON file → Python
dump()    Python → JSON file
```

La **s** significa **string**.

---

## Actividad: Analizar productos

Usa `productos.json`:

```json
[
    {"nombre": "Laptop", "precio": 15000, "stock": 5},
    {"nombre": "Mouse", "precio": 450, "stock": 12},
    {"nombre": "Monitor", "precio": 4200, "stock": 3},
    {"nombre": "Webcam", "precio": 800, "stock": 0}
]
```

Crea un programa que:

1.  Cargue los productos.
2.  Muestre únicamente productos con stock mayor a `0`.
3.  Calcule el valor total del inventario.

```python
precio * stock
```

---

## 6. Entornos virtuales

Un entorno virtual permite que cada proyecto tenga sus propias librerías
y versiones.

Activarlo hace que la terminal use el Python y los paquetes de ese entorno. No cambia los datos ni copia tu código. `.venv` es el nombre de carpeta que elegimos; no es un archivo de configuración.

```text
Proyecto A → requests
Proyecto B → requests + fastapi
```

### Crear

Comprueba que `python --version` corresponda a Python 3.10 o superior. En macOS/Linux el comando puede llamarse `python3`; úsalo para crear el entorno. Una vez activado, usa `python`.

```bash
python -m venv .venv
```

### Activar: Windows PowerShell

```powershell
.\.venv\Scripts\Activate.ps1
```

### Activar: Windows CMD

```bat
.venv\Scripts\activate.bat
```

### Activar: macOS/Linux

```bash
source .venv/bin/activate
```

### Instalar paquetes

`python -m pip` ejecuta el módulo `pip` con ese mismo Python. `python -m venv` usa el módulo `venv` para crear el entorno. Instalar un paquete lo hace disponible en ese entorno; `import requests` lo carga en un programa.

```bash
python -m pip install requests
```

### Ver paquetes

```bash
python -m pip list
```

### Desactivar

```bash
deactivate
```

Concepto clave:

```text
venv = entorno aislado para las dependencias
pip  = herramienta para instalar paquetes
```

---

### Reutilizar las versiones del curso

Desde la raíz del repositorio y con el entorno activado:

```bash
python -m pip install -r requirements.txt
```

`-r` lee la lista de dependencias del archivo. Una línea como `requests==2.34.2` fija una versión. `fastapi[standard]` solicita FastAPI junto con sus dependencias opcionales para ejecutar el servidor; los corchetes pertenecen al nombre usado por `pip`, no son una lista de Python.

## Variables de entorno para claves de API

Una variable de entorno es un valor con nombre que un proceso recibe del sistema o de la terminal. Es distinta del entorno virtual: el primero comunica configuración y el segundo aísla dependencias. Una API key es una credencial que identifica y autoriza a tu programa ante un servicio.

La próxima clase necesita una API key. Configúrala en la terminal que ejecutará el cliente; no la escribas en el código. Usa tu propia clave en lugar del marcador del ejemplo:

macOS/Linux:

```bash
export OPENROUTER_API_KEY="TU_CLAVE"
```

Windows PowerShell:

```powershell
$env:OPENROUTER_API_KEY = "TU_CLAVE"
```

Windows CMD:

```bat
set OPENROUTER_API_KEY=TU_CLAVE
```

Leerla desde Python:

```python
import os

api_key = os.getenv("OPENROUTER_API_KEY")
if not api_key:
    raise ValueError("Configura OPENROUTER_API_KEY antes de ejecutar el programa")
```

`os.getenv()` devuelve una cadena o `None`. Cada terminal tiene su propio entorno; el programa hereda el de la terminal donde lo inicias. Un archivo `.env` no se carga automáticamente: aquí no necesitamos una librería adicional. No guardes claves reales ni archivos `.env` en Git.

### Errores de archivos JSON

```python
import json

try:
    with open("productos.json", encoding="utf-8") as archivo:
        productos = json.load(archivo)
except FileNotFoundError:
    print("Crea productos.json con los datos de la actividad")
except json.JSONDecodeError:
    print("El archivo no contiene JSON válido")
```

Captura estos errores en el programa principal. Las funciones de almacenamiento pueden dejarlos propagarse: así una lectura fallida no se confunde con un inventario vacío y no se sobrescribe el archivo por accidente.

---

## Ejercicio final: Inventario persistente

Ejecuta desde `inventory/` con `python main.py`. Crea `data/products.json` copiando los datos de la actividad anterior.

Estructura:

```text
inventory/
├── main.py
├── products.py
├── storage.py
└── data/
    └── products.json
```

### `storage.py`

```python
import json

def cargar_productos(archivo: str) -> list[dict]:
    with open(archivo, encoding="utf-8") as file:
        return json.load(file)

def guardar_productos(
    archivo: str,
    productos: list[dict]
) -> None:
    with open(archivo, "w", encoding="utf-8") as file:
        json.dump(productos, file, indent=4, ensure_ascii=False)
```

### `products.py`

Crea o reutiliza:

Completa las plantillas de la clase 07 si todavía no lo hiciste. `actualizar_stock()` busca el producto, rechaza una cantidad negativa con `ValueError`, asigna el nuevo stock y devuelve `True`; si el producto no existe, devuelve `False`. Aquí `cantidad` es el nuevo total, no un incremento. Solo guarda si la actualización devuelve `True`.

```python
def buscar_producto(
    productos: list[dict],
    nombre: str
) -> dict | None:
    ...

def productos_disponibles(
    productos: list[dict]
) -> list[dict]:
    ...

def actualizar_stock(
    productos: list[dict],
    nombre: str,
    cantidad: int
) -> bool:
    ...
```

### `main.py`

Importa las funciones desde `products` y `storage`, define `main()` y captura `FileNotFoundError`, `json.JSONDecodeError` y `ValueError`. Guarda solamente cuando la carga y la modificación hayan terminado correctamente.

El programa debe:

1.  Cargar productos desde `data/products.json`.
2.  Mostrar productos disponibles.
3.  Buscar un producto.
4.  Reemplazar su stock por una cantidad no negativa, reutilizando la función de la clase 07.
5.  Guardar los cambios en JSON.
6.  Usar los módulos `products` y `storage`.
7.  Ejecutarse con:

```python
if __name__ == "__main__":
    main()
```

---

## Cheat sheet

```python
json.loads(texto)
json.dumps(datos, indent=4)
json.load(archivo)
json.dump(datos, archivo, indent=4)
```

```bash
python -m venv .venv

# Windows CMD
.venv\Scripts\activate.bat

# Windows PowerShell: .\.venv\Scripts\Activate.ps1

# macOS/Linux
source .venv/bin/activate

python -m pip install requests
python -m pip list
deactivate
```

---

[Índice del curso](README.md)
