# Python: Listas

> Material de estudio en español inspirado en la progresión temática de W3Schools, reescrito y ampliado con ejemplos y ejercicios propios.

## Objetivos

Al terminar esta sección podrás:

- Crear listas en Python.
- Acceder a elementos individuales o rangos.
- Modificar, agregar y eliminar elementos.
- Recorrer listas con ciclos.
- Crear listas con *list comprehensions*.
- Ordenar y copiar listas.
- Unir listas.
- Utilizar los métodos principales de `list`.

---

# 1. Introducción a las listas

Una **lista** es una colección ordenada de elementos.

Se crea utilizando corchetes `[]`.

```python
frutas = ["manzana", "plátano", "cereza"]
print(frutas)
```

Las listas tienen cuatro características importantes:

1. Mantienen el orden de los elementos.
2. Pueden modificarse después de ser creadas.
3. Permiten valores repetidos.
4. Pueden contener distintos tipos de datos.

```python
datos = ["Adrián", 42, True, 1.74]
```

## Longitud de una lista

Usa `len()` para conocer cuántos elementos contiene.

```python
frutas = ["manzana", "plátano", "cereza"]
print(len(frutas))
```

Resultado:

```text
3
```

## Crear una lista con `list()`

También es posible utilizar el constructor `list()`.

```python
frutas = list(("manzana", "plátano", "cereza"))
print(frutas)
```

## Ejercicio sugerido

Crea una lista llamada `lenguajes` con cinco lenguajes de programación.

Después:

1. Imprime la lista completa.
2. Imprime cuántos elementos tiene.
3. Agrega un lenguaje repetido y observa qué ocurre.

---

# 2. Acceder a elementos de una lista

Cada elemento tiene una posición llamada **índice**.

Los índices comienzan en `0`.

```python
frutas = ["manzana", "plátano", "cereza"]

print(frutas[0])
print(frutas[1])
```

Resultado:

```text
manzana
plátano
```

## Índices negativos

Python permite contar desde el final usando números negativos.

```python
frutas = ["manzana", "plátano", "cereza"]

print(frutas[-1])
```

Resultado:

```text
cereza
```

`-1` representa el último elemento, `-2` el penúltimo, etc.

## Obtener un rango

Puedes obtener una parte de una lista usando *slicing*.

```python
numeros = [10, 20, 30, 40, 50, 60]

print(numeros[1:4])
```

Resultado:

```python
[20, 30, 40]
```

El límite inicial se incluye y el final no.

También puedes omitir uno de los límites:

```python
print(numeros[:3])
print(numeros[3:])
```

## Comprobar si un elemento existe

```python
frutas = ["manzana", "plátano", "cereza"]

if "plátano" in frutas:
    print("Sí existe")
```

## Ejercicio sugerido

Dada la lista:

```python
ciudades = ["Querétaro", "Medellín", "Bogotá", "Madrid", "Tokio"]
```

Obtén:

1. La primera ciudad.
2. La última ciudad sin usar el índice `4`.
3. Las tres ciudades centrales.
4. Comprueba si `"Bogotá"` está en la lista.

---

# 3. Modificar elementos

Las listas son **mutables**, por lo que sus elementos pueden cambiar.

```python
frutas = ["manzana", "plátano", "cereza"]

frutas[1] = "mango"
print(frutas)
```

Resultado:

```python
['manzana', 'mango', 'cereza']
```

## Modificar varios elementos

También puedes reemplazar un rango.

```python
frutas = ["manzana", "plátano", "cereza", "uva"]

frutas[1:3] = ["mango", "pera"]
print(frutas)
```

Python incluso permite sustituir un rango por una cantidad diferente de elementos.

```python
frutas[1:2] = ["mango", "pera", "kiwi"]
```

## Ejercicio sugerido

Crea:

```python
precios = [100, 200, 300, 400]
```

Después:

1. Cambia `200` por `250`.
2. Reemplaza `300` y `400` por `350`, `450` y `500`.
3. Imprime la lista final.

---

# 4. Agregar elementos

## `append()`

Agrega un elemento al final.

```python
frutas = ["manzana", "plátano"]
frutas.append("cereza")

print(frutas)
```

## `insert()`

Agrega un elemento en una posición específica.

```python
frutas = ["manzana", "cereza"]
frutas.insert(1, "plátano")

print(frutas)
```

## `extend()`

Agrega todos los elementos de otro iterable.

```python
frutas = ["manzana", "plátano"]
tropicales = ["mango", "piña"]

frutas.extend(tropicales)
print(frutas)
```

`extend()` también puede recibir otras colecciones iterables, por ejemplo una tupla.

```python
frutas.extend(("uva", "pera"))
```

## Ejercicio sugerido

Comienza con:

```python
usuarios = ["ana", "luis"]
```

Haz lo siguiente:

1. Agrega `"maria"` al final.
2. Inserta `"carlos"` en la segunda posición.
3. Agrega los usuarios `"pedro"` y `"sofia"` usando una sola operación.

---

# 5. Eliminar elementos

## `remove()`

Elimina la primera aparición de un valor.

