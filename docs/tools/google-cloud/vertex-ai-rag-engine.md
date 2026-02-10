---
catalog_title: Vertex AI RAG Engine
catalog_description: Perform private data retrieval using Vertex AI RAG Engine
catalog_icon: /assets/tools-vertex-ai.png
---

# Herramienta Vertex AI RAG Engine para ADK

<div class="language-support-tag">
  <span class="lst-supported">Compatible con ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-java">Java v0.2.0</span>
</div>

La herramienta `vertex_ai_rag_retrieval` permite al agente realizar recuperación de datos privados usando Vertex
AI RAG Engine.

Cuando utilices grounding con Vertex AI RAG Engine, necesitas preparar un corpus RAG de antemano.
Por favor consulta el [ejemplo de agente RAG ADK](https://github.com/google/adk-samples/blob/main/python/agents/RAG/rag/shared_libraries/prepare_corpus_and_data.py) o la [página de Vertex AI RAG Engine](https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/rag-quickstart) para configurarlo.

!!! warning "Advertencia: Limitación de una sola herramienta por agente"

    Esta herramienta solo puede ser usada ***por sí misma*** dentro de una instancia de agente.
    Para más información sobre esta limitación y soluciones alternativas, consulta
    [Limitaciones para las herramientas de ADK](/tools/limitations/).

=== "Python"

    ```py
    --8<-- "examples/python/snippets/tools/built-in-tools/vertexai_rag_engine.py"
    ```