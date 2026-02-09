# Usar el Servidor API

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Antes de desplegar tu agente, debes probarlo para asegurarte de que funciona según
lo previsto. Usa el servidor API en ADK para exponer tus agentes a través de una API REST para
pruebas programáticas e integración.

![ADK API Server](../assets/adk-api-server.png)

## Iniciar el servidor API

Usa el siguiente comando para ejecutar tu agente en un servidor API de ADK:

=== "Python"

    ```shell
    adk api_server
    ```

=== "TypeScript"

    ```shell
    npx adk api_server
    ```

=== "Go"

    ```shell
    go run agent.go web api
    ```

=== "Java"

    Asegúrate de actualizar el número de puerto.
    === "Maven"
        Con Maven, compila y ejecuta el servidor web de ADK:
        ```console
        mvn compile exec:java \
         -Dexec.args="--adk.agents.source-dir=src/main/java/agents --server.port=8080"
        ```
    === "Gradle"
        Con Gradle, el archivo de construcción `build.gradle` o `build.gradle.kts` debe tener el siguiente plugin de Java en su sección de plugins:

        ```groovy
        plugins {
            id('java')
            // otros plugins
        }
        ```
        Luego, en otra parte del archivo de construcción, en el nivel superior, crea una nueva tarea:

        ```groovy
        tasks.register('runADKWebServer', JavaExec) {
            dependsOn classes
            classpath = sourceSets.main.runtimeClasspath
            mainClass = 'com.google.adk.web.AdkWebServer'
            args '--adk.agents.source-dir=src/main/java/agents', '--server.port=8080'
        }
        ```

        Finalmente, en la línea de comandos, ejecuta el siguiente comando:
        ```console
        gradle runADKWebServer
        ```


    En Java, tanto la Dev UI como el servidor API están empaquetados juntos.

Este comando lanzará un servidor web local, donde puedes ejecutar comandos cURL o
enviar peticiones API para probar tu agente. Por defecto, el servidor se ejecuta en
`http://localhost:8000`.

!!! tip "Uso Avanzado y Depuración"

    Para una referencia completa de todos los endpoints disponibles, formatos de petición/respuesta, y consejos para depuración (incluyendo cómo usar la documentación interactiva de la API), consulta la **Guía del Servidor API de ADK** a continuación.

## Probar localmente

Probar localmente implica lanzar un servidor web local, crear una sesión y
enviar consultas a tu agente. Primero, asegúrate de estar en el directorio de trabajo
correcto.

Para TypeScript, debes estar dentro del directorio del proyecto del agente.

```console
parent_folder/
└── my_sample_agent/  <-- Para TypeScript, ejecuta comandos desde aquí
    └── agent.py (o Agent.java o agent.ts)
```

**Lanzar el Servidor Local**

A continuación, lanza el servidor local usando los comandos listados anteriormente.

La salida debería aparecer similar a:

=== "Python"

    ```shell
    INFO:     Started server process [12345]
    INFO:     Waiting for application startup.
    INFO:     Application startup complete.
    INFO:     Uvicorn running on http://localhost:8000 (Press CTRL+C to quit)
    ```

=== "TypeScript"

    ```shell
    +-----------------------------------------------------------------------------+
    | ADK Web Server started                                                      |
    |                                                                             |
    | For local testing, access at http://localhost:8000.                         |
    +-----------------------------------------------------------------------------+
    ```

=== "Java"

    ```shell
    2025-05-13T23:32:08.972-06:00  INFO 37864 --- [ebServer.main()] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
    2025-05-13T23:32:08.980-06:00  INFO 37864 --- [ebServer.main()] com.google.adk.web.AdkWebServer          : Started AdkWebServer in 1.15 seconds (process running for 2.877)
    2025-05-13T23:32:08.981-06:00  INFO 37864 --- [ebServer.main()] com.google.adk.web.AdkWebServer          : AdkWebServer application started successfully.
    ```

Tu servidor ahora se está ejecutando localmente. Asegúrate de usar el **_número de puerto_** correcto en todos los comandos subsiguientes.

**Crear una nueva sesión**

Con el servidor API aún en ejecución, abre una nueva ventana o pestaña de terminal y crea
una nueva sesión con el agente usando:

```shell
curl -X POST http://localhost:8000/apps/my_sample_agent/users/u_123/sessions/s_123 \
  -H "Content-Type: application/json" \
  -d '{"key1": "value1", "key2": 42}'
```

Analicemos qué está sucediendo:

