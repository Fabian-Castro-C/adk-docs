# Referencia de la API REST

Esta página proporciona una referencia para la API REST proporcionada por el servidor web de ADK.
Para obtener detalles sobre el uso de la API REST de ADK en la práctica, consulte
[Usar el Servidor API](/adk-docs/runtime/api-server/).

!!! tip
    Puede ver una referencia de API actualizada en un servidor web ADK en ejecución navegando
    a la ubicación `/docs`, por ejemplo en: `http://localhost:8000/docs`

## Endpoints

### `/run`

Este endpoint ejecuta una ejecución del agente. Recibe un payload JSON con los detalles de la ejecución y devuelve una lista de eventos generados durante la ejecución.

**Cuerpo de la Solicitud**

El cuerpo de la solicitud debe ser un objeto JSON con los siguientes campos:

- `app_name` (string, requerido): El nombre del agente a ejecutar.
- `user_id` (string, requerido): El ID del usuario.
- `session_id` (string, requerido): El ID de la sesión.
- `new_message` (Content, requerido): El nuevo mensaje a enviar al agente. Consulte la sección [Content](#content-object) para más detalles.
- `streaming` (boolean, opcional): Si se debe usar streaming. Por defecto es `false`.
- `state_delta` (object, opcional): Un delta del estado a aplicar antes de la ejecución.

**Cuerpo de la Respuesta**

El cuerpo de la respuesta es un array JSON de objetos [Event](#event-object).

### `/run_sse`

Este endpoint ejecuta una ejecución del agente usando Server-Sent Events (SSE) para respuestas de streaming. Recibe el mismo payload JSON que el endpoint `/run`.

**Cuerpo de la Solicitud**

El cuerpo de la solicitud es el mismo que para el endpoint `/run`.

**Cuerpo de la Respuesta**

La respuesta es un flujo de Server-Sent Events. Cada evento es un objeto JSON que representa un [Event](#event-object).

## Objetos

### Objeto `Content`

El objeto `Content` representa el contenido de un mensaje. Tiene la siguiente estructura:

```json
{
  "parts": [
    {
      "text": "..."
    }
  ],
  "role": "..."
}
```

- `parts`: Una lista de partes. Cada parte puede ser texto o una llamada a función.
- `role`: El rol del autor del mensaje (por ejemplo, "user", "model").

### Objeto `Event`

El objeto `Event` representa un evento que ocurrió durante una ejecución del agente. Tiene una estructura compleja con muchos campos opcionales. Los campos más importantes son:

- `id`: El ID del evento.
- `timestamp`: La marca de tiempo del evento.
- `author`: El autor del evento.
- `content`: El contenido del evento.