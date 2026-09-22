# Clase 1 — Python moderno: Type Hints, utilidades y módulos

**Duración:** 2 horas  
**Objetivo:** escribir código Python más claro, aprovechar herramientas modernas del lenguaje y aprender a organizar un programa en varios módulos.

---

## 1. Type Hints

Python permite indicar qué tipos de datos esperamos recibir y devolver.

```python
def calcular_total(precio: float, cantidad: int) -> float:
    return precio * cantidad
```

Tipos comunes:

```python
nombre: str = "Laptop"
cantidad: int = 5
precio: float = 14999.99
disponible: bool = True
```

Colecciones:

```python
def mostrar_productos(productos: list[str]) -> None:
    for producto in productos:
        print(producto)

def obtener_productos() -> list[dict]:
    return [
        {"nombre": "Laptop", "precio": 15000},
        {"nombre": "Mouse", "precio": 450}
    ]
```

Los Type Hints no obligan a Python a respetar los tipos. Sirven como información para desarrolladores, IDEs, analizadores y frameworks como FastAPI.

---

## 2. `None` y valores opcionales

`None` representa la ausencia de un valor.

```python
def buscar_producto(productos: list[dict], nombre: str) -> dict | None:
    for producto in productos:
        if producto["nombre"] == nombre:
            return producto

    return None
```

Uso:

```python
producto = buscar_producto(productos, "Laptop")

if producto is None:
    print("Producto no encontrado")
else:
    print(producto)
```

`dict | None` significa que la función puede regresar un dictionary o `None`.

---

# Actividad 1 — Type Hints y `None`

```python
usuarios = [
    {"id": 1, "nombre": "Ana", "activo": True},
    {"id": 2, "nombre": "Luis", "activo": False},
    {"id": 3, "nombre": "Carlos", "activo": True}
]
```

Crea:

```python
buscar_usuario(usuarios, user_id)
usuarios_activos(usuarios)
```

Requisitos:

1. Agregar Type Hints a parámetros y retornos.
2. `buscar_usuario()` debe regresar el usuario o `None`.
3. `usuarios_activos()` debe regresar una lista.
4. Mostrar `"Usuario no encontrado"` usando `is None`.
5. Probar con un ID existente y uno inexistente.

Salida esperada aproximada:

```text
Ana
Usuario no encontrado

Usuarios activos:
Ana
Carlos
```

---

## 3. Unpacking

```python
producto = ("Laptop", 15000, 5)

nombre, precio, stock = producto
```

Equivale a acceder manualmente a cada posición.

Con `*` podemos capturar varios elementos:

```python
producto = ["Laptop", 15000, 5, "Lenovo", "Computadoras"]

nombre, *datos = producto
```

Otro ejemplo:

```python
primero, *otros, ultimo = [10, 20, 30, 40, 50]
```

---

## 4. `enumerate()`

En lugar de:

```python
for i in range(len(productos)):
    print(i, productos[i])
```

podemos usar:

```python
for i, producto in enumerate(productos, start=1):
    print(i, producto)
```

Resultado:

```text
1 Laptop
2 Mouse
3 Monitor
```

---

## 5. `zip()`

Permite recorrer varias colecciones simultáneamente:

```python
nombres = ["Laptop", "Mouse", "Monitor"]
precios = [15000, 450, 4200]

for nombre, precio in zip(nombres, precios):
    print(nombre, precio)
```

También podemos crear nuevas estructuras:

```python
productos = list(zip(nombres, precios))
```

o:

```python
productos = dict(zip(nombres, precios))
```

---

## 6. `*args`

Permite recibir cualquier cantidad de argumentos posicionales.

```python
def sumar(*numeros):
    return sum(numeros)

print(sumar(10, 20))
print(sumar(10, 20, 30, 40))
```

Dentro de la función, `numeros` es una tuple.

---

## 7. `**kwargs`

Permite recibir argumentos con nombre:

```python
def mostrar_producto(**datos):
    print(datos)

mostrar_producto(
    nombre="Laptop",
    precio=15000,
    stock=5
)
```

Dentro de la función, `datos` es un dictionary.

```text
*args       → tuple
**kwargs    → dictionary
```

---

## 8. Desempacar argumentos

