# Clase 3 — HTTP, APIs y AI con `requests`

**Duración:** 2 horas  
**Proyecto final:** 🤖 AI Review Analyzer

## Objetivos

- Entender el flujo básico de HTTP.
- Consumir una API desde Python.
- Utilizar `GET` y `POST`.
- Trabajar con status codes, headers, query parameters y body.
- Convertir respuestas JSON a objetos de Python.
- Manejar errores de red con `requests`.
- Utilizar una API key.
- Enviar texto a una API de inteligencia artificial.
- Convertir texto no estructurado en datos estructurados.
- Construir un analizador inteligente de reseñas.

---

# 1. ¿Qué es una API?

Hasta ahora:

```text
Python
  ↓
productos.json
```

Ahora:

```text
Python
  │
  │ HTTP Request
  ▼
Internet
  │
  ▼
API
  │
  │ HTTP Response
  ▼
Python
```

Una **API** define una forma para que diferentes programas puedan comunicarse.

```text
GET /users
```

Respuesta:

```json
[
    {
        "id": 1,
        "name": "Ana"
    }
]
```

Conexión con la clase anterior:

```text
API → JSON → Python
```

---

# 2. HTTP Request / Response

Una petición HTTP normalmente contiene:

```text
METHOD
URL
HEADERS
QUERY PARAMETERS
BODY
```

## Métodos principales

```text
GET       Obtener información
POST      Crear o enviar información
PUT       Reemplazar
PATCH     Modificar
DELETE    Eliminar
```

Hoy nos concentraremos en `GET` y `POST`.

## Status codes

```text
200   OK
201   Created

400   Bad Request
401   Unauthorized
403   Forbidden
404   Not Found

500   Internal Server Error
```

Regla rápida:

```text
2xx → funcionó
4xx → problema con la petición
5xx → problema del servidor
```

---

# 3. Primer request con Python

Dentro del entorno virtual:

```bash
pip install requests
```

```python
import requests

url = "https://jsonplaceholder.typicode.com/users"

response = requests.get(url)

print(response)
print(response.status_code)
print(response.text)
```

Convertir JSON recibido a Python:

```python
usuarios = response.json()

print(type(usuarios))
print(type(usuarios[0]))
```

Resultado:

```text
<class 'list'>
<class 'dict'>
```

Ahora podemos usar todo lo aprendido:

```python
for usuario in usuarios:
    print(usuario["name"])
```

Flujo:

```text
Internet
   ↓
HTTP
   ↓
JSON
   ↓
response.json()
   ↓
list[dict]
```

---

# 4. Query Parameters

Ejemplo:

```text
https://jsonplaceholder.typicode.com/comments?postId=1
```

Con `requests`:

```python
params = {
    "postId": 1
}

response = requests.get(
    "https://jsonplaceholder.typicode.com/comments",
    params=params
)

print(response.url)

comentarios = response.json()

for comentario in comentarios:
    print(comentario["email"])
```

---

# 5. Headers

Los headers contienen información adicional sobre la petición.

```python
headers = {
    "Authorization": "Bearer API_KEY"
}
```

```python
response = requests.get(
    url,
    headers=headers
)
```

Un uso muy común es enviar una **API key**:

```text
API KEY
   ↓
identifica / autoriza nuestra aplicación
```

> No subas API keys reales a GitHub.

Más adelante veremos variables de entorno.

---

# 6. POST

Con `GET` normalmente obtenemos información:

```text
Python ← información ← API
```

Con `POST` enviamos información:

```text
Python → información → API
```

Ejemplo:

```python
import requests

nuevo_post = {
    "title": "Aprendiendo APIs",
    "body": "Primer POST desde Python",
    "userId": 1
}

response = requests.post(
    "https://jsonplaceholder.typicode.com/posts",
    json=nuevo_post
)

print(response.status_code)
print(response.json())
```

`requests` serializa el `dict` para enviarlo como JSON:

```text
dict
 ↓
requests
 ↓
JSON
 ↓
HTTP POST
```

> JSONPlaceholder es una API de prueba. Simula operaciones de escritura, pero los cambios no se persisten realmente.

---

# 7. Manejo de errores

Nunca debemos asumir que Internet o el servidor siempre funcionarán.

```python
import requests

try:
    response = requests.get(
        "https://jsonplaceholder.typicode.com/users",
        timeout=10
    )

    response.raise_for_status()

    usuarios = response.json()

except requests.RequestException as error:
    print("Error:", error)
```

## `timeout`

```python
timeout=10
```

Evita esperar indefinidamente.

## `raise_for_status()`

Hace que respuestas HTTP de error generen una excepción.

```text
HTTP error
    ↓
Exception
    ↓
try / except
```

---

# Actividad 1 — API Client

Crea:

```python
def obtener_usuarios() -> list[dict]:
    ...
```

Debe:

1. Hacer un `GET` a `https://jsonplaceholder.typicode.com/users`.
2. Utilizar `timeout`.
3. Utilizar `raise_for_status()`.
4. Convertir la respuesta a Python.
5. Retornar los usuarios.
6. Manejar `RequestException`.

