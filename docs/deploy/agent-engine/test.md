# Probar agentes desplegados en Agent Engine

Estas instrucciones explican cómo probar un agente ADK desplegado en el
entorno de ejecución de
[Agent Engine](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/overview).
Antes de usar estas instrucciones, necesitas haber completado el despliegue de
tu agente en el entorno de ejecución de Agent Engine usando uno de los
[métodos disponibles](/adk-docs/deploy/agent-engine/). Esta guía te muestra
cómo ver, interactuar y probar tu agente desplegado a través de Google Cloud
Console, e interactuar con el agente usando llamadas a la API REST o el SDK de
Vertex AI para Python.

## Ver el agente desplegado en Cloud Console

Para ver tu agente desplegado en Cloud Console:

-   Navega a la página de Agent Engine en Google Cloud Console:
    [https://console.cloud.google.com/vertex-ai/agents/agent-engines](https://console.cloud.google.com/vertex-ai/agents/agent-engines)

Esta página enumera todos los agentes desplegados en tu proyecto de Google Cloud
actualmente seleccionado. Si no ves tu agente en la lista, asegúrate de tener tu
proyecto objetivo seleccionado en Google Cloud Console. Para más información sobre
cómo seleccionar un proyecto de Google Cloud existente, consulta
[Crear y gestionar proyectos](https://cloud.google.com/resource-manager/docs/creating-managing-projects#identifying_projects).

## Encontrar información del proyecto de Google Cloud

Necesitas la dirección y la identificación de recursos para tu proyecto (`PROJECT_ID`,
`LOCATION_ID`, `RESOURCE_ID`) para poder probar tu despliegue. Puedes usar Cloud
Console o la herramienta de línea de comandos `gcloud` para encontrar esta información.

??? note "Clave API del modo express de Vertex AI"
    Si estás usando el modo express de Vertex AI, puedes omitir este paso y usar tu clave API.

Para encontrar la información de tu proyecto con Google Cloud Console:

1.  En Google Cloud Console, navega a la página de Agent Engine:
    [https://console.cloud.google.com/vertex-ai/agents/agent-engines](https://console.cloud.google.com/vertex-ai/agents/agent-engines)

1.  En la parte superior de la página, selecciona **API URLs**, y luego copia la
    cadena de **Query URL** para tu agente desplegado, que debería estar en este formato:

        https://$(LOCATION_ID)-aiplatform.googleapis.com/v1/projects/$(PROJECT_ID)/locations/$(LOCATION_ID)/reasoningEngines/$(RESOURCE_ID):query

Para encontrar la información de tu proyecto con la herramienta de línea de comandos `gcloud`:

1.  En tu entorno de desarrollo, asegúrate de estar autenticado en
    Google Cloud y ejecuta el siguiente comando para listar tu proyecto:

    ```shell
    gcloud projects list
    ```

1.  Con el Project ID que usaste para el despliegue, ejecuta este comando para obtener
    los detalles adicionales:

    ```shell
    gcloud asset search-all-resources \
        --scope=projects/$(PROJECT_ID) \
        --asset-types='aiplatform.googleapis.com/ReasoningEngine' \
        --format="table(name,assetType,location,reasoning_engine_id)"
    ```

## Probar usando llamadas REST

Una forma simple de interactuar con tu agente desplegado en Agent Engine es usar llamadas
REST con la herramienta `curl`. Esta sección describe cómo verificar tu conexión al agente
y también cómo probar el procesamiento de una solicitud por parte del agente desplegado.

### Verificar la conexión al agente

Puedes verificar tu conexión al agente en ejecución usando la **Query URL**
disponible en la sección de Agent Engine de Cloud Console. Esta verificación no
ejecuta el agente desplegado, pero devuelve información sobre el agente.

Para enviar una llamada REST y obtener una respuesta del agente desplegado:

-   En una ventana de terminal de tu entorno de desarrollo, construye una solicitud
    y ejecútala:

    === "Proyecto de Google Cloud"

        ```shell
        curl -X GET \
            -H "Authorization: Bearer $(gcloud auth print-access-token)" \
            "https://$(LOCATION_ID)-aiplatform.googleapis.com/v1/projects/$(PROJECT_ID)/locations/$(LOCATION_ID)/reasoningEngines"
        ```

    === "Modo express de Vertex AI"

        ```shell
        curl -X GET \
            -H "x-goog-api-key:YOUR-EXPRESS-MODE-API-KEY" \
            "https://aiplatform.googleapis.com/v1/reasoningEngines"
        ```

Si tu despliegue fue exitoso, esta solicitud responde con una lista de solicitudes
válidas y formatos de datos esperados.

!!! tip "Eliminar el parámetro `:query` para la URL de conexión"
    Si usas la **Query URL** disponible en la sección de Agent Engine de Cloud
    Console, asegúrate de eliminar el parámetro `:query` del final de la dirección.

!!! tip "Acceso para conexiones de agente"
    Esta prueba de conexión requiere que el usuario que realiza la llamada tenga un token de acceso válido para el
    agente desplegado. Al probar desde otros entornos, asegúrate de que el usuario que realiza la llamada
    tenga acceso para conectarse al agente en tu proyecto de Google Cloud.

### Enviar una solicitud al agente

Al obtener respuestas de tu proyecto de agente, primero debes crear una
sesión, recibir un Session ID y luego enviar tus solicitudes usando ese Session
ID. Este proceso se describe en las siguientes instrucciones.

Para probar la interacción con el agente desplegado mediante REST:

1.  En una ventana de terminal de tu entorno de desarrollo, crea una sesión
    construyendo una solicitud usando esta plantilla:

    === "Proyecto de Google Cloud"

        ```shell
        curl \
            -H "Authorization: Bearer $(gcloud auth print-access-token)" \
            -H "Content-Type: application/json" \
            https://$(LOCATION_ID)-aiplatform.googleapis.com/v1/projects/$(PROJECT_ID)/locations/$(LOCATION_ID)/reasoningEngines/$(RESOURCE_ID):query \
            -d '{"class_method": "async_create_session", "input": {"user_id": "u_123"},}'
        ```

    === "Modo express de Vertex AI"

        ```shell
        curl \
            -H "x-goog-api-key:YOUR-EXPRESS-MODE-API-KEY" \
            -H "Content-Type: application/json" \
            https://aiplatform.googleapis.com/v1/reasoningEngines/$(RESOURCE_ID):query \
            -d '{"class_method": "async_create_session", "input": {"user_id": "u_123"},}'
        ```

1.  En la respuesta del comando anterior, extrae el **Session ID** creado
    del campo **id**:

    ```json
    {
        "output": {
            "userId": "u_123",
            "lastUpdateTime": 1757690426.337745,
            "state": {},
            "id": "4857885913439920384", # Session ID
            "appName": "9888888855577777776",
            "events": []
        }
    }
    ```

1.  En una ventana de terminal de tu entorno de desarrollo, envía un mensaje a
    tu agente construyendo una solicitud usando esta plantilla y el Session ID
    creado en el paso anterior:

    === "Proyecto de Google Cloud"

        ```shell
        curl \
        -H "Authorization: Bearer $(gcloud auth print-access-token)" \
        -H "Content-Type: application/json" \
        https://$(LOCATION_ID)-aiplatform.googleapis.com/v1/projects/$(PROJECT_ID)/locations/$(LOCATION_ID)/reasoningEngines/$(RESOURCE_ID):query?alt=sse -d '{
        "class_method": "async_stream_query",
        "input": {
            "user_id": "u_123",
            "session_id": "4857885913439920384",
            "message": "Hey whats the weather in new york today?",
        }
        }'
        ```

    === "Modo express de Vertex AI"

        ```shell
        curl \
        -H "x-goog-api-key:YOUR-EXPRESS-MODE-API-KEY" \
        -H "Content-Type: application/json" \
        https://aiplatform.googleapis.com/v1/reasoningEngines/$(RESOURCE_ID):query?alt=sse -d '{
        "class_method": "async_stream_query",
        "input": {
            "user_id": "u_123",
            "session_id": "4857885913439920384",
            "message": "Hey whats the weather in new york today?",
        }
        }'
        ```

Esta solicitud debería generar una respuesta de tu código de agente desplegado en formato
JSON. Para más información sobre cómo interactuar con un agente ADK desplegado en
Agent Engine usando llamadas REST, consulta
[Gestionar agentes desplegados](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/manage/overview#console)
y
[Usar un agente de Agent Development Kit](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/use/adk)
en la documentación de Agent Engine.

## Probar usando Python

Puedes usar código Python para realizar pruebas más sofisticadas y repetibles de tu
agente desplegado en Agent Engine. Estas instrucciones describen cómo crear
una sesión con el agente desplegado, y luego enviar una solicitud al agente para
su procesamiento.

### Crear una sesión remota

Usa el objeto `remote_app` para crear una conexión a un agente remoto desplegado:

```py
# Si estás en un script nuevo o usaste el CLI de ADK para desplegar, puedes conectarte así:
# remote_app = agent_engines.get("your-agent-resource-name")
remote_session = await remote_app.async_create_session(user_id="u_456")
print(remote_session)
```

Salida esperada para `create_session` (remoto):

```console
{'events': [],
'user_id': 'u_456',
'state': {},
'id': '7543472750996750336',
'app_name': '7917477678498709504',
'last_update_time': 1743683353.030133}
```

El valor `id` es el ID de sesión, y `app_name` es el ID de recurso del
agente desplegado en Agent Engine.

#### Enviar consultas a tu agente remoto

```py
async for event in remote_app.async_stream_query(
    user_id="u_456",
    session_id=remote_session["id"],
    message="whats the weather in new york",
):
    print(event)
```

Salida esperada para `async_stream_query` (remoto):

```console
{'parts': [{'function_call': {'id': 'af-f1906423-a531-4ecf-a1ef-723b05e85321', 'args': {'city': 'new york'}, 'name': 'get_weather'}}], 'role': 'model'}
{'parts': [{'function_response': {'id': 'af-f1906423-a531-4ecf-a1ef-723b05e85321', 'name': 'get_weather', 'response': {'status': 'success', 'report': 'The weather in New York is sunny with a temperature of 25 degrees Celsius (41 degrees Fahrenheit).'}}}], 'role': 'user'}
{'parts': [{'text': 'The weather in New York is sunny with a temperature of 25 degrees Celsius (41 degrees Fahrenheit).'}], 'role': 'model'}
```

Para más información sobre cómo interactuar con un agente ADK desplegado en
Agent Engine, consulta
[Gestionar agentes desplegados](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/manage/overview)
y
[Usar un agente de Agent Development Kit](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/use/adk)
en la documentación de Agent Engine.

### Enviar consultas multimodales

Para enviar consultas multimodales (por ejemplo, incluyendo imágenes) a tu agente, puedes construir el parámetro `message` de `async_stream_query` con una lista de objetos `types.Part`. Cada parte puede ser texto o una imagen.

Para incluir una imagen, puedes usar `types.Part.from_uri`, proporcionando un URI de Google Cloud Storage (GCS) para la imagen.

```python
from google.genai import types

image_part = types.Part.from_uri(
    file_uri="gs://cloud-samples-data/generative-ai/image/scones.jpg",
    mime_type="image/jpeg",
)
text_part = types.Part.from_text(
    text="What is in this image?",
)

async for event in remote_app.async_stream_query(
    user_id="u_456",
    session_id=remote_session["id"],
    message=[text_part, image_part],
):
    print(event)
```

!!!note
    Aunque la comunicación subyacente con el modelo puede involucrar codificación Base64
    para las imágenes, el método recomendado y soportado para enviar datos de imagen
    a un agente desplegado en Agent Engine es proporcionando un URI de GCS.

## Limpiar despliegues

Si has realizado despliegues como pruebas, es una buena práctica limpiar
tus recursos en la nube después de haber terminado. Puedes eliminar la instancia de Agent
Engine desplegada para evitar cargos inesperados en tu cuenta de Google Cloud.

```python
remote_app.delete(force=True)
```

El parámetro `force=True` también elimina cualquier recurso hijo que se haya generado
desde el agente desplegado, como sesiones. También puedes eliminar tu agente desplegado
a través de la
[interfaz de usuario de Agent Engine](https://console.cloud.google.com/vertex-ai/agents/agent-engines)
en Google Cloud.