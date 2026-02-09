# Modelos Google Gemini para agentes ADK

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.2.0</span>
</div>

ADK soporta la familia Google Gemini de modelos de IA generativa que proporcionan un
poderoso conjunto de modelos con una amplia gama de características. ADK proporciona soporte para muchas
características de Gemini, incluyendo
[Ejecución de código](/adk-docs/tools/gemini-api/code-execution/),
[Búsqueda de Google](/adk-docs/tools/gemini-api/google-search/),
[Caché de contexto](/adk-docs/context/caching/),
[Uso de computadora](/adk-docs/tools/gemini-api/computer-use/)
y la [API de Interacciones](#interactions-api).

## Comenzar

Los siguientes ejemplos de código muestran una implementación básica para usar modelos Gemini
en tus agentes:

=== "Python"

    ```python
    from google.adk.agents import LlmAgent

    # --- Ejemplo usando un modelo Gemini Flash estable ---
    agent_gemini_flash = LlmAgent(
        # Usar el identificador del modelo Flash estable más reciente
        model="gemini-2.5-flash",
        name="gemini_flash_agent",
        instruction="You are a fast and helpful Gemini assistant.",
        # ... otros parámetros del agente
    )
    ```

=== "TypeScript"

    ```typescript
    import {LlmAgent} from '@google/adk';

    // --- Ejemplo #2: usando un poderoso modelo Gemini Pro con Clave API en el modelo ---
    export const rootAgent = new LlmAgent({
      name: 'hello_time_agent',
      model: 'gemini-2.5-flash',
      description: 'Gemini flash agent',
      instruction: `You are a fast and helpful Gemini assistant.`,
    });
    ```

=== "Go"

    ```go
    import (
    	"google.golang.org/adk/agent/llmagent"
    	"google.golang.org/adk/model/gemini"
    	"google.golang.org/genai"
    )

    --8<-- "examples/go/snippets/agents/models/models.go:gemini-example"
    ```

=== "Java"

    ```java
    // --- Ejemplo #1: usando un modelo Gemini Flash estable con variables ENV ---
    LlmAgent agentGeminiFlash =
        LlmAgent.builder()
            // Usar el identificador del modelo Flash estable más reciente
            .model("gemini-2.5-flash") // Establecer variables ENV para usar este modelo
            .name("gemini_flash_agent")
            .instruction("You are a fast and helpful Gemini assistant.")
            // ... otros parámetros del agente
            .build();
    ```


## Autenticación de modelos Gemini

Esta sección cubre la autenticación con los modelos Gemini de Google, ya sea a través de Google AI Studio para desarrollo rápido o Google Cloud Vertex AI para aplicaciones empresariales. Esta es la forma más directa de usar los modelos insignia de Google dentro de ADK.

**Método de integración:** Una vez que estés autenticado usando uno de los métodos siguientes, puedes pasar la cadena identificadora del modelo directamente al
parámetro `model` de `LlmAgent`.


!!! tip

    La biblioteca `google-genai`, usada internamente por ADK para modelos Gemini, puede conectarse
    a través de Google AI Studio o Vertex AI.

    **Soporte de modelos para streaming de voz/video**

    Para usar streaming de voz/video en ADK, necesitarás usar modelos Gemini
    que soporten la Live API. Puedes encontrar el **ID(s) de modelo** que
    soportan la Gemini Live API en la documentación:

    - [Google AI Studio: Gemini Live API](https://ai.google.dev/gemini-api/docs/models#live-api)
    - [Vertex AI: Gemini Live API](https://cloud.google.com/vertex-ai/generative-ai/docs/live-api)

### Google AI Studio

Este es el método más simple y es recomendado para comenzar rápidamente.

*   **Método de autenticación:** Clave API
*   **Configuración:**
    1.  **Obtén una clave API:** Obtén tu clave desde [Google AI Studio](https://aistudio.google.com/apikey).
    2.  **Establece variables de entorno:** Crea un archivo `.env` (Python) o `.properties` (Java) en el directorio raíz de tu proyecto y agrega las siguientes líneas. ADK cargará automáticamente este archivo.

        ```shell
        export GOOGLE_API_KEY="YOUR_GOOGLE_API_KEY"
        export GOOGLE_GENAI_USE_VERTEXAI=FALSE
        ```

        (o)

        Pasa estas variables durante la inicialización del modelo a través del `Client` (ver ejemplo abajo).

* **Modelos:** Encuentra todos los modelos disponibles en el
  [sitio de Google AI para Desarrolladores](https://ai.google.dev/gemini-api/docs/models).

### Google Cloud Vertex AI

Para casos de uso escalables y orientados a producción, Vertex AI es la plataforma recomendada. Gemini en Vertex AI soporta características de grado empresarial, seguridad y controles de cumplimiento. Basándose en tu entorno de desarrollo y caso de uso, *elige uno de los métodos siguientes para autenticarte*.

**Prerrequisitos:** Un Proyecto de Google Cloud con [Vertex AI habilitado](https://console.cloud.google.com/apis/enableflow;apiid=aiplatform.googleapis.com).

### **Método A: Credenciales de Usuario (para Desarrollo Local)**

1.  **Instala el CLI de gcloud:** Sigue las [instrucciones de instalación](https://cloud.google.com/sdk/docs/install) oficiales.
2.  **Inicia sesión usando ADC:** Este comando abre un navegador para autenticar tu cuenta de usuario para desarrollo local.
    ```bash
    gcloud auth application-default login
    ```
3.  **Establece variables de entorno:**
    ```shell
    export GOOGLE_CLOUD_PROJECT="YOUR_PROJECT_ID"
    export GOOGLE_CLOUD_LOCATION="YOUR_VERTEX_AI_LOCATION" # ej., us-central1
    ```

    Indica explícitamente a la biblioteca que use Vertex AI:

    ```shell
    export GOOGLE_GENAI_USE_VERTEXAI=TRUE
    ```

4. **Modelos:** Encuentra IDs de modelos disponibles en la
  [documentación de Vertex AI](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/models).

### **Método B: Modo Express de Vertex AI**
[Vertex AI Express Mode](https://cloud.google.com/vertex-ai/generative-ai/docs/start/express-mode/overview) ofrece una configuración simplificada basada en clave API para prototipado rápido.

1.  **Regístrate en Express Mode** para obtener tu clave API.
2.  **Establece variables de entorno:**
    ```shell
    export GOOGLE_API_KEY="PASTE_YOUR_EXPRESS_MODE_API_KEY_HERE"
    export GOOGLE_GENAI_USE_VERTEXAI=TRUE
    ```

### **Método C: Cuenta de Servicio (para Producción y Automatización)**

Para aplicaciones desplegadas, una cuenta de servicio es el método estándar.

1.  [**Crea una Cuenta de Servicio**](https://cloud.google.com/iam/docs/service-accounts-create#console) y otórgale el rol de `Vertex AI User`.
2.  **Proporciona credenciales a tu aplicación:**
    *   **En Google Cloud:** Si estás ejecutando el agente en Cloud Run, GKE, VM u otros servicios de Google Cloud, el entorno puede proporcionar automáticamente las credenciales de la cuenta de servicio. No tienes que crear un archivo de clave.
    *   **En otro lugar:** Crea un [archivo de clave de cuenta de servicio](https://cloud.google.com/iam/docs/keys-create-delete#console) y apunta a él con una variable de entorno:
        ```bash
        export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your/keyfile.json"
        ```
    En lugar del archivo de clave, también puedes autenticar la cuenta de servicio usando Workload Identity. Pero esto está fuera del alcance de esta guía.

!!! warning "Asegura Tus Credenciales"

    Las credenciales de cuenta de servicio o claves API son credenciales poderosas. Nunca
    las expongas públicamente. Usa un gestor de secretos como [Google Cloud Secret
    Manager](https://cloud.google.com/security/products/secret-manager) para almacenar
    y acceder a ellas de forma segura en producción.

!!! note "Versiones de modelos Gemini"

    Siempre verifica la documentación oficial de Gemini para los nombres de modelos más recientes,
    incluyendo versiones preview específicas si es necesario. Los modelos preview pueden tener
    diferentes disponibilidades o limitaciones de cuota.

## Solución de problemas

### Código de Error 429 - RESOURCE_EXHAUSTED

Este error generalmente ocurre si el número de tus solicitudes excede la capacidad asignada para procesar solicitudes.

Para mitigar esto, puedes hacer uno de los siguientes:

1.  Solicitar límites de cuota más altos para el modelo que estás intentando usar.

2.  Habilitar reintentos del lado del cliente. Los reintentos permiten al cliente reintentar automáticamente la solicitud después de un retraso, lo que puede ayudar si el problema de cuota es temporal.

    Hay dos formas en que puedes establecer opciones de reintento:

    **Opción 1:** Establece opciones de reintento en el Agente como parte de generate_content_config.

    Usarías esta opción si estás instanciando este adaptador de modelo por
    ti mismo.

    ```python
    root_agent = Agent(
        model='gemini-2.5-flash',
        ...
        generate_content_config=types.GenerateContentConfig(
            ...
            http_options=types.HttpOptions(
                ...
                retry_options=types.HttpRetryOptions(initial_delay=1, attempts=2),
                ...
            ),
            ...
        )
    ```

    **Opción 2:** Opciones de reintento en este adaptador de modelo.

    Usarías esta opción si estuvieras instanciando la instancia del adaptador
    por ti mismo.

    ```python
    from google.genai import types

    # ...

    agent = Agent(
        model=Gemini(
        retry_options=types.HttpRetryOptions(initial_delay=1, attempts=2),
        )
    )
    ```

## API de Interacciones de Gemini {#interactions-api}

<div class="language-support-tag" title="Java ADK currently supports Gemini and Anthropic models.">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v1.21.0</span>
</div>

La [API de Interacciones](https://ai.google.dev/gemini-api/docs/interactions) de Gemini
es una alternativa a la API de inferencia ***generateContent***, que proporciona
capacidades de conversación con estado, permitiéndote encadenar interacciones usando un
`previous_interaction_id` en lugar de enviar el historial completo de la conversación con
cada solicitud. Usar esta característica puede ser más eficiente para conversaciones largas.

Puedes habilitar la API de Interacciones estableciendo el parámetro `use_interactions_api=True`
en la configuración del modelo Gemini, como se muestra en el siguiente fragmento de código:

```python
from google.adk.agents.llm_agent import Agent
from google.adk.models.google_llm import Gemini
from google.adk.tools.google_search_tool import GoogleSearchTool

root_agent = Agent(
    model=Gemini(
        model="gemini-2.5-flash",
        use_interactions_api=True,  # Habilitar API de Interacciones
    ),
    name="interactions_test_agent",
    tools=[
        GoogleSearchTool(bypass_multi_tools_limit=True),  # Convertida a herramienta de función
        get_current_weather,  # Herramienta de función personalizada
    ],
)
```

Para un ejemplo de código completo, consulta la
[muestra de API de Interacciones](https://github.com/google/adk-python/tree/main/contributing/samples/interactions_api).

### Limitaciones conocidas

La API de Interacciones **no** soporta mezclar herramientas de llamado de función personalizadas con
herramientas integradas, como la
herramienta [Búsqueda de Google](/adk-docs/tools/built-in-tools/#google-search),
dentro del mismo agente. Puedes solucionar esta limitación configurando la
herramienta integrada para operar como una herramienta personalizada usando el parámetro `bypass_multi_tools_limit`:

```python
# Usar bypass_multi_tools_limit=True para convertir google_search a una herramienta de función
GoogleSearchTool(bypass_multi_tools_limit=True)
```

En este ejemplo, esta opción convierte la google_search integrada en una herramienta de llamado
de función (a través de GoogleSearchAgentTool), lo que le permite trabajar junto con
herramientas de función personalizadas.