* `http://localhost:8000/apps/my_sample_agent/users/u_123/sessions/s_123`: Esto
  crea una nueva sesión para tu agente `my_sample_agent`, que es el nombre de
  la carpeta del agente, para un ID de usuario (`u_123`) y para un ID de sesión (`s_123`). Puedes
  reemplazar `my_sample_agent` con el nombre de la carpeta de tu agente. Puedes
  reemplazar `u_123` con un ID de usuario específico, y `s_123` con un ID de sesión
  específico.
* `{"key1": "value1", "key2": 42}`: Esto es opcional. Puedes usar
  esto para personalizar el estado (dict) preexistente del agente al crear la
  sesión.

Esto debería devolver la información de la sesión si se creó exitosamente. La
salida debería aparecer similar a:

```json
{"id":"s_123","appName":"my_sample_agent","userId":"u_123","state":{"key1":"value1","key2":42},"events":[],"lastUpdateTime":1743711430.022186}
```

!!! info

    No puedes crear múltiples sesiones con exactamente el mismo ID de usuario e
    ID de sesión. Si lo intentas, puedes ver una respuesta como:
    `{"detail":"Session already exists: s_123"}`. Para solucionar esto, puedes
    eliminar esa sesión (ej., `s_123`), o elegir un ID de sesión diferente.

**Enviar una consulta**

Hay dos formas de enviar consultas vía POST a tu agente, a través de las rutas `/run` o
`/run_sse`.

* `POST http://localhost:8000/run`: recopila todos los eventos como una lista y devuelve la
  lista de una vez. Adecuado para la mayoría de los usuarios (si no estás seguro, recomendamos
  usar este).
* `POST http://localhost:8000/run_sse`: devuelve como Server-Sent-Events, que es un
  flujo de objetos de eventos. Adecuado para aquellos que quieren ser notificados tan pronto como
  el evento esté disponible. Con `/run_sse`, también puedes establecer `streaming` en
  `true` para habilitar streaming a nivel de token.

**Usando `/run`**

```shell
curl -X POST http://localhost:8000/run \
-H "Content-Type: application/json" \
-d '{
"appName": "my_sample_agent",
"userId": "u_123",
"sessionId": "s_123",
"newMessage": {
    "role": "user",
    "parts": [{
    "text": "Hey whats the weather in new york today"
    }]
}
}'
```

En TypeScript, actualmente solo se admiten nombres de campo en `camelCase` (ej. `appName`, `userId`, `sessionId`, etc.).

Si usas `/run`, verás la salida completa de eventos al mismo tiempo, como una
lista, que debería aparecer similar a:

```json
[{"content":{"parts":[{"functionCall":{"id":"af-e75e946d-c02a-4aad-931e-49e4ab859838","args":{"city":"new york"},"name":"get_weather"}}],"role":"model"},"invocationId":"e-71353f1e-aea1-4821-aa4b-46874a766853","author":"weather_time_agent","actions":{"stateDelta":{},"artifactDelta":{},"requestedAuthConfigs":{}},"longRunningToolIds":[],"id":"2Btee6zW","timestamp":1743712220.385936},{"content":{"parts":[{"functionResponse":{"id":"af-e75e946d-c02a-4aad-931e-49e4ab859838","name":"get_weather","response":{"status":"success","report":"The weather in New York is sunny with a temperature of 25 degrees Celsius (41 degrees Fahrenheit)."}}}],"role":"user"},"invocationId":"e-71353f1e-aea1-4821-aa4b-46874a766853","author":"weather_time_agent","actions":{"stateDelta":{},"artifactDelta":{},"requestedAuthConfigs":{}},"id":"PmWibL2m","timestamp":1743712221.895042},{"content":{"parts":[{"text":"OK. The weather in New York is sunny with a temperature of 25 degrees Celsius (41 degrees Fahrenheit).\n"}],"role":"model"},"invocationId":"e-71353f1e-aea1-4821-aa4b-46874a766853","author":"weather_time_agent","actions":{"stateDelta":{},"artifactDelta":{},"requestedAuthConfigs":{}},"id":"sYT42eVC","timestamp":1743712221.899018}]
```

**Usando `/run_sse`**

```shell
curl -X POST http://localhost:8000/run_sse \
-H "Content-Type: application/json" \
-d '{
"appName": "my_sample_agent",
"userId": "u_123",
"sessionId": "s_123",
"newMessage": {
    "role": "user",
    "parts": [{
    "text": "Hey whats the weather in new york today"
    }]
},
"streaming": false
}'
```

Puedes establecer `streaming` en `true` para habilitar streaming a nivel de token, lo que significa
que la respuesta será devuelta en múltiples fragmentos y la salida debería
aparecer similar a:


