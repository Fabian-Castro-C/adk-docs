# Agentes de bucle

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.2.0</span>
</div>

El `LoopAgent` es un agente de flujo de trabajo que ejecuta sus sub-agentes en un bucle (es decir, iterativamente). **_Ejecuta repetidamente_ una secuencia de agentes** durante un número especificado de iteraciones o hasta que se cumpla una condición de terminación.

Usa el `LoopAgent` cuando tu flujo de trabajo implique repetición o refinamiento iterativo, como revisar código.

### Ejemplo

* Quieres construir un agente que pueda generar imágenes de comida, pero a veces cuando quieres generar un número específico de elementos (por ejemplo, 5 plátanos), genera un número diferente de esos elementos en la imagen, como una imagen de 7 plátanos. Tienes dos herramientas: `Generate Image`, `Count Food Items`. Como quieres seguir generando imágenes hasta que genere correctamente el número especificado de elementos, o después de un cierto número de iteraciones, debes construir tu agente usando un `LoopAgent`.

Al igual que con otros [agentes de flujo de trabajo](index.md), el `LoopAgent` no está impulsado por un LLM, y por lo tanto es determinista en cómo se ejecuta. Dicho esto, los agentes de flujo de trabajo solo se preocupan por su ejecución, como en un bucle, y no por su lógica interna; las herramientas o sub-agentes de un agente de flujo de trabajo pueden o no utilizar LLMs.

### Cómo Funciona

Cuando se llama al método `Run Async` del `LoopAgent`, realiza las siguientes acciones:

1. **Ejecución de Sub-Agentes:** Itera a través de la lista de Sub Agentes _en orden_. Para _cada_ sub-agente, llama al método `Run Async` del agente.
2. **Verificación de Terminación:**

    _Crucialmente_, el `LoopAgent` en sí mismo _no_ decide inherentemente cuándo dejar de hacer bucles. _Debes_ implementar un mecanismo de terminación para prevenir bucles infinitos. Las estrategias comunes incluyen:

    * **Máximo de Iteraciones**: Establece un número máximo de iteraciones en el `LoopAgent`. **El bucle terminará después de esa cantidad de iteraciones**.
    * **Escalamiento desde sub-agente**: Diseña uno o más sub-agentes para evaluar una condición (por ejemplo, "¿Es la calidad del documento lo suficientemente buena?", "¿Se ha alcanzado un consenso?"). Si se cumple la condición, el sub-agente puede señalar la terminación (por ejemplo, lanzando un evento personalizado, estableciendo una bandera en un contexto compartido, o retornando un valor específico).

![Loop Agent](../../assets/loop-agent.png)

### Ejemplo Completo: Mejora Iterativa de Documentos

Imagina un escenario donde quieres mejorar iterativamente un documento:

* **Agente Escritor:** Un `LlmAgent` que genera o refina un borrador sobre un tema.
* **Agente Crítico:** Un `LlmAgent` que critica el borrador, identificando áreas de mejora.

    ```py
    LoopAgent(sub_agents=[WriterAgent, CriticAgent], max_iterations=5)
    ```

En esta configuración, el `LoopAgent` gestionaría el proceso iterativo. El `CriticAgent` podría ser **diseñado para retornar una señal "STOP" cuando el documento alcance un nivel de calidad satisfactorio**, previniendo más iteraciones. Alternativamente, el parámetro `max iterations` podría usarse para limitar el proceso a un número fijo de ciclos, o se podría implementar lógica externa para tomar decisiones de parada. El **bucle se ejecutaría como máximo cinco veces**, asegurando que el refinamiento iterativo no continúe indefinidamente.

???+ "Full Code"

    === "Python"
        ```py
        --8<-- "examples/python/snippets/agents/workflow-agents/loop_agent_doc_improv_agent.py:init"
        ```

    === "Typescript"
        ```typescript
        --8<-- "examples/typescript/snippets/agents/workflow-agents/loop_agent_doc_improv_agent.ts:init"
        ```

    === "Go"
        ```go
        --8<-- "examples/go/snippets/agents/workflow-agents/loop/main.go:init"
        ```

    === "Java"
        ```java
        --8<-- "examples/java/snippets/src/main/java/agents/workflow/LoopAgentExample.java:init"
        ```