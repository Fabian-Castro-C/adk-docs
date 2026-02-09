# Modelos de IA para agentes de ADK

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span><span class="lst-typescript">Typescript</span><span class="lst-go">Go</span><span class="lst-java">Java</span>
</div>

El Kit de Desarrollo de Agentes (ADK) está diseñado para ser flexible, permitiéndote
integrar varios Modelos de Lenguaje Grande (LLMs) en tus agentes. Esta sección
detalla cómo aprovechar Gemini e integrar otros modelos populares de manera efectiva,
incluyendo aquellos alojados externamente o ejecutándose localmente.

ADK utiliza principalmente dos mecanismos para la integración de modelos:

1. **String Directo / Registro:** Para modelos estrechamente integrados con Google Cloud,
   como los modelos Gemini accedidos a través de Google AI Studio o Vertex AI, o modelos
   alojados en endpoints de Vertex AI. Accedes a estos modelos proporcionando el nombre del modelo o el string del recurso del endpoint y el registro interno de ADK
   resuelve este string al cliente backend apropiado.

      *  [Modelos Gemini](/adk-docs/agents/models/google-gemini/)
      *  [Modelos Claude](/adk-docs/agents/models/anthropic/)
      *  [Modelos alojados en Vertex AI](/adk-docs/agents/models/vertex/)

2. **Conectores de modelos:** Para una compatibilidad más amplia, especialmente modelos
   fuera del ecosistema de Google o aquellos que requieren configuraciones específicas
   del cliente, como modelos accedidos a través de Apigee o LiteLLM. Instancias una clase wrapper específica, como `ApigeeLlm` o
   `LiteLlm`, y pasas este objeto como el parámetro `model`
   a tu `LlmAgent`.

      *  [Modelos Apigee](/adk-docs/agents/models/apigee/)
      *  [Modelos LiteLLM](/adk-docs/agents/models/litellm/)
      *  [Alojamiento de modelos Ollama](/adk-docs/agents/models/ollama/)
      *  [Alojamiento de modelos vLLM](/adk-docs/agents/models/vllm/)