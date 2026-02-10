# Integra APIs REST con OpenAPI

<div class="language-support-tag">
  <span class="lst-supported">Compatible con ADK</span><span class="lst-python">Python v0.1.0</span>
</div>

ADK simplifica la interacción con APIs REST externas generando automáticamente herramientas invocables directamente desde una [Especificación OpenAPI (v3.x)](https://swagger.io/specification/). Esto elimina la necesidad de definir manualmente herramientas de función individuales para cada endpoint de la API.

!!! tip "Beneficio Principal"
    Usa `OpenAPIToolset` para crear instantáneamente herramientas de agente (`RestApiTool`) desde tu documentación de API existente (especificación OpenAPI), permitiendo que los agentes llamen sin problemas a tus servicios web.

## Componentes Clave

* **`OpenAPIToolset`**: Esta es la clase principal que usarás. La inicializas con tu especificación OpenAPI, y maneja el análisis y generación de herramientas.
* **`RestApiTool`**: Esta clase representa una única operación de API invocable (como `GET /pets/{petId}` o `POST /pets`). `OpenAPIToolset` crea una instancia de `RestApiTool` por cada operación definida en tu especificación.

## Cómo Funciona

El proceso involucra estos pasos principales cuando usas `OpenAPIToolset`:

1. **Inicialización y Análisis**:
    * Proporcionas la especificación OpenAPI a `OpenAPIToolset` ya sea como un diccionario de Python, un string JSON, o un string YAML.
    * El conjunto de herramientas analiza internamente la especificación, resolviendo cualquier referencia interna (`$ref`) para entender la estructura completa de la API.

2. **Descubrimiento de Operaciones**:
    * Identifica todas las operaciones de API válidas (ej., `GET`, `POST`, `PUT`, `DELETE`) definidas dentro del objeto `paths` de tu especificación.

3. **Generación de Herramientas**:
    * Por cada operación descubierta, `OpenAPIToolset` crea automáticamente una instancia correspondiente de `RestApiTool`.
    * **Nombre de Herramienta**: Derivado del `operationId` en la especificación (convertido a `snake_case`, máximo 60 caracteres). Si falta `operationId`, se genera un nombre desde el método y la ruta.
    * **Descripción de Herramienta**: Usa el `summary` o `description` de la operación para el LLM.
    * **Detalles de API**: Almacena internamente el método HTTP requerido, ruta, URL base del servidor, parámetros (path, query, header, cookie), y esquema del cuerpo de la petición.

4. **Funcionalidad de `RestApiTool`**: Cada `RestApiTool` generado:
    * **Generación de Esquema**: Crea dinámicamente una `FunctionDeclaration` basada en los parámetros de la operación y el cuerpo de la petición. Este esquema le dice al LLM cómo llamar la herramienta (qué argumentos se esperan).
    * **Ejecución**: Cuando es llamado por el LLM, construye la petición HTTP correcta (URL, headers, parámetros de query, body) usando los argumentos proporcionados por el LLM y los detalles de la especificación OpenAPI. Maneja la autenticación (si está configurada) y ejecuta la llamada a la API usando la librería `requests`.
    * **Manejo de Respuesta**: Devuelve la respuesta de la API (típicamente JSON) de vuelta al flujo del agente.

5. **Autenticación**: Puedes configurar autenticación global (como claves API u OAuth - ver [Authentication](/tools/authentication/) para detalles) al inicializar `OpenAPIToolset`. Esta configuración de autenticación se aplica automáticamente a todas las instancias de `RestApiTool` generadas.

## Flujo de Trabajo de Uso

Sigue estos pasos para integrar una especificación OpenAPI en tu agente:

1. **Obtener Especificación**: Obtén tu documento de especificación OpenAPI (ej., cargar desde un archivo `.json` o `.yaml`, obtener desde una URL).
2. **Instanciar Conjunto de Herramientas**: Crea una instancia de `OpenAPIToolset`, pasando el contenido de la especificación y el tipo (`spec_str`/`spec_dict`, `spec_str_type`). Proporciona detalles de autenticación (`auth_scheme`, `auth_credential`) si la API lo requiere.

    ```python
    from google.adk.tools.openapi_tool.openapi_spec_parser.openapi_toolset import OpenAPIToolset

    # Ejemplo con un string JSON
    openapi_spec_json = '...' # Tu string JSON de OpenAPI
    toolset = OpenAPIToolset(spec_str=openapi_spec_json, spec_str_type="json")

    # Ejemplo con un diccionario
    # openapi_spec_dict = {...} # Tu especificación OpenAPI como dict
    # toolset = OpenAPIToolset(spec_dict=openapi_spec_dict)
    ```

3. **Agregar al Agente**: Incluye las herramientas recuperadas en la lista `tools` de tu `LlmAgent`.

    ```python
    from google.adk.agents import LlmAgent

    my_agent = LlmAgent(
        name="api_interacting_agent",
        model="gemini-2.0-flash", # O tu modelo preferido
        tools=[toolset], # Pasa el conjunto de herramientas
        # ... otra configuración del agente ...
    )
    ```

4. **Instruir al Agente**: Actualiza las instrucciones de tu agente para informarle sobre las nuevas capacidades de API y los nombres de las herramientas que puede usar (ej., `list_pets`, `create_pet`). Las descripciones de herramientas generadas desde la especificación también ayudarán al LLM.
5. **Ejecutar Agente**: Ejecuta tu agente usando el `Runner`. Cuando el LLM determine que necesita llamar a una de las APIs, generará una llamada de función apuntando a la `RestApiTool` apropiada, que luego manejará la petición HTTP automáticamente.

## Ejemplo

Este ejemplo demuestra la generación de herramientas desde una especificación OpenAPI simple de Pet Store (usando `httpbin.org` para respuestas simuladas) e interactuando con ellas a través de un agente.

???+ "Code: Pet Store API"

    ```python title="openapi_example.py"
    --8<-- "examples/python/snippets/tools/openapi_tool.py"
    ```