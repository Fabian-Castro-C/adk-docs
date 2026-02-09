---
catalog_title: Code Execution
catalog_description: Execute code and debug using Gemini models
catalog_icon: /adk-docs/assets/tools-gemini-spark.svg
---

# Ejecución de Código con Gemini API

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-java">Java v0.2.0</span>
</div>

La herramienta `built_in_code_execution` permite al agente ejecutar código,
específicamente cuando se usan modelos Gemini 2 y superiores. Esto permite al modelo
realizar tareas como cálculos, manipulación de datos o ejecutar pequeños scripts.

!!! warning "Advertencia: Limitación de una herramienta por agente"

    Esta herramienta solo puede usarse ***por sí sola*** dentro de una instancia de agente.
    Para más información sobre esta limitación y soluciones alternativas, consulta
    [Limitaciones para las herramientas de ADK](/adk-docs/tools/limitations/#one-tool-one-agent).

=== "Python"

    ```py
    --8<-- "examples/python/snippets/tools/built-in-tools/code_execution.py"
    ```

=== "Java"

    ```java
    --8<-- "examples/java/snippets/src/main/java/tools/CodeExecutionAgentApp.java:full_code"
    ```