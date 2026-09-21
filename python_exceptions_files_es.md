# Python: Exceptions + Files

## Try / Except

```python
try:
    numero = int("abc")
except ValueError:
    print("El valor no es un número válido")
```

### Diferentes errores

```python
try:
    numero = int(input("Número: "))
    resultado = 100 / numero
except ValueError:
    print("Debes escribir un número")
except ZeroDivisionError:
    print("No puedes dividir entre cero")
```

## Else y Finally

```python
try:
    numero = int("25")
except ValueError:
    print("Número inválido")
else:
    print("Número:", numero)
finally:
    print("Proceso terminado")
```

## Raise

```python
def calcular_descuento(precio):
    if precio < 0:
        raise ValueError("El precio no puede ser negativo")
    return precio * 0.90
```

---

# Files

La forma recomendada con `open()` es usar `with`:

```python
with open("datos.txt") as archivo:
    contenido = archivo.read()
```

| Modo | Significado |
|---|---|
| `r` | Read |
| `w` | Write / reemplazar |
| `a` | Append |
| `x` | Crear archivo nuevo |

## Leer

```python
with open("datos.txt") as archivo:
    contenido = archivo.read()
```

Procesar línea por línea:

```python
with open("datos.txt") as archivo:
    for linea in archivo:
        print(linea)
```

## Escribir

```python
with open("resultado.txt", "w") as archivo:
    archivo.write("Hola Python")
```

Agregar contenido:

```python
with open("resultado.txt", "a") as archivo:
    archivo.write("\nNuevo registro")
```

---

# Exceptions + Files

```python
try:
    with open("ventas.txt") as archivo:
        contenido = archivo.read()
except FileNotFoundError:
    print("El archivo no existe")
```

---

# Pathlib

`pathlib` ofrece una forma moderna de trabajar con archivos y rutas.

```python
from pathlib import Path

archivo = Path("datos.txt")

if archivo.exists():
    print("El archivo existe")

contenido = archivo.read_text()
archivo.write_text("Hola Python")
```

## Rutas

```python
from pathlib import Path

carpeta = Path("data")
archivo = carpeta / "ventas.txt"

carpeta.mkdir(exist_ok=True)
```

## Pathlib + Exceptions

```python
from pathlib import Path

archivo = Path("ventas.txt")

try:
    contenido = archivo.read_text()
except FileNotFoundError:
    print("No se encontró el archivo")
```

---

# Cheat Sheet

```python
try:
    ...
except ValueError:
    ...
else:
    ...
finally:
    ...

raise ValueError("Mensaje")

with open("data.txt") as file:
    content = file.read()

with open("data.txt", "w") as file:
    file.write("Hello")

from pathlib import Path

file = Path("data.txt")
file.exists()
file.read_text()
file.write_text("Hello")

Path("data").mkdir(exist_ok=True)
```

---

# 🟡 Ejercicio final — Procesador de ventas

Archivo `ventas.txt`:

```text
Laptop,15000,2
Mouse,450,5
Teclado,900,3
Monitor,4200,2
Webcam,800,4
Audifonos,abc,3
Laptop,15000,1
```

Cada línea contiene:

```text
producto,precio,cantidad
```

Crea un programa que:

1. Cree una función:

```python
def leer_ventas(path):
    # ...
```

Debe utilizar `Path` y manejar `FileNotFoundError`.

2. Divida cada línea utilizando:

```python
split(",")
```

3. Convierta precio y cantidad a números.

4. Maneje la línea inválida:

```text
Audifonos,abc,3
```

sin detener el programa:

```text
Venta inválida: Audifonos,abc,3
```

5. Calcule cada venta:

```text
precio × cantidad
```

6. Calcule el total:

```text
TOTAL: $61550
```

7. Genere `reporte.txt`:

```text
Laptop - $30000
Mouse - $2250
Teclado - $2700
Monitor - $8400
Webcam - $3200
Laptop - $15000

TOTAL: $61550
```

## Bonus

Crea `output/` y guarda el reporte en `output/reporte.txt`:

```python
from pathlib import Path

output = Path("output")
output.mkdir(exist_ok=True)

reporte = output / "reporte.txt"
```

El programa debe combinar:

```text
Functions
Lists
Strings
Exceptions
Files
Pathlib
```
