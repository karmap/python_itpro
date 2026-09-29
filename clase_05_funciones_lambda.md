# Clase 05: Funciones y lambda

**Duración:** 1 a 2 horas

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

Una **función** es un bloque de código reutilizable que realiza una tarea específica.

`def` define la función, pero no ejecuta su cuerpo. La llamada `saludar()` sí lo ejecuta. Los parámetros se escriben en la definición; los argumentos son los valores que entregamos al llamar.

```python
def saludar():
    print("Hola")

saludar()
```

## Parámetros y retorno

`return` termina la función y entrega un valor al código que la llamó. `print()` solo lo muestra en pantalla. Si una función termina sin `return`, devuelve `None`, que representa la ausencia de un resultado. Por eso la función `saludar()` anterior imprime un saludo, pero no devuelve ese texto.

```python
def sumar(a, b):
    return a + b

resultado = sumar(10, 5)
print(resultado)
```

### Valores por defecto

Un valor por defecto se usa cuando omites ese argumento. En este ejemplo, `saludar("Ana")` usa `"Ana"` y `saludar()` usa `"Usuario"`.

```python
def saludar(nombre="Usuario"):
    print("Hola", nombre)

saludar("Ana")
saludar()
```

### Argumentos por nombre

Puedes entregar los argumentos por posición o por nombre. Al usar nombres, deben coincidir con los parámetros de la definición; su orden puede cambiar.

```python
def crear_usuario(nombre, edad, pais):
    print(nombre, edad, pais)

crear_usuario(nombre="Ana", pais="México", edad=25)
```

### Número variable de argumentos

```python
def sumar(*numeros):
    total = 0
    for numero in numeros:
        total += numero
    return total

print(sumar(10, 20, 30, 40))
```

Dentro de esta función, `numeros` es una **tupla**. El nombre habitual `args` es una convención.

Las plantillas con `...` son sintácticamente válidas, pero todavía no implementan la lógica solicitada.

---

## Funciones con listas y diccionarios

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

Con diccionarios:

```python
def mostrar_usuario(usuario):
    print(usuario["nombre"])
    print(usuario["edad"])
```

Dividir un programa en funciones permite separar responsabilidades:

```python
def calcular_total(productos):
    ...  # Completa esta función durante la actividad.

def encontrar_producto(productos):
    ...  # Completa esta función durante la actividad.

def mostrar_resultados(productos):
    ...  # Completa esta función durante la actividad.
```

---

## Alcance de las variables

Una variable creada dentro de una función normalmente solo existe dentro de ella:

```python
def calcular():
    resultado = 10 + 20
    print(resultado)
```

En general, es preferible pasar información mediante parámetros y devolver resultados con `return`.

---

## Lambda

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

### Lambda para ordenar

`key` recibe una función que calcula el criterio de comparación para cada elemento. `lambda producto: producto[1]` obtiene el precio de cada producto; `reverse=True` ordena de mayor a menor. La función se entrega a `sort()` para que este la llame durante la ordenación.

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

| Función | Lambda |
|---|---|
| Usa `def` | Usa `lambda` |
| Puede tener varias líneas | Una expresión |
| Lógica compleja | Operaciones pequeñas |
| Más fácil de leer | Más compacta |

---

`sum(iterable)` suma sus números, por ejemplo `sum([10, 20, 30])` devuelve `60`. Es una función incorporada, diferente de las funciones `sumar()` que hemos definido. En el ejercicio, `.lower()` permite comparar nombres sin distinguir mayúsculas de minúsculas; no elimina acentos.

## Cheat sheet

```python
def sumar(a, b):
    return a + b

def saludar(nombre="Usuario"):
    print(nombre)

def sumar_varios(*numeros):
    return sum(numeros)

duplicar = lambda x: x * 2

sumar = lambda a, b: a + b

productos = [
    {"nombre": "Café", "precio": 8},
    {"nombre": "Pan", "precio": 9}
]
productos.sort(
    key=lambda producto: producto["precio"]
)
```

---

## Ejercicio final: Procesador de productos

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
    ...  # Completa esta función durante la actividad.

def calcular_valor(productos):
    ...  # Completa esta función durante la actividad.

def buscar_producto(productos, nombre):
    ...  # Completa esta función durante la actividad.

def mostrar_productos(productos):
    ...  # Completa esta función durante la actividad.
```

### Requisitos

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

## Salida esperada aproximada

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

### Práctica opcional

Agrega un parámetro opcional:

```python
def mostrar_productos(productos, mostrar_sin_stock=False):
    ...
```

Si es `False`, no debe mostrar productos sin stock. Si es `True`, debe mostrarlos todos.

---

[Índice del curso](README.md)
