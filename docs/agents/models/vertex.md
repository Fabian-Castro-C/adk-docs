# Modelos alojados en Vertex AI para agentes ADK

Para escalabilidad de nivel empresarial, confiabilidad e integración con el
ecosistema MLOps de Google Cloud, puedes usar modelos desplegados en Endpoints de
Vertex AI. Esto incluye modelos de Model Garden o tus propios modelos ajustados.

**Método de Integración:** Pasa la cadena completa del recurso Endpoint de Vertex AI
(`projects/PROJECT_ID/locations/LOCATION/endpoints/ENDPOINT_ID`) directamente al
parámetro `model` de `LlmAgent`.

## Configuración de Vertex AI

Asegúrate de que tu entorno esté configurado para Vertex AI:

1. **Autenticación:** Usa Application Default Credentials (ADC):

    ```shell
    gcloud auth application-default login
    ```

2. **Variables de Entorno:** Establece tu proyecto y ubicación:

    ```shell
    export GOOGLE_CLOUD_PROJECT="YOUR_PROJECT_ID"
    export GOOGLE_CLOUD_LOCATION="YOUR_VERTEX_AI_LOCATION" # ej., us-central1
    ```

3. **Habilitar Backend de Vertex:** Crucialmente, asegúrate de que la biblioteca
   `google-genai` apunte a Vertex AI:

    ```shell
    export GOOGLE_GENAI_USE_VERTEXAI=TRUE
    ```

## Despliegues de Model Garden

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.2.0</span>
</div>