```python
def crear_producto(nombre: str, precio: float, stock: int) -> None:
    print(nombre, precio, stock)
```

Con una lista o tuple:

```python
datos = ["Laptop", 15000, 5]

crear_producto(*datos)
```

Con un dictionary:

```python
producto = {
    "nombre": "Laptop",
    "precio": 15000,
    "stock": 5
}

crear_producto(**producto)
```

---

# Actividad 2 — Unpacking, `enumerate()`, `zip()` y `**`

```python
nombres = ["Laptop", "Mouse", "Monitor", "Webcam"]
precios = [15000, 450, 4200, 800]
stocks = [5, 12, 3, 0]
```

Crea:

```python
def mostrar_producto(
    nombre: str,
    precio: float,
    stock: int
) -> None:
    ...
```

Requisitos:

1. Utilizar `zip()` para recorrer las tres listas simultáneamente.
2. Crear un dictionary por producto.
3. Guardarlos en una lista llamada `productos`.
4. Utilizar `enumerate(..., start=1)` para recorrer `productos`.
5. Llamar `mostrar_producto()` usando `**producto`.

Ejemplo:

```python
producto = {
    "nombre": "Laptop",
    "precio": 15000,
    "stock": 5
}

mostrar_producto(**producto)
```

Salida aproximada:

```text
1. Laptop - $15000 - Stock: 5
2. Mouse - $450 - Stock: 12
3. Monitor - $4200 - Stock: 3
4. Webcam - $800 - Stock: 0
```

**Bonus:** mostrar `"AGOTADO"` cuando el stock sea `0`.

---

## 9. Módulos

Conforme un programa crece, no queremos tener todo en `main.py`.

Podemos separar responsabilidades:

```text
inventory/
├── main.py
├── products.py
└── reports.py
```

Cada archivo `.py` puede funcionar como un módulo.

### `products.py`

```python
def buscar_producto(
    productos: list[dict],
    nombre: str
) -> dict | None:
    for producto in productos:
        if producto["nombre"] == nombre:
            return producto

    return None
```

Desde `main.py`:

```python
from products import buscar_producto

producto = buscar_producto(productos, "Laptop")
```

---

## 10. Formas de importar

Módulo completo:

```python
import products

products.buscar_producto(productos, "Laptop")
```

Una función:

```python
from products import buscar_producto

buscar_producto(productos, "Laptop")
```

Varias:

```python
from products import buscar_producto, agregar_producto
```

Alias:

```python
import products as p
```

Ejemplos conocidos:

```python
import pandas as pd
import numpy as np
```

---

## 11. Módulos estándar

```python
import random

numero = random.randint(1, 10)
```

```python
import math

print(math.sqrt(25))
```

También:

```python
from math import sqrt

print(sqrt(25))
```

---

## 12. `__name__ == "__main__"`

Cada módulo tiene automáticamente una variable llamada `__name__`.

Si ejecutamos:

```bash
python main.py
```

el archivo principal recibe:

```python
__name__ == "__main__"
```

Por eso es común:

```python
def main():
    print("Iniciando aplicación")


if __name__ == "__main__":
    main()
```

### ¿Por qué?

Sin protección:

```python
# calculadora.py

def sumar(a: int, b: int) -> int:
    return a + b

print("Calculadora iniciada")
```

Al hacer:

```python
import calculadora
```

también se ejecuta el `print`.

Mejor:

```python
def sumar(a: int, b: int) -> int:
    return a + b


def main() -> None:
    print("Calculadora iniciada")


if __name__ == "__main__":
    main()
```

Ahora podemos importar y reutilizar las funciones sin ejecutar `main()`.

---

## 13. Ejemplo completo

Estructura:

```text
inventory/
├── main.py
├── products.py
└── reports.py
```

### `products.py`

```python
def buscar_producto(
    productos: list[dict],
    nombre: str
) -> dict | None:
    for producto in productos:
        if producto["nombre"].lower() == nombre.lower():
            return producto

    return None


def productos_disponibles(
    productos: list[dict]
) -> list[dict]:
    return [
        producto
        for producto in productos
        if producto["stock"] > 0
    ]
```

### `reports.py`