```shell
data: {"content":{"parts":[{"functionCall":{"id":"af-f83f8af9-f732-46b6-8cb5-7b5b73bbf13d","args":{"city":"new york"},"name":"get_weather"}}],"role":"model"},"invocationId":"e-3f6d7765-5287-419e-9991-5fffa1a75565","author":"weather_time_agent","actions":{"stateDelta":{},"artifactDelta":{},"requestedAuthConfigs":{}},"longRunningToolIds":[],"id":"ptcjaZBa","timestamp":1743712255.313043}

data: {"content":{"parts":[{"functionResponse":{"id":"af-f83f8af9-f732-46b6-8cb5-7b5b73bbf13d","name":"get_weather","response":{"status":"success","report":"The weather in New York is sunny with a temperature of 25 degrees Celsius (41 degrees Fahrenheit)."}}}],"role":"user"},"invocationId":"e-3f6d7765-5287-419e-9991-5fffa1a75565","author":"weather_time_agent","actions":{"stateDelta":{},"artifactDelta":{},"requestedAuthConfigs":{}},"id":"5aocxjaq","timestamp":1743712257.387306}

data: {"content":{"parts":[{"text":"OK. The weather in New York is sunny with a temperature of 25 degrees Celsius (41 degrees Fahrenheit).\n"}],"role":"model"},"invocationId":"e-3f6d7765-5287-419e-9991-5fffa1a75565","author":"weather_time_agent","actions":{"stateDelta":{},"artifactDelta":{},"requestedAuthConfigs":{}},"id":"rAnWGSiV","timestamp":1743712257.391317}
```
**Enviar una consulta con un archivo codificado en base64 usando `/run` o `/run_sse`**

```shell
curl -X POST http://localhost:8000/run \
-H 'Content-Type: application/json' \
-d '{
   "appName":"my_sample_agent",
   "userId":"u_123",
   "sessionId":"s_123",
   "newMessage":{
      "role":"user",
      "parts":[
         {
            "text":"Describe this image"
         },
         {
            "inlineData":{
               "displayName":"my_image.png",
               "data":"iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAYAAAD0eNT6AAAACXBIWXMAAAsTAAALEwEAmpw...",
               "mimeType":"image/png"
            }
         }
      ]
   },
   "streaming":false
}'
```

!!! info

    Si estás usando `/run_sse`, deberías ver cada evento tan pronto como esté
    disponible.

## Integraciones

ADK usa [Callbacks](../callbacks/index.md) para integrarse con herramientas de
observabilidad de terceros. Estas integraciones capturan trazas detalladas de llamadas de agentes
e interacciones, que son cruciales para comprender el comportamiento, depurar
problemas y evaluar el rendimiento.

