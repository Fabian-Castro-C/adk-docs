---
catalog_title: Vertex AI Search
catalog_description: Search across your private, configured data stores in Vertex AI Search
catalog_icon: /assets/tools-vertex-ai.png
---

# Herramienta Vertex AI Search para ADK

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span>
</div>

La herramienta `vertex_ai_search_tool` usa Google Cloud Vertex AI Search, permitiendo que el
agente busque en sus almacenes de datos privados y configurados (por ejemplo, documentos internos,
políticas de la empresa, bases de conocimiento). Esta herramienta integrada requiere que
proporcione el ID específico del almacén de datos durante la configuración. Para más detalles
sobre la herramienta, consulte
[Comprensión del grounding de Vertex AI Search](/grounding/vertex_ai_search_grounding/).

!!! warning "Advertencia: Limitación de una herramienta por agente"

    Esta herramienta solo puede usarse ***por sí misma*** dentro de una instancia de agente.
    Para más información sobre esta limitación y soluciones alternativas, consulte
    [Limitaciones para las herramientas de ADK](/tools/limitations/#one-tool-one-agent).

```py
--8<-- "examples/python/snippets/tools/built-in-tools/vertexai_search.py"
```