```python
frutas = ["manzana", "plátano", "cereza"]
frutas.remove("plátano")
```

## `pop()`

Elimina un elemento por índice y además devuelve el valor eliminado.

```python
frutas = ["manzana", "plátano", "cereza"]

eliminada = frutas.pop(1)
print(eliminada)
```

Si no proporcionas un índice, elimina el último elemento.

```python
frutas.pop()
```

## `del`

También puedes eliminar por índice usando `del`.

```python
frutas = ["manzana", "plátano", "cereza"]
del frutas[0]
```

`del` también puede eliminar por completo la variable.

```python
del frutas
```

## `clear()`

Vacía la lista, pero conserva la variable.

```python
frutas = ["manzana", "plátano", "cereza"]
frutas.clear()

print(frutas)
```

Resultado:

```python
[]
```

## Ejercicio sugerido

Usa:

```python
tareas = ["correo", "reunión", "programar", "revisar PR"]
```

Después:

1. Elimina `"reunión"` por valor.
2. Elimina el último elemento usando `pop()`.
3. Guarda el elemento eliminado en una variable.
4. Vacía la lista completamente.

---

# 6. Recorrer listas

## `for`

La forma más común de recorrer una lista es con un ciclo `for`.

```python
frutas = ["manzana", "plátano", "cereza"]

for fruta in frutas:
    print(fruta)
```

## Recorrer utilizando índices

Puedes combinar `range()` y `len()`.

```python
frutas = ["manzana", "plátano", "cereza"]

for i in range(len(frutas)):
    print(i, frutas[i])
```

## `while`

También puedes utilizar un ciclo `while`.

```python
frutas = ["manzana", "plátano", "cereza"]

i = 0

while i < len(frutas):
    print(frutas[i])
    i += 1
```

## Ejercicio sugerido

Dada:

```python
ventas = [1200, 850, 1500, 2000, 700]
```

1. Imprime cada venta.
2. Imprime únicamente las ventas mayores a `1000`.
3. Calcula manualmente la suma usando un `for` y una variable acumuladora.

---

# 7. List Comprehension

Una **list comprehension** permite construir una lista nueva de forma compacta.

Forma general:

```python
nueva_lista = [expresion for elemento in iterable]
```

Ejemplo:

```python
numeros = [1, 2, 3, 4, 5]
cuadrados = [n ** 2 for n in numeros]

print(cuadrados)
```

Resultado:

```python
[1, 4, 9, 16, 25]
```

## Agregar una condición

```python
numeros = [1, 2, 3, 4, 5, 6]
pares = [n for n in numeros if n % 2 == 0]

print(pares)
```

Resultado:

```python
[2, 4, 6]
```

## Transformar elementos

```python
nombres = ["ana", "luis", "maria"]
mayusculas = [nombre.upper() for nombre in nombres]
```

## Expresión condicional

También puedes utilizar `if/else` dentro de la expresión.

```python
numeros = [1, 2, 3, 4]
resultado = ["par" if n % 2 == 0 else "impar" for n in numeros]
```

## Ejercicio sugerido

Usa:

```python
precios = [100, 250, 80, 500, 320]
```

Crea mediante *list comprehension*:

1. Una lista con todos los precios multiplicados por `1.16`.
2. Una lista únicamente con precios mayores a `200`.
3. Una lista donde cada valor se convierta en `"caro"` si es mayor a `300` o `"barato"` en caso contrario.

---

# 8. Ordenar listas

## `sort()`

Ordena la lista original.

```python
numeros = [50, 10, 40, 20]
numeros.sort()

print(numeros)
```

Resultado:

```python
[10, 20, 40, 50]
```

## Orden descendente

```python
numeros.sort(reverse=True)
```

## Ordenar texto

```python
frutas = ["pera", "manzana", "uva", "cereza"]
frutas.sort()
```

## Usar una función como criterio

El parámetro `key` permite personalizar el orden.

```python
numeros = [-10, 3, -2, 8]
numeros.sort(key=abs)

print(numeros)
```

## `sorted()`

Si quieres conservar la lista original, puedes usar `sorted()`.

```python
numeros = [50, 10, 40, 20]
ordenados = sorted(numeros)

print(numeros)
print(ordenados)
```

## `reverse()`

Invierte el orden actual, sin ordenar por valor.

```python
frutas = ["a", "b", "c"]
frutas.reverse()
```

## Ejercicio sugerido

Dada:

```python
calificaciones = [8.5, 10, 7.2, 9.1, 6.8]
```

1. Ordénala de menor a mayor.
2. Ordénala de mayor a menor.
3. Crea una segunda lista ordenada sin modificar la original.

---

# 9. Copiar listas

Asignar una lista a otra variable **no crea una copia independiente**.

```python
a = [1, 2, 3]
b = a

b.append(4)

print(a)
```

Resultado:

```python
[1, 2, 3, 4]
```

Ambas variables hacen referencia a la misma lista.

## `copy()`

```python
a = [1, 2, 3]
b = a.copy()

b.append(4)

print(a)
print(b)
```

## `list()`

También puedes crear una copia con el constructor.

```python
b = list(a)
```

## Slicing