* [Comet Opik](https://github.com/comet-ml/opik) es una plataforma de observabilidad y evaluación de LLM de código abierto que
  [admite nativamente ADK](https://www.comet.com/docs/opik/tracing/integrations/adk).

## Desplegar tu agente

Ahora que has verificado la operación local de tu agente, estás listo para
desplegar tu agente. Aquí hay algunas formas en que puedes desplegar tu agente:

* Desplegar en [Agent Engine](../deploy/agent-engine/index.md), una forma simple de desplegar
  tus agentes de ADK a un servicio administrado en Vertex AI en Google Cloud.
* Desplegar en [Cloud Run](../deploy/cloud-run.md) y tener control total sobre cómo
  escalas y administras tus agentes usando arquitectura serverless en Google
  Cloud.

## Documentación interactiva de la API

El servidor API genera automáticamente documentación interactiva de la API usando Swagger UI. Esta es una herramienta invaluable para explorar endpoints, comprender formatos de petición y probar tu agente directamente desde tu navegador.

Para acceder a la documentación interactiva, inicia el servidor API y navega a [http://localhost:8000/docs](http://localhost:8000/docs) en tu navegador web.

Verás una lista completa e interactiva de todos los endpoints de API disponibles, que puedes expandir para ver información detallada sobre parámetros, cuerpos de petición y esquemas de respuesta. Incluso puedes hacer clic en "Try it out" para enviar peticiones en vivo a tus agentes en ejecución.

## Endpoints de la API

Las siguientes secciones detallan los endpoints principales para interactuar con tus agentes.

!!! note "Convención de Nomenclatura JSON"
    - **Tanto los cuerpos de Petición como de Respuesta** usarán `camelCase` para nombres de campo (ej., `"appName"`).

### Endpoints de utilidad

#### Listar agentes disponibles

Devuelve una lista de todas las aplicaciones de agentes descubiertas por el servidor.

*   **Método:** `GET`
*   **Ruta:** `/list-apps`

**Ejemplo de Petición**
```shell
curl -X GET http://localhost:8000/list-apps
```

**Ejemplo de Respuesta**
```json
["my_sample_agent", "another_agent"]
```

---

### Gestión de sesiones

Las sesiones almacenan el estado y el historial de eventos para la interacción de un usuario específico con un agente.

#### Actualizar una sesión

Actualiza una sesión existente.

*   **Método:** `PATCH`
*   **Ruta:** `/apps/{app_name}/users/{user_id}/sessions/{session_id}`

**Cuerpo de la Petición**
```json
{
  "stateDelta": {
    "key1": "value1",
    "key2": 42
  }
}
```

**Ejemplo de Petición**
```shell
curl -X PATCH http://localhost:8000/apps/my_sample_agent/users/u_123/sessions/s_abc \
  -H "Content-Type: application/json" \
  -d '{"stateDelta":{"visit_count": 5}}'
```

**Ejemplo de Respuesta**
```json
{"id":"s_abc","appName":"my_sample_agent","userId":"u_123","state":{"visit_count":5},"events":[],"lastUpdateTime":1743711430.022186}
```

#### Obtener una sesión

Recupera los detalles de una sesión específica, incluyendo su estado actual y todos los eventos asociados.

*   **Método:** `GET`
*   **Ruta:** `/apps/{app_name}/users/{user_id}/sessions/{session_id}`

**Ejemplo de Petición**
```shell
curl -X GET http://localhost:8000/apps/my_sample_agent/users/u_123/sessions/s_abc
```

**Ejemplo de Respuesta**
```json
{"id":"s_abc","appName":"my_sample_agent","userId":"u_123","state":{"visit_count":5},"events":[...],"lastUpdateTime":1743711430.022186}
```

#### Eliminar una sesión

Elimina una sesión y todos sus datos asociados.

*   **Método:** `DELETE`
*   **Ruta:** `/apps/{app_name}/users/{user_id}/sessions/{session_id}`

**Ejemplo de Petición**
```shell
curl -X DELETE http://localhost:8000/apps/my_sample_agent/users/u_123/sessions/s_abc
```

**Ejemplo de Respuesta**
Una eliminación exitosa devuelve una respuesta vacía con un código de estado `204 No Content`.

---

### Ejecución del agente

Estos endpoints se usan para enviar un nuevo mensaje a un agente y obtener una respuesta.

#### Ejecutar agente (respuesta única)

Ejecuta el agente y devuelve todos los eventos generados en un único array JSON después de que se complete la ejecución.

*   **Método:** `POST`
*   **Ruta:** `/run`

**Cuerpo de la Petición**
```json
{
  "appName": "my_sample_agent",
  "userId": "u_123",
  "sessionId": "s_abc",
  "newMessage": {
    "role": "user",
    "parts": [
      { "text": "What is the capital of France?" }
    ]
  }
}
```

En TypeScript, actualmente solo se admiten nombres de campo en `camelCase` (ej.
`appName`, `userId`, `sessionId`, etc.).

**Ejemplo de Petición**
```shell
curl -X POST http://localhost:8000/run \
  -H "Content-Type: application/json" \
  -d '{
    "appName": "my_sample_agent",
    "userId": "u_123",
    "sessionId": "s_abc",
    "newMessage": {
      "role": "user",
      "parts": [{"text": "What is the capital of France?"}]
    }
  }'
```

#### Ejecutar agente (streaming)

Ejecuta el agente y transmite eventos de vuelta al cliente a medida que se generan usando [Server-Sent Events (SSE)](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events).

*   **Método:** `POST`
*   **Ruta:** `/run_sse`

**Cuerpo de la Petición**
El cuerpo de la petición es el mismo que para `/run`, con una bandera opcional adicional `streaming`.
```json
{
  "appName": "my_sample_agent",
  "userId": "u_123",
  "sessionId": "s_abc",
  "newMessage": {
    "role": "user",
    "parts": [
      { "text": "What is the weather in New York?" }
    ]
  },
  "streaming": true
}
```
- `streaming`: (Opcional) Establece en `true` para habilitar streaming a nivel de token para respuestas del modelo. Por defecto es `false`.

**Ejemplo de Petición**
```shell
curl -X POST http://localhost:8000/run_sse \
  -H "Content-Type: application/json" \
  -d '{
    "appName": "my_sample_agent",
    "userId": "u_123",
    "sessionId": "s_abc",
    "newMessage": {
      "role": "user",
      "parts": [{"text": "What is the weather in New York?"}]
    },
    "streaming": false
  }'
```