Puedes desplegar varios modelos abiertos y propietarios desde el
[Vertex AI Model Garden](https://console.cloud.google.com/vertex-ai/model-garden)
a un endpoint.

**Ejemplo:**

```python
from google.adk.agents import LlmAgent
from google.genai import types # Para objetos de configuración

# --- Agente de ejemplo usando un modelo Llama 3 desplegado desde Model Garden ---

# Reemplaza con el nombre real del recurso Endpoint de Vertex AI
llama3_endpoint = "projects/YOUR_PROJECT_ID/locations/us-central1/endpoints/YOUR_LLAMA3_ENDPOINT_ID"

agent_llama3_vertex = LlmAgent(
    model=llama3_endpoint,
    name="llama3_vertex_agent",
    instruction="You are a helpful assistant based on Llama 3, hosted on Vertex AI.",
    generate_content_config=types.GenerateContentConfig(max_output_tokens=2048),
    # ... otros parámetros del agente
)
```

## Endpoints de Modelos Ajustados

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.2.0</span>
</div>

Desplegar tus modelos ajustados (ya sea basados en Gemini u otras arquitecturas
soportadas por Vertex AI) resulta en un endpoint que puede usarse directamente.

**Ejemplo:**

```python
from google.adk.agents import LlmAgent

# --- Agente de ejemplo usando un endpoint de modelo Gemini ajustado ---

# Reemplaza con el nombre del recurso endpoint de tu modelo ajustado
finetuned_gemini_endpoint = "projects/YOUR_PROJECT_ID/locations/us-central1/endpoints/YOUR_FINETUNED_ENDPOINT_ID"

agent_finetuned_gemini = LlmAgent(
    model=finetuned_gemini_endpoint,
    name="finetuned_gemini_agent",
    instruction="You are a specialized assistant trained on specific data.",
    # ... otros parámetros del agente
)
```

## Anthropic Claude en Vertex AI {#third-party-models-on-vertex-ai-eg-anthropic-claude}

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.2.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Algunos proveedores, como Anthropic, hacen que sus modelos estén disponibles
directamente a través de Vertex AI.

=== "Python"

    **Método de Integración:** Usa la cadena directa del modelo (ej.,
    `"claude-3-sonnet@20240229"`), *pero requiere registro manual* dentro de ADK.

    **¿Por qué Registro?** El registro de ADK reconoce automáticamente cadenas
    `gemini-*` y cadenas estándar de endpoint de Vertex AI
    (`projects/.../endpoints/...`) y las enruta a través de la biblioteca
    `google-genai`. Para otros tipos de modelos usados directamente a través de
    Vertex AI (como Claude), debes indicar explícitamente al registro de ADK qué
    clase de envoltura específica (`Claude` en este caso) sabe cómo manejar esa
    cadena de identificador de modelo con el backend de Vertex AI.

    **Configuración:**

    1. **Entorno de Vertex AI:** Asegúrate de que la configuración consolidada de
       Vertex AI (ADC, Vars de Entorno, `GOOGLE_GENAI_USE_VERTEXAI=TRUE`) esté
       completa.

    2. **Instalar Biblioteca del Proveedor:** Instala la biblioteca cliente necesaria
       configurada para Vertex AI.

        ```shell
        pip install "anthropic[vertex]"
        ```

    3. **Registrar Clase del Modelo:** Agrega este código cerca del inicio de tu
       aplicación, *antes* de crear un agente usando la cadena del modelo Claude:

        ```python
        # Requerido para usar cadenas de modelo Claude directamente a través de Vertex AI con LlmAgent
        from google.adk.models.anthropic_llm import Claude
        from google.adk.models.registry import LLMRegistry

        LLMRegistry.register(Claude)
        ```

       **Ejemplo:**

       ```python
       from google.adk.agents import LlmAgent
       from google.adk.models.anthropic_llm import Claude # Importación necesaria para el registro
       from google.adk.models.registry import LLMRegistry # Importación necesaria para el registro
       from google.genai import types

       # --- Registrar clase Claude (hacer esto una vez al inicio) ---
       LLMRegistry.register(Claude)

       # --- Agente de ejemplo usando Claude 3 Sonnet en Vertex AI ---

       # Nombre de modelo estándar para Claude 3 Sonnet en Vertex AI
       claude_model_vertexai = "claude-3-sonnet@20240229"

       agent_claude_vertexai = LlmAgent(
           model=claude_model_vertexai, # Pasar la cadena directa después del registro
           name="claude_vertexai_agent",
           instruction="You are an assistant powered by Claude 3 Sonnet on Vertex AI.",
           generate_content_config=types.GenerateContentConfig(max_output_tokens=4096),
           # ... otros parámetros del agente
       )
       ```

=== "Java"

    **Método de Integración:** Instancia directamente la clase de modelo específica del
    proveedor (ej., `com.google.adk.models.Claude`) y configúrala con un backend de
    Vertex AI.

    **¿Por qué Instanciación Directa?** El `LlmRegistry` del ADK de Java maneja
    principalmente modelos Gemini por defecto. Para modelos de terceros como Claude en
    Vertex AI, proporcionas directamente una instancia de la clase de envoltura del ADK
    (ej., `Claude`) al `LlmAgent`. Esta clase de envoltura es responsable de
    interactuar con el modelo a través de su biblioteca cliente específica, configurada
    para Vertex AI.

    **Configuración:**

    1.  **Entorno de Vertex AI:**
        *   Asegúrate de que tu proyecto y región de Google Cloud estén configurados
            correctamente.
        *   **Application Default Credentials (ADC):** Asegúrate de que ADC esté
            configurado correctamente en tu entorno. Esto se hace típicamente ejecutando
            `gcloud auth application-default login`. Las bibliotecas cliente de Java
            usan estas credenciales para autenticarse con Vertex AI. Sigue la
            [documentación de Java de Google Cloud sobre ADC](https://cloud.google.com/java/docs/reference/google-auth-library/latest/com.google.auth.oauth2.GoogleCredentials#com_google_auth_oauth2_GoogleCredentials_getApplicationDefault__)
            para una configuración detallada.

    2.  **Dependencias de Bibliotecas del Proveedor:**
        *   **Bibliotecas Cliente de Terceros (A menudo Transitivas):** La biblioteca
            principal de ADK a menudo incluye las bibliotecas cliente necesarias para
            modelos comunes de terceros en Vertex AI (como las clases requeridas de
            Anthropic) como **dependencias transitivas**. Esto significa que podrías no
            necesitar agregar explícitamente una dependencia separada para el SDK de
            Anthropic Vertex en tu `pom.xml` o `build.gradle`.

    3.  **Instanciar y Configurar el Modelo:**
        Al crear tu `LlmAgent`, instancia la clase `Claude` (o equivalente para otro
        proveedor) y configura su `VertexBackend`.

    **Ejemplo:**

    ```java
    import com.anthropic.client.AnthropicClient;
    import com.anthropic.client.okhttp.AnthropicOkHttpClient;
    import com.anthropic.vertex.backends.VertexBackend;
    import com.google.adk.agents.LlmAgent;
    import com.google.adk.models.Claude; // Envoltura de ADK para Claude
    import com.google.auth.oauth2.GoogleCredentials;
    import java.io.IOException;

    // ... otras importaciones

    public class ClaudeVertexAiAgent {

        public static LlmAgent createAgent() throws IOException {
            // Nombre del modelo para Claude 3 Sonnet en Vertex AI (u otras versiones)
            String claudeModelVertexAi = "claude-3-7-sonnet"; // O cualquier otro modelo Claude

            // Configurar el AnthropicOkHttpClient con el VertexBackend
            AnthropicClient anthropicClient = AnthropicOkHttpClient.builder()
                .backend(
                    VertexBackend.builder()
                        .region("us-east5") // Especifica tu región de Vertex AI
                        .project("your-gcp-project-id") // Especifica tu ID de Proyecto GCP
                        .googleCredentials(GoogleCredentials.getApplicationDefault())
                        .build())
                .build();

            // Instanciar LlmAgent con la envoltura Claude de ADK
            LlmAgent agentClaudeVertexAi = LlmAgent.builder()
                .model(new Claude(claudeModelVertexAi, anthropicClient)) // Pasar la instancia de Claude
                .name("claude_vertexai_agent")
                .instruction("You are an assistant powered by Claude 3 Sonnet on Vertex AI.")
                // .generateContentConfig(...) // Opcional: Agregar configuración de generación si es necesario
                // ... otros parámetros del agente
                .build();

            return agentClaudeVertexAi;
        }

        public static void main(String[] args) {
            try {
                LlmAgent agent = createAgent();
                System.out.println("Successfully created agent: " + agent.name());
                // Aquí típicamente configurarías un Runner y Session para interactuar con el agente
            } catch (IOException e) {
                System.err.println("Failed to create agent: " + e.getMessage());
                e.printStackTrace();
            }
        }
    }
    ```

## Modelos Abiertos en Vertex AI {#open-models}

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span>
</div>

Vertex AI ofrece una selección curada de modelos de código abierto, como Meta Llama,
a través de Model-as-a-Service (MaaS). Estos modelos son accesibles mediante APIs
administradas, permitiéndote desplegar y escalar sin gestionar la infraestructura
subyacente. Para una lista completa de opciones disponibles, consulta la
documentación de [modelos abiertos de Vertex AI para MaaS](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/maas/use-open-models#open-models).

=== "Python"

    Puedes usar la biblioteca [LiteLLM](https://docs.litellm.ai/) para acceder a
    modelos abiertos como Llama de Meta en VertexAI MaaS

    **Método de Integración:** Usa la clase de envoltura `LiteLlm` y establécela
    como el parámetro `model` de `LlmAgent`. Asegúrate de revisar la documentación
    del [conector de modelo LiteLLM para agentes ADK](/adk-docs/agents/models/litellm/#litellm-model-connector-for-adk-agents)
    sobre cómo usar LiteLLM en ADK

    **Configuración:**

    1. **Entorno de Vertex AI:** Asegúrate de que la configuración consolidada de
       Vertex AI (ADC, Vars de Entorno, `GOOGLE_GENAI_USE_VERTEXAI=TRUE`) esté
       completa.

    2. **Instalar LiteLLM:**
            ```shell
            pip install litellm
            ```
    
    **Ejemplo:**

    ```python
    from google.adk.agents import LlmAgent
    from google.adk.models.lite_llm import LiteLlm

    # --- Agente de ejemplo usando Llama 4 Scout de Meta ---
    agent_llama_vertexai = LlmAgent(
        model=LiteLlm(model="vertex_ai/meta/llama-4-scout-17b-16e-instruct-maas"), # Formato de cadena de modelo LiteLLM
        name="llama4_agent",
        instruction="You are a helpful assistant powered by Llama 4 Scout.",
        # ... otros parámetros del agente
    )

    ```