Otra alternativa frecuente es:

```python
b = a[:]
```

> Estas técnicas realizan una copia superficial (*shallow copy*). Si la lista contiene otras listas, existen consideraciones adicionales.

## Ejercicio sugerido

Crea:

```python
original = [10, 20, 30]
```

1. Asigna `original` a una variable llamada `referencia`.
2. Crea otra variable llamada `copia` usando `copy()`.
3. Agrega `40` a `referencia`.
4. Agrega `50` a `copia`.
5. Imprime las tres variables y explica el resultado.

---

# 10. Unir listas

## Operador `+`

```python
frontend = ["HTML", "CSS", "JavaScript"]
backend = ["Python", "SQL"]

stack = frontend + backend
print(stack)
```

## `extend()`

`extend()` modifica la lista existente.

```python
frontend = ["HTML", "CSS", "JavaScript"]
backend = ["Python", "SQL"]

frontend.extend(backend)
print(frontend)
```

## Con un ciclo

```python
lista1 = [1, 2, 3]
lista2 = [4, 5, 6]

for elemento in lista2:
    lista1.append(elemento)
```

## Diferencia importante

```python
lista1 + lista2
```

crea una nueva lista.

Mientras que:

```python
lista1.extend(lista2)
```

modifica `lista1`.

## Ejercicio sugerido

Crea:

```python
alumnos_a = ["Ana", "Luis", "Carlos"]
alumnos_b = ["María", "Sofía"]
```

1. Crea una nueva lista con ambos grupos usando `+`.
2. Repite el ejercicio usando `extend()`.
3. Observa cuál de las listas originales cambia.

---

# 11. Métodos principales de las listas

Python incluye varios métodos útiles para trabajar con listas.

| Método | Uso |
|---|---|
| `append(x)` | Agrega `x` al final. |
| `clear()` | Elimina todos los elementos. |
| `copy()` | Crea una copia superficial. |
| `count(x)` | Cuenta cuántas veces aparece `x`. |
| `extend(iterable)` | Agrega elementos de otro iterable. |
| `index(x)` | Devuelve el índice de la primera aparición de `x`. |
| `insert(i, x)` | Inserta `x` en la posición `i`. |
| `pop(i)` | Elimina y devuelve el elemento de la posición `i`. |
| `remove(x)` | Elimina la primera aparición de `x`. |
| `reverse()` | Invierte el orden actual. |
| `sort()` | Ordena la lista. |

## Ejemplo integrado

```python
numeros = [5, 2, 5, 8, 1]

numeros.append(10)
numeros.insert(1, 7)

print(numeros.count(5))
print(numeros.index(8))

numeros.remove(5)
numeros.sort()

print(numeros)
```

---

# 12. Ejercicio final

Construye un pequeño programa para administrar una lista de compras.

Comienza con:

```python
compras = ["leche", "pan", "huevos"]
```

El programa debe:

1. Mostrar todos los productos.
2. Agregar `"café"`.
3. Insertar `"agua"` al inicio.
4. Eliminar `"pan"`.
5. Mostrar cuántos productos quedan.
6. Ordenar los productos alfabéticamente.
7. Crear una segunda lista llamada `compras_mayusculas` usando *list comprehension*.
8. Imprimir ambas listas.

Resultado esperado aproximado:

```text
Lista original:
['agua', 'café', 'huevos', 'leche']

Lista en mayúsculas:
['AGUA', 'CAFÉ', 'HUEVOS', 'LECHE']
```

## Reto adicional

Permite que el usuario agregue un producto desde teclado:

```python
producto = input("Producto: ")
compras.append(producto)
```

Después comprueba si el producto ya existía antes de agregarlo.

---

# Cheat Sheet

```python
# Crear
lista = [1, 2, 3]

# Acceder
lista[0]
lista[-1]
lista[1:3]

# Longitud
len(lista)

# Agregar
lista.append(4)
lista.insert(0, 0)
lista.extend([5, 6])

# Modificar
lista[0] = 100

# Eliminar
lista.remove(2)
lista.pop()
del lista[0]
lista.clear()

# Recorrer
for elemento in lista:
    print(elemento)

# List comprehension
cuadrados = [n ** 2 for n in range(10)]

# Ordenar
lista.sort()
lista.sort(reverse=True)
nueva = sorted(lista)

# Copiar
copia = lista.copy()

# Unir
lista3 = lista1 + lista2
lista1.extend(lista2)
```

---

# Preguntas de repaso

1. ¿Cuál es el índice del primer elemento de una lista?
2. ¿Qué diferencia existe entre `append()` e `insert()`?
3. ¿Qué diferencia existe entre `remove()` y `pop()`?
4. ¿Por qué `b = a` no crea una copia independiente de una lista?
5. ¿Qué diferencia existe entre `sort()` y `sorted()`?
6. ¿Qué hace `extend()`?
7. ¿Para qué sirve una *list comprehension*?
8. ¿Cómo accederías al último elemento sin conocer la longitud de la lista?

---

## Referencia

Estructura temática basada en la sección **Python Lists** de W3Schools:

https://www.w3schools.com/python/python_lists.asp
