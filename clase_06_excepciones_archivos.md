# Clase 06: Excepciones y archivos

**Duración:** 1 a 2 horas

> Los bloques de desarrollo se leen en orden dentro de cada ejemplo. Las plantillas con `...` se completan en las actividades; las cheat sheets reúnen operaciones independientes. Los programas completos incluyen sus imports.

## `try` / `except`

Una **excepción** señala que una operación no pudo completarse. Python interrumpe el bloque actual y busca un `except` compatible; si no lo encuentra, el programa termina con un mensaje de error. `try` contiene la operación que puede fallar y `except` indica cómo responder a un tipo concreto de error.

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

### `else` y `finally`

`else` se ejecuta si el bloque `try` termina sin excepción. `finally` se ejecuta al salir, también cuando hubo un error; sirve para tareas de cierre. Un `except` no vuelve a ejecutar la instrucción que falló.

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

### `raise`

`raise` genera una excepción de forma explícita. Aquí rechazamos un precio negativo en lugar de calcular un descuento sobre un dato inválido.

```python
def calcular_descuento(precio):
    if precio < 0:
        raise ValueError("El precio no puede ser negativo")
    return precio * 0.90
```

---

### Leer el error capturado

`as error` permite consultar el error. Para responder igual a varios tipos, se escriben en una tupla:

```python
try:
    numero = int("abc")
    resultado = 100 / numero
except (ValueError, ZeroDivisionError) as error:
    print("No se pudo calcular:", error)
```

Captura los tipos que sabes manejar. Si tienes varios `except`, coloca los más específicos primero.

## Archivos

Crea primero `datos.txt` con un par de líneas de texto en la carpeta de trabajo. La forma recomendada con `open()` es usar `with`.

`open()` devuelve un objeto que representa el archivo abierto. `as archivo` le asigna un nombre y `with` lo cierra automáticamente al salir del bloque, incluso si ocurre una excepción. `encoding="utf-8"` indica cómo convertir entre texto y los bytes del archivo para conservar acentos y otros caracteres. Si omites el modo, se usa lectura (`"r"`).

```python
with open("datos.txt", encoding="utf-8") as archivo:
    contenido = archivo.read()
```

| Modo | Significado |
|---|---|
| `r` | Leer |
| `w` | Escribir / reemplazar |
| `a` | Agregar al final |
| `x` | Crear archivo nuevo |

### Leer

```python
with open("datos.txt", encoding="utf-8") as archivo:
    contenido = archivo.read()
```

Procesar línea por línea:

```python
with open("datos.txt", encoding="utf-8") as archivo:
    for linea in archivo:
        print(linea.rstrip("\n"))
```

### Escribir

`write()` escribe el texto recibido y no agrega un salto de línea automáticamente. `"\n"` representa ese salto. En el ejemplo anterior, `rstrip("\n")` retira los saltos del final de la línea; `print()` agrega uno propio.

```python
with open("resultado.txt", "w", encoding="utf-8") as archivo:
    archivo.write("Hola Python")
```

Agregar contenido:

```python
with open("resultado.txt", "a", encoding="utf-8") as archivo:
    archivo.write("\nNuevo registro")
```

---

## Excepciones y archivos

```python
try:
    with open("ventas.txt", encoding="utf-8") as archivo:
        contenido = archivo.read()
except FileNotFoundError:
    print("El archivo no existe")
```

---

## Pathlib

`pathlib` permite representar rutas y trabajar con archivos. `from pathlib import Path` importa `Path`, una herramienta incluida en Python; veremos los módulos en detalle en la clase 07. `Path("datos.txt")` crea un objeto de ruta y sus métodos, como `exists()` y `read_text()`, operan sobre esa ruta.

```python
from pathlib import Path

archivo = Path("datos.txt")

if archivo.exists():
    contenido = archivo.read_text(encoding="utf-8")
    print(contenido)
else:
    archivo.write_text("Hola Python", encoding="utf-8")
```

### Rutas

Con objetos `Path`, `/` une partes de una ruta; aquí no divide números. `mkdir()` crea la carpeta y `exist_ok=True` evita un error si esa carpeta ya existe. Construir la ruta no crea el archivo ni la carpeta por sí solo.

```python
from pathlib import Path

carpeta = Path("data")
archivo = carpeta / "ventas.txt"

carpeta.mkdir(exist_ok=True)
```

### Pathlib y excepciones

```python
from pathlib import Path

archivo = Path("ventas.txt")

try:
    contenido = archivo.read_text(encoding="utf-8")
except FileNotFoundError:
    print("No se encontró el archivo")
```

---

## Cheat sheet

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

with open("data.txt", encoding="utf-8") as file:
    content = file.read()

with open("data.txt", "w", encoding="utf-8") as file:
    file.write("Hello")

from pathlib import Path

file = Path("data.txt")
file.exists()
file.read_text(encoding="utf-8")
file.write_text("Hello", encoding="utf-8")

Path("data").mkdir(exist_ok=True)
```

---

## Cadenas para procesar archivos

Usaremos `strip()` para retirar espacios o saltos de línea y `linea.split(",")` para obtener una lista de campos. Desempaquetamos los tres valores como aprendimos con tuplas. Una *f-string* empieza con `f` e inserta expresiones entre `{}`:

```python
linea = "Mouse,450,5\n"
producto, precio_texto, cantidad_texto = linea.strip().split(",")
precio = int(precio_texto)
cantidad = int(cantidad_texto)
venta = precio * cantidad
print(f"{producto} - ${venta}")
```

Para juntar líneas de un reporte usamos `"\n".join(lineas)`: une una lista de cadenas usando un salto de línea como separador. Añade un salto final si quieres que el archivo termine en una línea completa. Captura `ValueError` por cada línea para continuar con las demás; no envuelvas todo el recorrido en un único `try`.

El archivo del ejercicio usa filas con tres campos separados por comas, una forma sencilla de CSV. `split(",")` es suficiente para estos datos porque ningún campo contiene comas internas. Si faltan o sobran campos, el desempaquetado lanza `ValueError`, igual que una conversión numérica inválida.

Las rutas relativas parten de la carpeta de trabajo de la terminal, no necesariamente de la carpeta del archivo Python.

---

## Ejercicio final: Procesador de ventas

Crea un archivo `ventas.txt` en la carpeta desde la que ejecutarás el programa y pega estos datos:

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
    ...  # Completa esta función durante la actividad.
```

Debe utilizar `Path` y manejar `FileNotFoundError`.

2. Divida cada línea utilizando:

```python
linea.strip().split(",")
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

### Práctica opcional

Crea `output/` y guarda el reporte en `output/reporte.txt`:

```python
from pathlib import Path

output = Path("output")
output.mkdir(exist_ok=True)

reporte = output / "reporte.txt"
```

El programa debe combinar:

```text
Funciones
Listas
Cadenas
Excepciones
Archivos
pathlib
```

---

[Índice del curso](README.md)