```python
def calcular_valor_inventario(
    productos: list[dict]
) -> float:
    total = 0

    for producto in productos:
        total += producto["precio"] * producto["stock"]

    return total


def producto_mas_caro(
    productos: list[dict]
) -> dict | None:
    if not productos:
        return None

    return max(
        productos,
        key=lambda producto: producto["precio"]
    )
```

### `main.py`

```python
from products import buscar_producto, productos_disponibles
from reports import calcular_valor_inventario, producto_mas_caro


productos = [
    {"nombre": "Laptop", "precio": 15000, "stock": 5},
    {"nombre": "Mouse", "precio": 450, "stock": 12},
    {"nombre": "Monitor", "precio": 4200, "stock": 3},
    {"nombre": "Webcam", "precio": 800, "stock": 0}
]


def main() -> None:
    disponibles = productos_disponibles(productos)

    print("Productos disponibles:")

    for numero, producto in enumerate(disponibles, start=1):
        print(numero, producto["nombre"], producto["precio"])

    total = calcular_valor_inventario(productos)
    print(f"
Valor del inventario: ${total}")

    caro = producto_mas_caro(productos)

    if caro is not None:
        print(f"Producto más caro: {caro['nombre']}")


if __name__ == "__main__":
    main()
```

---

# Ejercicio final — Refactorizar un sistema de inventario

Partimos de:

```python
productos = [
    {"nombre": "Laptop", "precio": 15000, "stock": 5},
    {"nombre": "Mouse", "precio": 450, "stock": 12},
    {"nombre": "Teclado", "precio": 900, "stock": 8},
    {"nombre": "Monitor", "precio": 4200, "stock": 3},
    {"nombre": "Audifonos", "precio": 1200, "stock": 0},
    {"nombre": "Webcam", "precio": 800, "stock": 6}
]


def buscar_producto(productos, nombre):
    for producto in productos:
        if producto["nombre"].lower() == nombre.lower():
            return producto
    return None


def productos_disponibles(productos):
    return [
        producto
        for producto in productos
        if producto["stock"] > 0
    ]


def calcular_valor_inventario(productos):
    total = 0
    for producto in productos:
        total += producto["precio"] * producto["stock"]
    return total


def producto_mas_caro(productos):
    if not productos:
        return None

    return max(
        productos,
        key=lambda producto: producto["precio"]
    )


disponibles = productos_disponibles(productos)

for i, producto in enumerate(disponibles, start=1):
    print(i, producto["nombre"])

print("Valor inventario:", calcular_valor_inventario(productos))
print("Más caro:", producto_mas_caro(productos))
```

## Objetivo

Refactorizarlo a:

```text
inventory/
├── main.py
├── products.py
└── reports.py
```

### `products.py`

Debe contener:

```text
buscar_producto()
productos_disponibles()
```

### `reports.py`

Debe contener:

```text
calcular_valor_inventario()
producto_mas_caro()
```

### `main.py`

Debe contener:

- los datos;
- los imports;
- la ejecución;
- una función `main()`.

Además:

1. Agregar Type Hints a todas las funciones.
2. Utilizar `dict | None` cuando una función pueda no encontrar un resultado.
3. Utilizar `enumerate()` para numerar productos.
4. Utilizar unpacking al menos una vez.
5. Importar las funciones desde sus módulos.
6. Utilizar:

```python
if __name__ == "__main__":
    main()
```

### Bonus

Agregar:

```python
def actualizar_stock(
    productos: list[dict],
    nombre: str,
    cantidad: int
) -> bool:
```

Debe buscar el producto, actualizar su stock y regresar `True` si lo encontró o `False` si no existe.

---

# Cheat Sheet

```python
# Type hints
nombre: str
edad: int
precio: float
activo: bool

list[str]
dict[str, int]

def funcion(valor: int) -> str:
    ...


# Opcional
def buscar() -> dict | None:
    ...


# None
if resultado is None:
    ...


# Unpacking
nombre, precio, stock = producto

primero, *otros = datos


# enumerate
for i, producto in enumerate(productos, start=1):
    ...


# zip
for nombre, precio in zip(nombres, precios):
    ...


# *args
def funcion(*args):
    ...


# **kwargs
def funcion(**kwargs):
    ...


# Desempacar
funcion(*datos)
funcion(**datos)


# Imports
import products
from products import buscar_producto
import products as p


# Main
def main():
    ...


if __name__ == "__main__":
    main()
```
