# Modelos Claude para agentes ADK

<div class="language-support-tag" title="Available for Java. Python support for direct Anthropic API (non-Vertex) is via LiteLLM.">
   <span class="lst-supported">Supported in ADK</span><span class="lst-java">Java v0.2.0</span>
</div>

Puedes integrar los modelos Claude de Anthropic directamente usando una clave API de Anthropic
o desde un backend de Vertex AI en tus aplicaciones Java ADK utilizando la
clase contenedora `Claude` del ADK. También puedes acceder a los modelos de Anthropic a través
de los servicios de Google Cloud Vertex AI. Para más información, consulta la
sección [Third-Party Models on Vertex AI](/agents/models/vertex/#third-party-models-on-vertex-ai-eg-anthropic-claude).
También puedes usar modelos de Anthropic a través de la
biblioteca [LiteLLM](/agents/models/litellm/) para Python.

## Comenzar

Los siguientes ejemplos de código muestran una implementación básica para usar modelos Gemini
en tus agentes:

```java
public static LlmAgent createAgent() {

  AnthropicClient anthropicClient = AnthropicOkHttpClient.builder()
      .apiKey("ANTHROPIC_API_KEY")
      .build();

  Claude claudeModel = new Claude(
      "claude-3-7-sonnet-latest", anthropicClient
  );

  return LlmAgent.builder()
      .name("claude_direct_agent")
      .model(claudeModel)
      .instruction("You are a helpful AI assistant powered by Anthropic Claude.")
      .build();
}
```

## Requisitos previos

1.  **Dependencias:**
    *   **Clases del SDK de Anthropic (Transitivas):** La clase contenedora `com.google.adk.models.Claude`
    del Java ADK depende de clases del SDK oficial de Java de Anthropic. Estas se incluyen típicamente
    como *dependencias transitivas*. Para más información, consulta el
    [Anthropic Java SDK](https://github.com/anthropics/anthropic-sdk-java).

2.  **Clave API de Anthropic:**
    *   Obtén una clave API de Anthropic. Gestiona esta clave de forma segura usando un administrador de secretos.

## Ejemplo de implementación

Instancia `com.google.adk.models.Claude`, proporcionando el nombre del modelo Claude deseado y
un `AnthropicOkHttpClient` configurado con tu clave API. Luego, pasa la instancia `Claude`
a tu `LlmAgent`, como se muestra en el siguiente ejemplo:

```java
import com.anthropic.client.AnthropicClient;
import com.google.adk.agents.LlmAgent;
import com.google.adk.models.Claude;
import com.anthropic.client.okhttp.AnthropicOkHttpClient; // Del SDK de Anthropic

public class DirectAnthropicAgent {

  private static final String CLAUDE_MODEL_ID = "claude-3-7-sonnet-latest"; // O tu modelo Claude preferido

  public static LlmAgent createAgent() {

    // Se recomienda cargar claves sensibles desde una configuración segura
    AnthropicClient anthropicClient = AnthropicOkHttpClient.builder()
        .apiKey("ANTHROPIC_API_KEY")
        .build();

    Claude claudeModel = new Claude(
        CLAUDE_MODEL_ID,
        anthropicClient
    );

    return LlmAgent.builder()
        .name("claude_direct_agent")
        .model(claudeModel)
        .instruction("You are a helpful AI assistant powered by Anthropic Claude.")
        // ... otras configuraciones de LlmAgent
        .build();
  }

  public static void main(String[] args) {
    try {
      LlmAgent agent = createAgent();
      System.out.println("Successfully created direct Anthropic agent: " + agent.name());
    } catch (IllegalStateException e) {
      System.err.println("Error creating agent: " + e.getMessage());
    }
  }
}
```