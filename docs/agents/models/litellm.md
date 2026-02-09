# Conector de modelo LiteLLM para agentes ADK

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span>
</div>

[LiteLLM](https://docs.litellm.ai/) es una biblioteca de Python que actúa como una
capa de traducción para modelos y servicios de alojamiento de modelos, proporcionando una
interfaz estandarizada compatible con OpenAI para más de 100 LLMs. ADK proporciona
integración a través de la biblioteca LiteLLM, permitiéndote acceder a una amplia gama de
LLMs de proveedores como OpenAI, Anthropic (no Vertex AI), Cohere, y muchos
otros. Puedes ejecutar modelos de código abierto localmente o auto-alojarlos e integrarlos
usando LiteLLM para control operacional, ahorro de costos, privacidad, o casos de uso
sin conexión.

Puedes usar la biblioteca LiteLLM para acceder a modelos de IA alojados de forma remota o local:

*   **Alojamiento remoto de modelos:** Usa la clase wrapper `LiteLlm` y establécela
    como el parámetro `model` de `LlmAgent`.
*   **Alojamiento local de modelos:** Usa la clase wrapper `LiteLlm` configurada para
    apuntar a tu servidor de modelo local. Para ejemplos de soluciones de alojamiento
    local de modelos, consulta la documentación de [Ollama](/adk-docs/agents/models/ollama/)
    o [vLLM](/adk-docs/agents/models/vllm/).

??? warning "Codificación en Windows con LiteLLM"

    Al usar agentes ADK con LiteLLM en Windows, podrías encontrar un
    `UnicodeDecodeError`. Este error ocurre porque LiteLLM puede intentar leer
    archivos en caché usando la codificación predeterminada de Windows (`cp1252`) en lugar de UTF-8.
    Prevén este error estableciendo la variable de entorno `PYTHONUTF8` a
    `1`. Esto fuerza a Python a usar UTF-8 para todas las operaciones de E/S de archivos.

    **Ejemplo (PowerShell):**
    ```powershell
    # Establecer para la sesión actual
    $env:PYTHONUTF8 = "1"

    # Establecer de forma persistente para el usuario
    [System.Environment]::SetEnvironmentVariable('PYTHONUTF8', '1', [System.EnvironmentVariableTarget]::User)
    ```

## Configuración

1. **Instalar LiteLLM:**
        ```shell
        pip install litellm
        ```
2. **Establecer Claves API del Proveedor:** Configura las claves API como variables de entorno para
   los proveedores específicos que pretendes usar.

    * *Ejemplo para OpenAI:*

        ```shell
        export OPENAI_API_KEY="YOUR_OPENAI_API_KEY"
        ```

    * *Ejemplo para Anthropic (no Vertex AI):*

        ```shell
        export ANTHROPIC_API_KEY="YOUR_ANTHROPIC_API_KEY"
        ```

    * *Consulta la
      [Documentación de Proveedores de LiteLLM](https://docs.litellm.ai/docs/providers)
      para los nombres correctos de variables de entorno para otros proveedores.*

## Ejemplo de implementación

```python
from google.adk.agents import LlmAgent
from google.adk.models.lite_llm import LiteLlm

# --- Agente de Ejemplo usando GPT-4o de OpenAI ---
# (Requiere OPENAI_API_KEY)
agent_openai = LlmAgent(
    model=LiteLlm(model="openai/gpt-4o"), # Formato de cadena de modelo LiteLLM
    name="openai_agent",
    instruction="You are a helpful assistant powered by GPT-4o.",
    # ... otros parámetros del agente
)

# --- Agente de Ejemplo usando Claude Haiku de Anthropic (no Vertex) ---
# (Requiere ANTHROPIC_API_KEY)
agent_claude_direct = LlmAgent(
    model=LiteLlm(model="anthropic/claude-3-haiku-20240307"),
    name="claude_direct_agent",
    instruction="You are an assistant powered by Claude Haiku.",
    # ... otros parámetros del agente
)
```