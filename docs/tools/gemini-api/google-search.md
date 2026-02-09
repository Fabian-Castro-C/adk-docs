---
catalog_title: Google Search
catalog_description: Perform web searches using Google Search with Gemini
catalog_icon: /adk-docs/assets/tools-google-search.png
---

# Herramienta Google Search para ADK

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.2.0</span>
</div>

La herramienta `google_search` permite al agente realizar búsquedas web usando Google Search. La herramienta `google_search` solo es compatible con modelos Gemini 2. Para más detalles sobre la herramienta, consulta [Entendiendo la fundamentación de Google Search](/adk-docs/grounding/google_search_grounding/).

!!! warning "Requisitos adicionales al usar la herramienta `google_search`"
    Cuando usas fundamentación con Google Search, y recibes sugerencias de búsqueda en tu respuesta, debes mostrar las sugerencias de búsqueda en producción y en tus aplicaciones.
    Para más información sobre fundamentación con Google Search, consulta la documentación de fundamentación con Google Search para [Google AI Studio](https://ai.google.dev/gemini-api/docs/grounding/search-suggestions) o [Vertex AI](https://cloud.google.com/vertex-ai/generative-ai/docs/grounding/grounding-search-suggestions). El código de UI (HTML) se devuelve en la respuesta de Gemini como `renderedContent`, y necesitarás mostrar el HTML en tu aplicación, de acuerdo con la política.

!!! warning "Advertencia: Limitación de una sola herramienta por agente"

    Esta herramienta solo puede ser usada ***por sí misma*** dentro de una instancia de agente.
    Para más información sobre esta limitación y soluciones alternativas, consulta
    [Limitaciones para herramientas ADK](/adk-docs/tools/limitations/#one-tool-one-agent).

=== "Python"

    ```py
    --8<-- "examples/python/snippets/tools/built-in-tools/google_search.py"
    ```

=== "TypeScript"

    ```typescript
    import {GOOGLE_SEARCH, LlmAgent} from '@google/adk';

    export const rootAgent = new LlmAgent({
      model: 'gemini-2.5-flash',
      name: 'root_agent',
      description:
          'an agent whose job it is to perform Google search queries and answer questions about the results.',
      instruction:
          'You are an agent whose job is to perform Google search queries and answer questions about the results.',
      tools: [GOOGLE_SEARCH],
    });
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/tools/built-in-tools/google_search.go"
    ```

=== "Java"

    ```java
    --8<-- "examples/java/snippets/src/main/java/tools/GoogleSearchAgentApp.java:full_code"
    ```