# Clase 2 --- JSON + entorno virtual

**Duración:** 1 hora

## Objetivos

-   Entender la relación entre JSON y Python.
-   Convertir JSON a objetos de Python y viceversa.
-   Leer y guardar archivos JSON.
-   Crear un entorno virtual con `venv`.
-   Instalar dependencias con `pip`.
-   Separar persistencia de datos y lógica.

------------------------------------------------------------------------

## 1. JSON

JSON es un formato de texto utilizado para almacenar e intercambiar
datos.

``` json
{
    "nombre": "Laptop",
    "precio": 15000,
    "stock": 5,
    "disponible": true
}
```

### JSON vs Python

  JSON     Python
  -------- -----------------
  object   `dict`
  array    `list`
  string   `str`
  number   `int` / `float`
  true     `True`
  false    `False`
  null     `None`

``` python
import json
```

## 2. JSON string → Python

``` python
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

``` python
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

``` json
[
    {"nombre": "Laptop", "precio": 15000, "stock": 5},
    {"nombre": "Mouse", "precio": 450, "stock": 12},
    {"nombre": "Monitor", "precio": 4200, "stock": 3},
    {"nombre": "Webcam", "precio": 800, "stock": 0}
]
```

Leerlo:

``` python
with open("productos.json") as archivo:
    productos = json.load(archivo)

for producto in productos:
    print(producto["nombre"])
```

## 5. Guardar un archivo JSON

``` python
with open("productos.json", "w") as archivo:
    json.dump(productos, archivo, indent=4)
```

### Resumen

``` text
loads()   JSON string → Python
dumps()   Python → JSON string

load()    JSON file → Python
dump()    Python → JSON file
```

La **s** significa **string**.

------------------------------------------------------------------------

# Actividad --- Analizar productos

Usa `productos.json`:

``` json
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

``` python
precio * stock
```

------------------------------------------------------------------------

# 6. Entornos virtuales

Un entorno virtual permite que cada proyecto tenga sus propias librerías
y versiones.

``` text
Proyecto A → requests
Proyecto B → requests + fastapi
```

## Crear

``` bash
python -m venv .venv
```

## Activar --- Windows

``` bash
.venv\Scripts\activate
```

## Activar --- macOS/Linux

``` bash
source .venv/bin/activate
```

## Instalar paquetes

``` bash
pip install requests
```

## Ver paquetes

``` bash
pip list
```

## Desactivar

``` bash
deactivate
```

Concepto clave:

``` text
venv = entorno aislado para las dependencias
pip  = herramienta para instalar paquetes
```

------------------------------------------------------------------------

# Ejercicio final --- Inventario persistente

Estructura:

``` text
inventory/
├── main.py
├── products.py
├── storage.py
└── data/
    └── products.json
```

## `storage.py`

``` python
import json

def cargar_productos(archivo: str) -> list[dict]:
    with open(archivo) as file:
        return json.load(file)

def guardar_productos(
    archivo: str,
    productos: list[dict]
) -> None:
    with open(archivo, "w") as file:
        json.dump(productos, file, indent=4)
```

## `products.py`

Crea o reutiliza:

``` python
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

## `main.py`

El programa debe:

1.  Cargar productos desde `data/products.json`.
2.  Mostrar productos disponibles.
3.  Buscar un producto.
4.  Modificar su stock.
5.  Guardar los cambios en JSON.
6.  Usar los módulos `products` y `storage`.
7.  Ejecutarse con:

``` python
if __name__ == "__main__":
    main()
```

------------------------------------------------------------------------

# Cheat Sheet

``` python
json.loads(texto)
json.dumps(datos, indent=4)
json.load(archivo)
json.dump(datos, archivo, indent=4)
```

``` bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS/Linux
source .venv/bin/activate

pip install requests
pip list
deactivate
```
