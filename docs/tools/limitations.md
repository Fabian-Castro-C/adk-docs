# Limitaciones de las herramientas ADK

Algunas herramientas ADK tienen limitaciones que pueden afectar cómo las implementas dentro de un
flujo de trabajo del agente. Esta página lista estas limitaciones de herramientas y soluciones alternativas, si están disponibles.

## Limitación de una herramienta por agente {#one-tool-one-agent}

!!! note "SOLO para Search en ADK Python v1.15.0 y anteriores"

    Esta limitación solo aplica al uso de las herramientas Google Search y Vertex AI Search
    en ADK Python v1.15.0 y anteriores. La versión v1.16.0 de ADK Python y superiores
    proporciona una solución alternativa incorporada para eliminar esta limitación.

En general, puedes usar más de una herramienta en un agente, pero el uso de herramientas específicas
dentro de un agente excluye el uso de cualquier otra herramienta en ese agente. Las
siguientes herramientas ADK solo pueden ser usadas por sí mismas, sin ninguna otra herramienta, en
un único objeto agente:

*   [Code Execution](/adk-docs/tools/gemini-api/code-execution/) con Gemini API
*   [Google Search](/adk-docs/tools/gemini-api/google-search/) con Gemini API
*   [Vertex AI Search](/adk-docs/tools/google-cloud/vertex-ai-search/)

Por ejemplo, el siguiente enfoque que usa una de estas herramientas junto con
otras herramientas, dentro de un único agente, ***no está soportado***:

=== "Python"

    ```py
    root_agent = Agent(
        name="RootAgent",
        model="gemini-2.5-flash",
        description="Code Agent",
        tools=[custom_function],
        code_executor=BuiltInCodeExecutor() # <-- NO soportado cuando se usa con herramientas
    )
    ```

=== "Java"

    ```java
     LlmAgent searchAgent =
            LlmAgent.builder()
                .model(MODEL_ID)
                .name("SearchAgent")
                .instruction("You're a specialist in Google Search")
                .tools(new GoogleSearchTool(), new YourCustomTool()) // <-- NO soportado
                .build();
    ```

### Solución alternativa #1: método AgentTool.create()

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python</span><span class="lst-java">Java</span>
</div>

El siguiente ejemplo de código demuestra cómo usar múltiples herramientas incorporadas o cómo
usar herramientas incorporadas con otras herramientas usando múltiples agentes:

=== "Python"

    ```py
    from google.adk.tools.agent_tool import AgentTool
    from google.adk.agents import Agent
    from google.adk.tools import google_search
    from google.adk.code_executors import BuiltInCodeExecutor

    search_agent = Agent(
        model='gemini-2.0-flash',
        name='SearchAgent',
        instruction="""
        You're a specialist in Google Search
        """,
        tools=[google_search],
    )
    coding_agent = Agent(
        model='gemini-2.0-flash',
        name='CodeAgent',
        instruction="""
        You're a specialist in Code Execution
        """,
        code_executor=BuiltInCodeExecutor(),
    )
    root_agent = Agent(
        name="RootAgent",
        model="gemini-2.0-flash",
        description="Root Agent",
        tools=[AgentTool(agent=search_agent), AgentTool(agent=coding_agent)],
    )
    ```

=== "Java"

    ```java
    import com.google.adk.agents.BaseAgent;
    import com.google.adk.agents.LlmAgent;
    import com.google.adk.tools.AgentTool;
    import com.google.adk.tools.BuiltInCodeExecutionTool;
    import com.google.adk.tools.GoogleSearchTool;
    import com.google.common.collect.ImmutableList;

    public class NestedAgentApp {

      private static final String MODEL_ID = "gemini-2.0-flash";

      public static void main(String[] args) {

        // Define el SearchAgent
        LlmAgent searchAgent =
            LlmAgent.builder()
                .model(MODEL_ID)
                .name("SearchAgent")
                .instruction("You're a specialist in Google Search")
                .tools(new GoogleSearchTool()) // Instancia GoogleSearchTool
                .build();


        // Define el CodingAgent
        LlmAgent codingAgent =
            LlmAgent.builder()
                .model(MODEL_ID)
                .name("CodeAgent")
                .instruction("You're a specialist in Code Execution")
                .tools(new BuiltInCodeExecutionTool()) // Instancia BuiltInCodeExecutionTool
                .build();

        // Define el RootAgent, que usa AgentTool.create() para envolver SearchAgent y CodingAgent
        BaseAgent rootAgent =
            LlmAgent.builder()
                .name("RootAgent")
                .model(MODEL_ID)
                .description("Root Agent")
                .tools(
                    AgentTool.create(searchAgent), // Usa el método create
                    AgentTool.create(codingAgent)   // Usa el método create
                 )
                .build();

        // Nota: Este ejemplo solo demuestra las definiciones de agentes.
        // Para ejecutar estos agentes, necesitarías integrarlos con un Runner y SessionService,
        // similar a los ejemplos anteriores.
        System.out.println("Agents defined successfully:");
        System.out.println("  Root Agent: " + rootAgent.name());
        System.out.println("  Search Agent (nested): " + searchAgent.name());
        System.out.println("  Code Agent (nested): " + codingAgent.name());
      }
    }
    ```

### Solución alternativa #2: bypass_multi_tools_limit

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python</span><span class="lst-java">Java</span>
</div>

ADK Python tiene una solución alternativa incorporada que evita esta limitación para
`GoogleSearchTool` y `VertexAiSearchTool` (usa `bypass_multi_tools_limit=True` para habilitarla),
como se muestra en el
[built_in_multi_tools](https://github.com/google/adk-python/tree/main/contributing/samples/built_in_multi_tools).
agente de ejemplo.

!!! warning

    Las herramientas incorporadas no pueden ser usadas dentro de un sub-agente, con la excepción de
    `GoogleSearchTool` y `VertexAiSearchTool` en ADK Python debido a la
    solución alternativa mencionada arriba.

Por ejemplo, el siguiente enfoque que usa herramientas incorporadas dentro de sub-agentes
**no está soportado**:

=== "Python"

    ```py
    url_context_agent = Agent(
        model='gemini-2.5-flash',
        name='UrlContextAgent',
        instruction="""
        You're a specialist in URL Context
        """,
        tools=[url_context],
    )
    coding_agent = Agent(
        model='gemini-2.5-flash',
        name='CodeAgent',
        instruction="""
        You're a specialist in Code Execution
        """,
        code_executor=BuiltInCodeExecutor(),
    )
    root_agent = Agent(
        name="RootAgent",
        model="gemini-2.5-flash",
        description="Root Agent",
        sub_agents=[
            url_context_agent,
            coding_agent
        ],
    )
    ```

=== "Java"

    ```java
    LlmAgent searchAgent =
        LlmAgent.builder()
            .model("gemini-2.5-flash")
            .name("SearchAgent")
            .instruction("You're a specialist in Google Search")
            .tools(new GoogleSearchTool())
            .build();

    LlmAgent codingAgent =
        LlmAgent.builder()
            .model("gemini-2.5-flash")
            .name("CodeAgent")
            .instruction("You're a specialist in Code Execution")
            .tools(new BuiltInCodeExecutionTool())
            .build();


    LlmAgent rootAgent =
        LlmAgent.builder()
            .name("RootAgent")
            .model("gemini-2.5-flash")
            .description("Root Agent")
            .subAgents(searchAgent, codingAgent) // No soportado, ya que los sub-agentes usan herramientas incorporadas.
            .build();
    ```