# Clase 03: Conjuntos (sets)

**Duración:** 1 hora

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

Un **set** es una colección de elementos **únicos**, sin un orden fijo y sin índices.

```python
frutas = {"manzana", "plátano", "cereza"}

print(frutas)
```

Los sets son útiles cuando queremos:

- Eliminar duplicados.
- Comprobar rápidamente si un elemento existe.
- Comparar grupos de datos.
- Realizar operaciones como unión, intersección y diferencia.

---

## Crear conjuntos

```python
frutas = {"manzana", "plátano", "cereza"}
```

Los elementos duplicados se eliminan automáticamente:

```python
frutas = {"manzana", "plátano", "manzana", "cereza"}

print(frutas)
```

Resultado posible:

```text
{'manzana', 'plátano', 'cereza'}
```

> El orden de los elementos puede cambiar.

También puedes crear un set con `set()`:

```python
frutas = set(("manzana", "plátano", "cereza"))
```

Para crear un set vacío debes usar:

```python
frutas = set()
```

Esto crea un diccionario vacío, no un set:

```python
frutas = {}
```

---

Los elementos de un set deben ser *hashables*: Python debe poder calcular un identificador numérico estable para buscarlos y compararlos. Los números y las cadenas que usamos aquí cumplen ese requisito; las listas y los diccionarios no pueden ser elementos de un set porque su contenido puede cambiar. No necesitas calcular ese identificador ni llamar a `hash()` para trabajar con estos ejemplos.

## Acceder a elementos

Los sets **no tienen índices**, por lo que esto no funciona:

```python
frutas[0]
```

Puedes recorrerlos con `for`:

```python
for fruta in frutas:
    print(fruta)
```

Y comprobar si un elemento existe:

```python
if "manzana" in frutas:
    print("Existe")
```

---

## Agregar elementos

Usa `add()` para agregar un elemento:

```python
frutas = {"manzana", "plátano"}

frutas.add("cereza")
```

Usa `update()` para agregar varios elementos:

```python
frutas.update(["mango", "uva", "pera"])
```

`update()` también puede recibir otros sets:

```python
tropicales = {"mango", "piña"}

frutas.update(tropicales)
```

---

## Eliminar elementos

Con `remove()`:

```python
frutas.remove("manzana")
```

Si el elemento no existe, produce un error.

Con `discard()`:

```python
frutas.discard("manzana")
```

`discard()` no produce error si el elemento no existe.

También puedes usar:

```python
frutas.clear()
```

para eliminar todos los elementos.

---

## Operaciones entre conjuntos

Los sets permiten comparar y combinar colecciones fácilmente.

### Unión

Combina todos los elementos sin duplicados:

```python
frontend = {"HTML", "CSS", "JavaScript"}
backend = {"Python", "SQL", "JavaScript"}

stack = frontend.union(backend)

print(stack)
```

También puedes usar `|`:

```python
stack = frontend | backend
```

---

### Intersección

Obtiene los elementos presentes en ambos sets:

```python
comunes = frontend.intersection(backend)

print(comunes)
```

También puedes usar `&`:

```python
comunes = frontend & backend
```

Resultado:

```text
{'JavaScript'}
```

---

### Diferencia

Obtiene los elementos que existen en el primer set pero no en el segundo:

```python
solo_frontend = frontend.difference(backend)
```

También puedes usar `-`:

```python
solo_frontend = frontend - backend
```

---

### Diferencia simétrica

Obtiene los elementos que están en un set u otro, pero **no en ambos**:

```python
diferentes = frontend.symmetric_difference(backend)
```

También puedes usar `^`:

```python
diferentes = frontend ^ backend
```

---

## Conjuntos, listas y tuplas

| Tipo | Sintaxis | Orden | Duplicados | Modificable |
|---|---|---|---|---|
| List | `[1, 2, 3]` | Sí | Sí | Sí |
| Tuple | `(1, 2, 3)` | Sí | Sí | No |
| Set | `{1, 2, 3}` | No garantizado | No | Sí |

---

## Métodos principales

| Método | Uso |
|---|---|
| `add()` | Agrega un elemento |
| `update()` | Agrega varios elementos |
| `remove()` | Elimina un elemento; error si no existe |
| `discard()` | Elimina un elemento sin error si no existe |
| `clear()` | Vacía el set |
| `union()` | Une sets |
| `intersection()` | Elementos en común |
| `difference()` | Elementos exclusivos del primer set |
| `symmetric_difference()` | Elementos que no están en ambos |

---

## Cheat sheet

```python
# Crear
datos = {"Python", "JavaScript", "SQL"}

# Set vacío
datos = set()

# Cantidad
len(datos)

# Buscar
"Python" in datos

# Agregar
datos.add("Java")
datos.update(["C#", "Go"])

# Eliminar
datos.remove("Java")
datos.discard("Ruby")

# Recorrer
for dato in datos:
    print(dato)

# Operaciones
a = {1, 2}
b = {2, 3}
a | b    # unión
a & b    # intersección
a - b    # diferencia
a ^ b    # diferencia simétrica

# Eliminar duplicados de una lista
numeros = [1, 1, 2, 2, 3, 3]
unicos = set(numeros)
```

---

## Ejercicio final: Comparación de usuarios

Dos aplicaciones tienen las siguientes listas de usuarios:

```python
app_a = [
    "ana", "carlos", "maria", "pedro",
    "lucia", "carlos", "diego", "ana",
    "sofia", "fernando"
]

app_b = [
    "maria", "pedro", "jorge", "lucia",
    "sofia", "elena", "jorge", "raul",
    "pedro", "monica"
]
```

Crea un programa que:

1. Convierta ambas listas a sets para eliminar usuarios duplicados.
2. Muestre cuántos usuarios únicos tiene cada aplicación.
3. Obtenga los usuarios que utilizan **ambas aplicaciones**.
4. Obtenga todos los usuarios únicos considerando las dos aplicaciones.
5. Obtenga los usuarios que utilizan **solamente App A**.
6. Obtenga los usuarios que utilizan **solamente App B**.
7. Obtenga los usuarios que utilizan una aplicación u otra, pero **no ambas**.
8. Agregue `"adrian"` a App A usando `add()`.
9. Intente eliminar `"roberto"` de App B sin provocar un error.

Referencia de operaciones (elige sus argumentos según el ejercicio):

```text
set(iterable)
len(conjunto)
conjunto.add(elemento)
discard()
union()
intersection()
difference()
symmetric_difference()

|
&
-
^
```

### Salida esperada

El orden puede variar porque los sets no mantienen un orden fijo. La salida siguiente corresponde a los pasos 1 a 7, antes de agregar `"adrian"`. Tras el paso 8, App A tiene 9 usuarios únicos; `discard("roberto")` no cambia App B.

```text
Usuarios únicos App A: 8
Usuarios únicos App B: 8

Usuarios en ambas:
{'maria', 'pedro', 'lucia', 'sofia'}

Todos los usuarios:
{'ana', 'carlos', 'maria', 'pedro', 'lucia', 'diego',
 'sofia', 'fernando', 'jorge', 'elena', 'raul', 'monica'}

Solo App A:
{'ana', 'carlos', 'diego', 'fernando'}

Solo App B:
{'jorge', 'elena', 'raul', 'monica'}

En una aplicación pero no en ambas:
{'ana', 'carlos', 'diego', 'fernando',
 'jorge', 'elena', 'raul', 'monica'}
```

---

[Índice del curso](README.md)