Después muestra para cada usuario:

```text
Nombre
Email
Ciudad
```

La ciudad está en:

```python
usuario["address"]["city"]
```

---

# 8. Una API de Inteligencia Artificial

Una API de IA utiliza los mismos conceptos:

```text
Python
   ↓
HTTP POST
   ↓
JSON
   ↓
AI API
```

Podemos enviar:

```text
Analiza esta reseña:

"Me cobraron dos veces y nadie responde."
```

y pedir una respuesta estructurada.

---

# 🤖 Proyecto — AI Review Analyzer

Tenemos:

```json
[
    {
        "id": 1,
        "comentario": "Excelente producto, llegó rapidísimo."
    },
    {
        "id": 2,
        "comentario": "El producto funciona pero tardó muchísimo en llegar."
    },
    {
        "id": 3,
        "comentario": "Me cobraron dos veces y nadie responde."
    }
]
```

Queremos transformar:

```text
"Me cobraron dos veces y nadie responde."
```

en:

```json
{
    "sentimiento": "negativo",
    "categoria": "cobro",
    "urgente": true,
    "resumen": "Posible cobro duplicado sin respuesta de soporte."
}
```

## ¿Por qué IA?

Podríamos hacer:

```python
if "cobro" in comentario:
    categoria = "cobro"
```

Pero:

```text
"Me quitaron el dinero dos veces."
```

expresa un problema similar sin utilizar la palabra `"cobro"`.

La IA puede interpretar el significado del texto.

---

# 9. OpenRouter

Endpoint:

```text
https://openrouter.ai/api/v1/chat/completions
```

Modelo/router gratuito:

```text
openrouter/free
```

Necesitamos una API key.

> Los modelos gratuitos y sus límites pueden cambiar. Verifica el acceso antes de la clase.

---

# 10. Primer request a una IA

```python
import requests

API_KEY = "TU_API_KEY"

url = "https://openrouter.ai/api/v1/chat/completions"

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

data = {
    "model": "openrouter/free",
    "messages": [
        {
            "role": "user",
            "content": "Explica Python en una oración."
        }
    ]
}

response = requests.post(
    url,
    headers=headers,
    json=data,
    timeout=30
)

response.raise_for_status()

resultado = response.json()

print(resultado)
```

Primero inspecciona el JSON completo.

Después:

```python
contenido = resultado["choices"][0]["message"]["content"]

print(contenido)
```

Estructura:

```text
dict
 ↓
"choices"
 ↓
list
 ↓
[0]
 ↓
dict
 ↓
"message"
 ↓
dict
 ↓
"content"
```

---

# 11. Prompt estructurado

Prompt poco específico:

```text
¿Qué opinas de esta reseña?
```

Mejor:

```text
Analiza la siguiente reseña.

Determina:
- sentimiento
- categoria
- urgente
- resumen

Sentimiento: positivo, neutral, negativo o mixto.
Categoria: producto, envio, soporte, cobro u otro.
Urgente: true o false.

Devuelve únicamente JSON válido.
No utilices bloques Markdown.

Reseña:
"Me cobraron dos veces y nadie responde."
```

Respuesta esperada:

```json
{
    "sentimiento": "negativo",
    "categoria": "cobro",
    "urgente": true,
    "resumen": "Posible cobro duplicado sin respuesta de soporte."
}
```

Idea clave:

```text
TEXTO NO ESTRUCTURADO
          ↓
          IA
          ↓
DATOS ESTRUCTURADOS
```

---

# 12. Convertir la respuesta a Python

```python
contenido = resultado["choices"][0]["message"]["content"]

print(type(contenido))
```

Resultado:

```text
<class 'str'>
```

Convertir el JSON generado a Python:

```python
import json

analisis = json.loads(contenido)

print(type(analisis))
```

Resultado:

```text
<class 'dict'>
```

Ahora:

```python
print(analisis["sentimiento"])
print(analisis["categoria"])
print(analisis["urgente"])
print(analisis["resumen"])
```

> Una IA puede producir una respuesta inesperada. Por eso también debemos manejar errores al ejecutar `json.loads()`.

---

# 13. Función `analizar_reseña`

```python
import json
import requests


def analizar_reseña(
    comentario: str,
    api_key: str
) -> dict | None:

    url = "https://openrouter.ai/api/v1/chat/completions"

    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json"
    }

    prompt = f"""
Analiza la siguiente reseña.

Determina:
- sentimiento
- categoria
- urgente
- resumen

Sentimiento: positivo, neutral, negativo o mixto.
Categoria: producto, envio, soporte, cobro u otro.
Urgente: true o false.

Devuelve únicamente JSON válido.
No utilices bloques Markdown.

Reseña:
{comentario}
"""

    data = {
        "model": "openrouter/free",
        "messages": [
            {
                "role": "user",
                "content": prompt
            }
        ]
    }

    try:
        response = requests.post(
            url,
            headers=headers,
            json=data,
            timeout=30
        )

        response.raise_for_status()

        resultado = response.json()
        contenido = resultado["choices"][0]["message"]["content"]

        return json.loads(contenido)

    except requests.RequestException as error:
        print("Error HTTP:", error)

    except (json.JSONDecodeError, KeyError, IndexError, TypeError) as error:
        print("Respuesta inesperada de la IA:", error)

    return None
```

