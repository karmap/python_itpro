# Proyecto final: Planificador de viajes con historial

Construye un cliente de consola que solicite planes de viaje a una API y permita consultar los planes guardados sin volver a llamar a la IA.

**Tecnologías:** Python, requests, FastAPI, Pydantic, OpenRouter, SQLite y SQLModel.

## 1. Funcionamiento

El cliente debe ofrecer estas opciones:

- **Crear un plan:** pedir destino, días e interés; enviar la consulta y mostrar el plan guardado con su ID.
- **Consultar el historial:** mostrar ID, destino, días e interés de cada registro; indicar si está vacío.
- **Ver un plan:** pedir un ID entero y mostrar su contenido completo.
- **Eliminar un plan:** pedir un ID entero y mostrar la confirmación de eliminación.
- **Salir.**

El menú se repite después de cada operación o error recuperable. Los mensajes y resultados se presentan en español.

La API valida la entrada, consulta OpenRouter, valida el resultado, lo guarda en SQLite y responde al cliente. Cada creación exitosa genera un registro nuevo. El cliente accede al historial mediante la API; listar, consultar y eliminar registros no realiza llamadas a OpenRouter.

## 2. Datos de entrada

| Campo | Tipo | Regla |
|---|---|---|
| `city` | Texto | Obligatorio, de 1 a 100 caracteres después de retirar espacios exteriores. |
| `days` | Entero | Obligatorio, entre 1 y 14. |
| `interest` | Texto | Obligatorio: `comida`, `historia`, `naturaleza` o `vida nocturna`. |

La API rechaza campos adicionales. El cliente debe manejar entradas numéricas inválidas y opciones desconocidas sin terminar el programa.

## 3. Resultado de IA

Solicita recomendaciones acordes con los tres datos de entrada. OpenRouter debe devolver un objeto JSON con:

| Campo | Tipo | Regla |
|---|---|---|
| `summary` | Texto | Resumen no vacío ni de solo espacios. |
| `must_do` | Lista de textos | Al menos una actividad; ninguna vacía ni de solo espacios. |
| `budget_level` | Texto | `bajo`, `medio` o `alto`. |

Valida el resultado y rechaza campos adicionales antes de guardarlo. Una consulta fallida o un resultado inválido no crea registros. El presupuesto es orientativo; las recomendaciones no incluyen precios, horarios o disponibilidad verificados.

## 4. API

| Método y ruta | Respuesta exitosa |
|---|---|
| `GET /health` | 200: `{"status": "ok"}`. |
| `POST /plans` | 201: plan completo con ID, después de guardarlo. |
| `GET /history` | 200: lista con `id`, `city`, `days` e `interest`; `[]` si está vacía. |
| `GET /history/{id}` | 200: plan completo. |
| `DELETE /history/{id}` | 200: `{"mensaje": "Plan eliminado"}`, después de confirmar la eliminación. |

`POST /plans` recibe los tres campos de entrada. POST y GET por ID devuelven esta estructura; el ID real lo genera SQLite:

```json
{
    "id": 7,
    "city": "Tokio",
    "days": 4,
    "interest": "comida",
    "result": {
        "summary": "Cuatro días para explorar mercados y comida local.",
        "must_do": ["Visitar un mercado", "Probar ramen"],
        "budget_level": "medio"
    }
}
```

`result` debe ser un objeto JSON, no una cadena con JSON escapado. El cliente muestra el resumen, las actividades y el presupuesto de forma legible.

## 5. Persistencia y errores

Guarda en `plans.db`, junto al servidor, una tabla con `id`, `city`, `days`, `interest` y `result_json`. El último campo contiene el resultado validado como texto JSON. Los registros y las eliminaciones deben conservarse después de reiniciar la API.

| Situación | Código HTTP |
|---|---|
| Entrada inválida o ID de ruta no numérico | 422. |
| ID entero inexistente | 404. |
| Error de conexión o HTTP de OpenRouter, contenido ausente o resultado inválido | 502, sin guardar un registro. |
| Tiempo de espera agotado al consultar OpenRouter | 504, sin guardar un registro. |

El cliente muestra mensajes comprensibles ante errores de la API, conexión, espera o respuesta y regresa al menú. Nunca anuncia éxito si falló el guardado o la eliminación.

Configura `OPENROUTER_API_KEY` únicamente en el entorno del servidor. Usa un tiempo de espera en todas las peticiones: 30 segundos para OpenRouter, 60 para crear un plan desde el cliente y 10 para consultar o eliminar registros.

## 6. Archivos y ejecución

Organiza el proyecto en `client.py`, `main.py`, `models.py`, `ai.py` y `database.py`. Incluye `requirements.txt`, `.gitignore` e instrucciones breves de ejecución. Excluye de Git el entorno virtual, la base de datos y los archivos con claves.

Con el entorno virtual activado, instala las dependencias:

```bash
python -m pip install -r requirements.txt
```

Configura la clave e inicia el servidor con `fastapi dev main.py`. En otra terminal con el entorno activado, ejecuta `python client.py`. Usa `http://localhost:8000` como URL base; las rutas pueden probarse en `/docs`.

## 7. Criterios de entrega

- Crear, listar, consultar por ID y eliminar planes desde la consola.
- Recuperar un plan después de reiniciar la API y comprobar que una eliminación también persiste.
- Consultar el historial cuando OpenRouter no esté disponible.
- Rechazar entradas y resultados inválidos sin crear registros adicionales.
- Manejar IDs inexistentes y errores de conexión sin cerrar el cliente.

Entrega el código completo, las dependencias y las instrucciones para ejecutar y comprobar estas operaciones.

---

[Índice del curso](README.md)
