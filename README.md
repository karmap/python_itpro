# Python aplicado: APIs, IA y bases de datos

Curso en español para estudiantes que ya conocen los fundamentos generales de programación y quieren aprender Python mediante ejemplos pequeños, actividades y aplicaciones reales. La progresión conecta estructuras de datos, funciones y archivos con HTTP, APIs de IA, FastAPI y persistencia con SQLite.

Los materiales están pensados para demostraciones en vivo y práctica durante clases de aproximadamente 1 a 2 horas. Incluyen ejercicios y cheat sheets para consultar lo aprendido.

Las explicaciones introducen la sintaxis y los conceptos antes de usarlos en los programas completos. Los fragmentos que dependen de código previo lo indican; las plantillas con `...` forman parte de los ejercicios. Ejecuta los programas desde la carpeta indicada y sigue la numeración del índice.

## Requisitos

- Python 3.10 o superior para las anotaciones de tipos que usa el curso.
- Un editor y una terminal para ejecutar los ejemplos.
- Conocimientos generales de variables, condiciones, ciclos y funciones.
- Acceso a Internet y una API key de OpenRouter para las actividades de IA. La clave debe configurarse fuera del código y del repositorio.

La creación de entornos virtuales y la instalación de dependencias se trabajan en la clase 08. La edición revisada se comprobó con Python 3.12.14. Para instalar las versiones usadas en las verificaciones, activa un entorno virtual y ejecuta desde la raíz:

```bash
python -m pip install -r requirements.txt
```

No es necesario instalar todo desde la primera clase: cada documento indica las dependencias que utiliza. La clave de OpenRouter se configura como variable de entorno en la clase 08.

## Contenido

Este índice reúne los fundamentos de Python y su aplicación a programas con APIs, IA y persistencia. Las clases siguen el patrón `clase_NN_tema.md`, con numeración continua del 01 al 12. El proyecto final tiene su propio documento al final del índice.

### 1. [Listas](clase_01_listas.md)

Creación, acceso, modificación, recorridos, comprehensions, ordenamiento y copias de listas.

### 2. [Tuplas](clase_02_tuplas.md)

Colecciones inmutables, slicing, desempaquetado y análisis de temperaturas.

### 3. [Conjuntos (sets)](clase_03_conjuntos.md)

Elementos únicos, eliminación de duplicados y comparación de conjuntos de usuarios.

### 4. [Diccionarios](clase_04_diccionarios.md)

Pares clave y valor, estructuras anidadas y un ejercicio de inventario.

### 5. [Funciones y lambda](clase_05_funciones_lambda.md)

Parámetros, retornos, argumentos variables, alcance y funciones para procesar productos.

### 6. [Excepciones y archivos](clase_06_excepciones_archivos.md)

Manejo de errores, lectura y escritura de archivos, `pathlib` y un reporte de ventas.

### 7. [Anotaciones de tipos, utilidades y módulos](clase_07_tipos_utilidades_modulos.md)

Type hints, `None`, unpacking, `enumerate`, `zip`, argumentos e imports para organizar un programa.

### 8. [JSON y entornos virtuales](clase_08_json_entornos_virtuales.md)

Serialización, archivos JSON, `venv`, `pip` y persistencia de un inventario en varios módulos.

### 9. [HTTP, APIs e IA con requests](clase_09_http_apis_ia_requests.md)

Consumo de APIs, GET, POST, headers, errores y OpenRouter mediante un analizador de reseñas.

### 10. [Servidor HTTP desde cero](clase_10_servidor_http.md)

Servidor con `http.server`, rutas, status codes y respuestas JSON probadas con navegador, `curl` y `requests`.

### 11. [FastAPI y operaciones sobre productos](clase_11_fastapi.md)

Migración del servidor manual a FastAPI con parámetros, Pydantic, validación, errores, DELETE y PUT como bonus.

### 12. [Persistencia con SQLite y SQLModel](clase_12_sqlite_sqlmodel.md)

Modelos, engine, sesiones y operaciones para guardar, consultar y eliminar productos de forma persistente.

Esta clase es obligatoria antes del proyecto final con SQLite.

### 13. [Proyecto final: Planificador de viajes con historial](proyecto_final.md)

Cliente de consola que crea y consulta planes de viaje mediante FastAPI, OpenRouter y SQLite. Incluye datos de entrada y salida, rutas, errores y criterios de entrega.

## Progresión del curso

```text
Python
  ↓
Estructuras de datos
  ↓
Funciones y archivos
  ↓
Módulos
  ↓
JSON
  ↓
HTTP / requests
  ↓
APIs de IA
  ↓
Servidor HTTP
  ↓
FastAPI
  ↓
SQLite / SQLModel
  ↓
Proyecto final
```