Uso:

```python
resultado = analizar_reseña(
    "Me cobraron dos veces y nadie responde.",
    API_KEY
)

if resultado is not None:
    print(resultado)
```

---

# Ejercicio final — AI Review Analyzer

Estructura:

```text
review_analyzer/
│
├── main.py
├── ai.py
├── reports.py
│
└── data/
    ├── reviews.json
    └── results.json
```

## `reviews.json`

```json
[
    {
        "id": 1,
        "comentario": "Excelente producto, llegó antes de lo esperado."
    },
    {
        "id": 2,
        "comentario": "El producto está bien pero tardó una semana."
    },
    {
        "id": 3,
        "comentario": "Me cobraron dos veces y soporte no responde."
    },
    {
        "id": 4,
        "comentario": "Llegó roto y la caja estaba completamente aplastada."
    }
]
```

## `ai.py`

Crea:

```python
def analizar_reseña(
    comentario: str,
    api_key: str
) -> dict | None:
    ...
```

Debe:

1. Enviar el comentario a la API.
2. Utilizar `POST`.
3. Enviar headers.
4. Enviar JSON.
5. Utilizar `timeout`.
6. Utilizar `raise_for_status()`.
7. Manejar errores HTTP.
8. Convertir la respuesta de la IA a `dict`.
9. Devolver `None` si ocurre un error.

Resultado esperado:

```python
{
    "sentimiento": "negativo",
    "categoria": "envio",
    "urgente": True,
    "resumen": "Producto recibido dañado."
}
```

---

# `main.py`

Carga:

```text
data/reviews.json
```

Para cada reseña:

```python
analisis = analizar_reseña(
    reseña["comentario"],
    API_KEY
)
```

Combina los datos originales con el análisis:

```python
{
    "id": 4,
    "comentario": "Llegó roto...",
    "sentimiento": "negativo",
    "categoria": "envio",
    "urgente": True,
    "resumen": "Producto recibido dañado."
}
```

Finalmente guarda los resultados en:

```text
data/results.json
```

---

# `reports.py`

Genera un reporte:

```text
========= AI REVIEW REPORT =========

Reseñas analizadas: 30

Sentimiento:

Positivas:  17
Neutrales:   4
Negativas:   7
Mixtas:      2

Problemas:

Envío:       5
Producto:    2
Soporte:     1
Cobro:       1

Casos urgentes: 3

====================================
```

Bonus: mostrar casos urgentes.

---

# Bonus — Modo interactivo

```python
while True:
    comentario = input("
Escribe una reseña: ")

    if comentario.lower() == "salir":
        break

    resultado = analizar_reseña(
        comentario,
        API_KEY
    )

    if resultado is None:
        continue

    print()
    print("Sentimiento:", resultado["sentimiento"])
    print("Categoría:", resultado["categoria"])
    print("Urgente:", resultado["urgente"])
    print("Resumen:", resultado["resumen"])
```

Ejemplo:

```text
Escribe una reseña:
> La laptop está increíble pero tardó casi tres semanas en llegar.

Sentimiento: mixto
Categoría: envio
Urgente: False
Resumen: Cliente satisfecho con el producto, pero inconforme con el tiempo de entrega.
```

---

# Flujo completo

```text
reviews.json
     ↓
json.load()
     ↓
list[dict]
     ↓
HTTP POST
     ↓
AI API
     ↓
JSON response
     ↓
response.json()
     ↓
contenido generado
     ↓
json.loads()
     ↓
dict
     ↓
results.json
     ↓
reporte
```

---

# Cheat Sheet

## GET

```python
response = requests.get(
    url,
    params=params,
    timeout=10
)
```

## POST

```python
response = requests.post(
    url,
    headers=headers,
    json=data,
    timeout=30
)
```

## Status

```python
response.status_code
```

## JSON → Python

```python
datos = response.json()
```

## Detectar errores HTTP

```python
response.raise_for_status()
```

## Manejar errores

```python
try:
    ...
except requests.RequestException as error:
    print(error)
```

## Header con API key

```python
headers = {
    "Authorization": f"Bearer {API_KEY}"
}
```

## JSON string → Python

```python
datos = json.loads(texto)
```

---

# Conceptos conectados

```text
venv
pip
requests
HTTP
GET
POST
status codes
headers
query parameters
JSON
dict
list
functions
type hints
exceptions
modules
AI
```

## Siguiente paso

Hasta ahora:

```text
Python → consume una API
```

Después:

```text
otros programas → consumen NUESTRA API
```

con **FastAPI**.
