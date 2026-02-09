# Agentes paralelos

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.2.0</span>
</div>

El `ParallelAgent` es un [agente de flujo de trabajo](index.md) que ejecuta sus sub-agentes *concurrentemente*. Esto acelera dramáticamente los flujos de trabajo donde las tareas pueden realizarse de manera independiente.

Usa `ParallelAgent` cuando: Para escenarios que priorizan la velocidad e involucran tareas independientes e intensivas en recursos, un `ParallelAgent` facilita la ejecución paralela eficiente. **Cuando los sub-agentes operan sin dependencias, sus tareas pueden realizarse concurrentemente**, reduciendo significativamente el tiempo total de procesamiento.

Al igual que con otros [agentes de flujo de trabajo](index.md), el `ParallelAgent` no está impulsado por un LLM, y por lo tanto es determinista en cómo se ejecuta. Dicho esto, los agentes de flujo de trabajo solo se preocupan por su ejecución (es decir, ejecutar sub-agentes en paralelo), y no por su lógica interna; las herramientas o sub-agentes de un agente de flujo de trabajo pueden o no utilizar LLMs.

### Ejemplo

Este enfoque es particularmente beneficioso para operaciones como la recuperación de datos de múltiples fuentes o cálculos pesados, donde la paralelización produce ganancias sustanciales de rendimiento. Importante destacar que esta estrategia asume que no hay necesidad inherente de estado compartido o intercambio directo de información entre los agentes que se ejecutan concurrentemente.

### Cómo funciona

Cuando se llama al método `run_async()` del `ParallelAgent`:

1. **Ejecución Concurrente:** Inicia el método `run_async()` de *cada* sub-agente presente en la lista `sub_agents` *concurrentemente*. Esto significa que todos los agentes comienzan a ejecutarse (aproximadamente) al mismo tiempo.
2. **Ramas Independientes:** Cada sub-agente opera en su propia rama de ejecución. ***No* hay compartición automática del historial de conversación o estado entre estas ramas** durante la ejecución.
3. **Recolección de Resultados:** El `ParallelAgent` gestiona la ejecución paralela y, típicamente, proporciona una forma de acceder a los resultados de cada sub-agente después de que hayan completado (por ejemplo, a través de una lista de resultados o eventos). El orden de los resultados puede no ser determinista.

### Ejecución Independiente y Gestión del Estado

Es *crucial* entender que los sub-agentes dentro de un `ParallelAgent` se ejecutan independientemente. Si *necesitas* comunicación o compartición de datos entre estos agentes, debes implementarlo explícitamente. Los enfoques posibles incluyen:

* **`InvocationContext` Compartido:** Podrías pasar un objeto `InvocationContext` compartido a cada sub-agente. Este objeto podría actuar como un almacén de datos compartido. Sin embargo, necesitarías gestionar el acceso concurrente a este contexto compartido cuidadosamente (por ejemplo, usando bloqueos) para evitar condiciones de carrera.
* **Gestión de Estado Externa:** Usa una base de datos externa, cola de mensajes u otro mecanismo para gestionar el estado compartido y facilitar la comunicación entre agentes.
* **Post-Procesamiento:** Recolecta resultados de cada rama, y luego implementa lógica para coordinar datos posteriormente.

![Parallel Agent](../../assets/parallel-agent.png){: width="600"}

### Ejemplo Completo: Investigación Web Paralela

Imagina investigar múltiples temas simultáneamente:

1. **Agente Investigador 1:** Un `LlmAgent` que investiga "fuentes de energía renovable."
2. **Agente Investigador 2:** Un `LlmAgent` que investiga "tecnología de vehículos eléctricos."
3. **Agente Investigador 3:** Un `LlmAgent` que investiga "métodos de captura de carbono."

    ```py
    ParallelAgent(sub_agents=[ResearcherAgent1, ResearcherAgent2, ResearcherAgent3])
    ```

Estas tareas de investigación son independientes. Usar un `ParallelAgent` les permite ejecutarse concurrentemente, potencialmente reduciendo el tiempo total de investigación significativamente en comparación con ejecutarlas secuencialmente. Los resultados de cada agente se recolectarían por separado después de que terminen.

???+ "Código Completo"

    === "Python"
        ```py
         --8<-- "examples/python/snippets/agents/workflow-agents/parallel_agent_web_research.py:init"
        ```

    === "Typescript"
        ```typescript
         --8<-- "examples/typescript/snippets/agents/workflow-agents/parallel_agent_web_research.ts:init"
        ```

    === "Go"
        ```go
         --8<-- "examples/go/snippets/agents/workflow-agents/parallel/main.go:init"
        ```

    === "Java"
        ```java
         --8<-- "examples/java/snippets/src/main/java/agents/workflow/ParallelResearchPipeline.java:full_code"
        ```