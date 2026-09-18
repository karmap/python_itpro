# Python: Functions y Lambda

Una **función** es un bloque de código reutilizable que realiza una tarea específica.

```python
def saludar():
    print("Hola")

saludar()
```

# Parámetros y Return

```python
def sumar(a, b):
    return a + b

resultado = sumar(10, 5)
print(resultado)
```

## Valores por defecto

```python
def saludar(nombre="Usuario"):
    print("Hola", nombre)

saludar("Ana")
saludar()
```

## Argumentos por nombre

```python
def crear_usuario(nombre, edad, pais):
    print(nombre, edad, pais)

crear_usuario(nombre="Ana", pais="México", edad=25)
```

## Número variable de argumentos

```python
def sumar(*numeros):
    total = 0
    for numero in numeros:
        total += numero
    return total

print(sumar(10, 20, 30, 40))
```

`args` se comporta como una **tupla**.

---

# Funciones con Lists y Dictionaries

```python
def mostrar_productos(productos):
    for producto in productos:
        print(producto.upper())
```

Una función también puede devolver una nueva lista:

```python
def convertir_mayusculas(productos):
    resultado = []

    for producto in productos:
        resultado.append(producto.upper())

    return resultado
```

Con dictionaries:

```python
def mostrar_usuario(usuario):
    print(usuario["nombre"])
    print(usuario["edad"])
```

Dividir un programa en funciones permite separar responsabilidades:

```python
def calcular_total(productos):
    # ...

def encontrar_producto(productos):
    # ...

def mostrar_resultados(productos):
    # ...
```

---

# Scope

Una variable creada dentro de una función normalmente solo existe dentro de ella:

```python
def calcular():
    resultado = 10 + 20
    print(resultado)
```

En general, es preferible pasar información mediante parámetros y devolver resultados con `return`.

---

# Lambda

Una función `lambda` es una función pequeña escrita en una sola expresión.

Función normal:

```python
def duplicar(numero):
    return numero * 2
```

Con lambda:

```python
duplicar = lambda numero: numero * 2

print(duplicar(10))
```

Con varios parámetros:

```python
sumar = lambda a, b: a + b
```

## Lambda para ordenar

```python
productos = [
    ["cafe", 8],
    ["pan", 9],
    ["leche", 6]
]

productos.sort(
    key=lambda producto: producto[1],
    reverse=True
)
```

Resultado:

```python
[
    ["pan", 9],
    ["cafe", 8],
    ["leche", 6]
]
```

| Function | Lambda |
|---|---|
| Usa `def` | Usa `lambda` |
| Puede tener varias líneas | Una expresión |
| Lógica compleja | Operaciones pequeñas |
| Más fácil de leer | Más compacta |

---

# Cheat Sheet

```python
def sumar(a, b):
    return a + b

def saludar(nombre="Usuario"):
    print(nombre)

def sumar_varios(*numeros):
    return sum(numeros)

duplicar = lambda x: x * 2

sumar = lambda a, b: a + b

productos.sort(
    key=lambda producto: producto["precio"]
)
```

---

# 🟡 Ejercicio final — Procesador de productos

```python
productos = [
    {"nombre": "Laptop", "precio": 15000, "stock": 5},
    {"nombre": "Mouse", "precio": 450, "stock": 12},
    {"nombre": "Teclado", "precio": 900, "stock": 8},
    {"nombre": "Monitor", "precio": 4200, "stock": 3},
    {"nombre": "Audifonos", "precio": 1200, "stock": 0},
    {"nombre": "Webcam", "precio": 800, "stock": 6}
]
```

Crea:

```python
def productos_disponibles(productos):
    # ...

def calcular_valor(productos):
    # ...

def buscar_producto(productos, nombre):
    # ...

def mostrar_productos(productos):
    # ...
```

## Requisitos

1. `productos_disponibles()` debe devolver una nueva lista únicamente con productos cuyo `stock` sea mayor a `0`.

2. `calcular_valor()` debe calcular el valor total del inventario usando `precio × stock`.

3. `buscar_producto()` debe buscar sin importar mayúsculas o minúsculas usando `.lower()`.

```python
buscar_producto(productos, "LAPTOP")
buscar_producto(productos, "laptop")
```

4. `mostrar_productos()` debe usar `.upper()` y producir:

```text
LAPTOP - $15000 - Stock: 5
MOUSE - $450 - Stock: 12
```

5. Ordena los productos de **mayor a menor precio** utilizando `sort()` y `lambda`.

---

# Output esperado aproximado

```text
PRODUCTOS DISPONIBLES

LAPTOP - $15000 - Stock: 5
MOUSE - $450 - Stock: 12
TECLADO - $900 - Stock: 8
MONITOR - $4200 - Stock: 3
WEBCAM - $800 - Stock: 6

Valor total del inventario:
$105000

Búsqueda: LAPTOP

Producto encontrado:
Laptop - $15000

ORDENADOS POR PRECIO

LAPTOP - $15000
MONITOR - $4200
AUDIFONOS - $1200
TECLADO - $900
WEBCAM - $800
MOUSE - $450
```

## Bonus

Agrega un parámetro opcional:

```python
def mostrar_productos(productos, mostrar_sin_stock=False):
```

Si es `False`, no debe mostrar productos sin stock. Si es `True`, debe mostrarlos todos.
