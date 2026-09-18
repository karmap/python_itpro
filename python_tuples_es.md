# Python: Tuplas

Una **tupla** (*tuple*) es una colección ordenada de elementos similar a una lista.

La diferencia principal es que una tupla es **inmutable**: después de crearla, no puedes modificar, agregar o eliminar directamente sus elementos.

```python
frutas = ("manzana", "plátano", "cereza")
print(frutas)
```

## Crear tuplas

```python
frutas = ("manzana", "plátano", "cereza")
persona = ("Adrian", 42, True, 1.74)
numeros = (1, 2, 2, 3, 3, 3)

print(len(numeros))
```

### Tupla de un solo elemento

Necesitas una coma:

```python
producto = ("cafe",)
```

Esto no es una tupla:

```python
producto = ("cafe")
```

También puedes usar `tuple()`:

```python
frutas = tuple(("manzana", "plátano", "cereza"))
```

---

# Acceder a elementos

```python
frutas = ("manzana", "plátano", "cereza")

print(frutas[0])
print(frutas[-1])
```

*Slicing*:

```python
numeros = (10, 20, 30, 40, 50)
print(numeros[1:4])
```

Comprobar si existe:

```python
if "manzana" in frutas:
    print("Existe")
```

---

# Las tuplas son inmutables

Esto produce un error:

```python
frutas[1] = "mango"
```

Para modificar, puedes convertir temporalmente a lista:

```python
frutas = ("manzana", "plátano", "cereza")

lista = list(frutas)
lista[1] = "mango"
frutas = tuple(lista)

print(frutas)
```

---

# Desempaquetar tuplas

```python
persona = ("Adrian", 42, "México")
nombre, edad, pais = persona
```

Con `*`:

```python
numeros = (10, 20, 30, 40, 50)
primero, *medio, ultimo = numeros
```

`medio` será `[20, 30, 40]`.

---

# Recorrer una tupla

```python
for fruta in frutas:
    print(fruta)
```

Mediante índices:

```python
for i in range(len(frutas)):
    print(frutas[i])
```

---

# Unir y repetir tuplas

```python
frontend = ("HTML", "CSS")
backend = ("Python", "SQL")

stack = frontend + backend
```

Repetir:

```python
numeros = (1, 2, 3)
resultado = numeros * 2
```

Resultado:

```text
(1, 2, 3, 1, 2, 3)
```

---

# Métodos de Tuple

| Método | Uso |
|---|---|
| `count(x)` | Cuenta cuántas veces aparece `x` |
| `index(x)` | Obtiene el índice de la primera aparición de `x` |

```python
numeros = (10, 20, 20, 30, 20)

print(numeros.count(20))
print(numeros.index(30))
```

---

# Tuple vs List

| List | Tuple |
|---|---|
| `[1, 2, 3]` | `(1, 2, 3)` |
| Mutable | Inmutable |
| Permite agregar elementos | No permite agregar directamente |
| Permite eliminar elementos | No permite eliminar directamente |
| Más métodos disponibles | Solo `count()` e `index()` |

Usa una **lista** cuando los datos necesitan cambiar.

Usa una **tupla** cuando representan un conjunto de valores que debería mantenerse fijo.

---

# Cheat Sheet

```python
tupla = (10, 20, 30)
tupla = (10,)

len(tupla)

tupla[0]
tupla[-1]
tupla[1:3]

20 in tupla

tupla.count(20)
tupla.index(30)

a, b, c = tupla
primero, *resto = tupla

for elemento in tupla:
    print(elemento)

nueva = tupla1 + tupla2
nueva = tupla * 2

lista = list(tupla)
tupla = tuple(lista)
```

---

# 🟡 Ejercicio final — Análisis de temperaturas

```python
temperaturas = (
    22, 24, 21, 25, 27, 27, 23,
    20, 22, 24, 27, 26, 21, 27
)
```

Crea un programa que:

1. Muestre cuántas temperaturas fueron registradas usando `len()`.
2. Muestre la primera y la última temperatura.
3. Cuente cuántas veces apareció `27` usando `count()`.
4. Encuentre la primera posición donde apareció `27` usando `index()`.
5. Obtenga las temperaturas de la primera semana mediante *slicing*.
6. Desempaque las primeras tres temperaturas en `dia1`, `dia2` y `dia3`.
7. Recorra la tupla y muestre únicamente temperaturas mayores a `24`.
8. Convierta la tupla a una lista, agregue una nueva temperatura de `28` y conviértala nuevamente a tupla.

## Output esperado aproximado

```text
Total de registros: 14
Primera temperatura: 22
Última temperatura: 27
Veces que apareció 27: 4
Primera posición de 27: 4

Primera semana:
(22, 24, 21, 25, 27, 27, 23)

Primeros tres días:
22 24 21

Temperaturas mayores a 24:
25
27
27
27
26
27

Nueva tupla:
(22, 24, 21, 25, 27, 27, 23, 20, 22, 24, 27, 26, 21, 27, 28)
```
