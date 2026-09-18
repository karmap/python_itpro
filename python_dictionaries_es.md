# Python: Diccionarios

Un **diccionario** (*dictionary*) almacena información en pares de **clave : valor**.

```python
persona = {
    "nombre": "Ana",
    "edad": 28,
    "pais": "México"
}
```

En lugar de acceder mediante una posición como en una lista, accedemos mediante una **clave**:

```python
print(persona["nombre"])
```

Resultado:

```text
Ana
```

Los diccionarios son útiles para representar datos con una estructura clara: usuarios, productos, configuraciones, registros, etc.

---

# Crear diccionarios

```python
producto = {
    "nombre": "Laptop",
    "precio": 15000,
    "stock": 8
}
```

Los valores pueden ser de diferentes tipos:

```python
usuario = {
    "nombre": "Carlos",
    "edad": 32,
    "activo": True,
    "lenguajes": ["Python", "JavaScript"]
}
```

Puedes conocer la cantidad de elementos con `len()`:

```python
print(len(usuario))
```

---

# Acceder a valores

Puedes utilizar la clave:

```python
print(usuario["nombre"])
```

O `get()`:

```python
print(usuario.get("nombre"))
```

Una diferencia importante es que `get()` puede evitar un error cuando la clave no existe:

```python
print(usuario.get("telefono"))
```

También puedes comprobar si una clave existe:

```python
if "edad" in usuario:
    print("La edad está registrada")
```

---

# Modificar valores

```python
producto = {
    "nombre": "Laptop",
    "precio": 15000,
    "stock": 8
}

producto["precio"] = 14000
```

También puedes usar `update()`:

```python
producto.update({"stock": 12})
```

---

# Agregar elementos

Para agregar una nueva clave:

```python
producto["marca"] = "Lenovo"
```

O con `update()`:

```python
producto.update({"categoria": "Computadoras"})
```

---

# Eliminar elementos

Con `pop()`:

```python
producto.pop("stock")
```

Con `del`:

```python
del producto["precio"]
```

Para eliminar todos los elementos:

```python
producto.clear()
```

---

# Obtener claves y valores

`keys()` devuelve las claves:

```python
print(producto.keys())
```

`values()` devuelve los valores:

```python
print(producto.values())
```

`items()` devuelve pares de clave y valor:

```python
print(producto.items())
```

Ejemplo:

```python
for clave, valor in producto.items():
    print(clave, valor)
```

---

# Recorrer diccionarios

Recorrer las claves:

```python
for clave in producto:
    print(clave)
```

Recorrer los valores:

```python
for valor in producto.values():
    print(valor)
```

Recorrer claves y valores:

```python
for clave, valor in producto.items():
    print(clave, valor)
```

---

# Diccionarios anidados

Un diccionario puede contener otros diccionarios:

```python
usuarios = {
    "usuario1": {
        "nombre": "Ana",
        "edad": 25
    },
    "usuario2": {
        "nombre": "Carlos",
        "edad": 32
    }
}
```

Para acceder a un valor:

```python
print(usuarios["usuario1"]["nombre"])
```

Resultado:

```text
Ana
```

También es común tener una **lista de diccionarios**:

```python
productos = [
    {"nombre": "Laptop", "precio": 15000},
    {"nombre": "Mouse", "precio": 500},
    {"nombre": "Teclado", "precio": 900}
]
```

Esta estructura es muy común al trabajar con APIs, bases de datos y JSON.

---

# Copiar diccionarios

No es recomendable copiar así:

```python
copia = producto
```

Ambas variables apuntarían al mismo diccionario.

Utiliza `copy()`:

```python
copia = producto.copy()
```

También puedes usar:

```python
copia = dict(producto)
```

---

# Métodos principales

| Método | Uso |
|---|---|
| `get()` | Obtiene un valor |
| `keys()` | Obtiene las claves |
| `values()` | Obtiene los valores |
| `items()` | Obtiene claves y valores |
| `update()` | Agrega o modifica elementos |
| `pop()` | Elimina una clave |
| `clear()` | Vacía el diccionario |
| `copy()` | Crea una copia |

---

# List vs Tuple vs Set vs Dictionary

| Tipo | Ejemplo | Uso principal |
|---|---|---|
| List | `["Ana", "Luis"]` | Colección ordenada y modificable |
| Tuple | `("Ana", "Luis")` | Colección ordenada e inmutable |
| Set | `{"Ana", "Luis"}` | Elementos únicos |
| Dictionary | `{"nombre": "Ana"}` | Pares clave : valor |

---

# Cheat Sheet

```python
# Crear
usuario = {
    "nombre": "Ana",
    "edad": 25
}

# Acceder
usuario["nombre"]
usuario.get("nombre")

# Cantidad
len(usuario)

# Buscar clave
"nombre" in usuario

# Agregar
usuario["pais"] = "México"

# Modificar
usuario["edad"] = 26
usuario.update({"edad": 27})

# Eliminar
usuario.pop("pais")
del usuario["edad"]

# Claves
usuario.keys()

# Valores
usuario.values()

# Claves y valores
usuario.items()

# Recorrer
for clave, valor in usuario.items():
    print(clave, valor)

# Copiar
copia = usuario.copy()
```

---

# 🟡 Ejercicio final — Sistema de inventario

Tienes el siguiente inventario:

```python
productos = {
    "laptop": {
        "precio": 15000,
        "stock": 5
    },
    "mouse": {
        "precio": 500,
        "stock": 12
    },
    "teclado": {
        "precio": 900,
        "stock": 8
    },
    "monitor": {
        "precio": 4200,
        "stock": 3
    },
    "audifonos": {
        "precio": 1200,
        "stock": 0
    }
}
```

Crea un programa que:

1. Muestre cuántos productos diferentes existen usando `len()`.
2. Muestre el precio y stock de `"laptop"`.
3. Cambie el precio de `"mouse"` a `450`.
4. Agregue un nuevo producto:

```python
"webcam": {
    "precio": 800,
    "stock": 6
}
```

5. Recorra todos los productos usando `items()` y muestre su nombre, precio y stock.
6. Muestre únicamente los productos que tengan stock mayor a `0`.
7. Calcule el valor total del inventario:

```text
precio × stock
```

8. Encuentre el producto con el precio más alto **sin escribir manualmente cuál es**.
9. Elimine `"audifonos"` usando `pop()`.
10. Compruebe si `"monitor"` continúa en el inventario usando `in`.

## Output esperado aproximado

```text
Productos diferentes: 5

Laptop
Precio: 15000
Stock: 5

Inventario:

laptop - $15000 - stock: 5
mouse - $450 - stock: 12
teclado - $900 - stock: 8
monitor - $4200 - stock: 3
audifonos - $1200 - stock: 0
webcam - $800 - stock: 6

Productos disponibles:
laptop
mouse
teclado
monitor
webcam

Valor total del inventario: $105000

Producto más caro: laptop
Precio: $15000

¿Existe monitor?: True
```

> Nota: el valor total se calcula después de cambiar el precio del mouse y agregar la webcam, pero antes de eliminar audífonos.
