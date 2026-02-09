# Tipos de Callbacks

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

El framework proporciona diferentes tipos de callbacks que se activan en varias etapas de la ejecución de un agente. Comprender cuándo se dispara cada callback y qué contexto recibe es clave para usarlos de manera efectiva.

## Callbacks del Ciclo de Vida del Agente

Estos callbacks están disponibles en *cualquier* agente que herede de `BaseAgent` (incluyendo `LlmAgent`, `SequentialAgent`, `ParallelAgent`, `LoopAgent`, etc).

!!! Note
    Los nombres específicos de los métodos o tipos de retorno pueden variar ligeramente según el lenguaje del SDK (por ejemplo, retornar `None` en Python, retornar `Optional.empty()` o `Maybe.empty()` en Java). Consulte la documentación de la API específica del lenguaje para más detalles.

### Before Agent Callback

**Cuándo:** Se llama *inmediatamente antes* de que se ejecute el método `_run_async_impl` (o `_run_live_impl`) del agente. Se ejecuta después de que se crea el `InvocationContext` del agente pero *antes* de que comience su lógica principal.

**Propósito:** Ideal para configurar recursos o estado necesarios solo para la ejecución específica de este agente, realizar verificaciones de validación en el estado de la sesión (callback\_context.state) antes de que comience la ejecución, registrar el punto de entrada de la actividad del agente, o potencialmente modificar el contexto de invocación antes de que la lógica principal lo use.


??? "Code"
    === "Python"

        ```python
        --8<-- "examples/python/snippets/callbacks/before_agent_callback.py"
        ```

    === "Typescript"
        ```typescript
        --8<-- "examples/typescript/snippets/callbacks/before_agent_callback.ts"
        ```

    === "Go"

        ```go
        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:imports"


        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:before_agent_example"
        ```

    === "Java"

        ```java
        --8<-- "examples/java/snippets/src/main/java/callbacks/BeforeAgentCallbackExample.java:init"
        ```


**Nota sobre el Ejemplo de `before_agent_callback`:**

* **Qué Muestra:** Este ejemplo demuestra el `before_agent_callback`. Este callback se ejecuta *justo antes* de que comience la lógica de procesamiento principal del agente para una solicitud dada.
* **Cómo Funciona:** La función callback (`check_if_agent_should_run`) examina un indicador (`skip_llm_agent`) en el estado de la sesión.
    * Si el indicador es `True`, el callback retorna un objeto `types.Content`. Esto le dice al framework ADK que **omita** la ejecución principal del agente por completo y use el contenido retornado por el callback como la respuesta final.
    * Si el indicador es `False` (o no está configurado), el callback retorna `None` o un objeto vacío. Esto le dice al framework ADK que **continúe** con la ejecución normal del agente (llamando al LLM en este caso).
* **Resultado Esperado:** Verá dos escenarios:
    1. En la sesión *con* el estado `skip_llm_agent: True`, la llamada al LLM del agente se omite, y la salida proviene directamente del callback ("Agent... skipped...").
    2. En la sesión *sin* ese indicador de estado, el callback permite que el agente se ejecute, y ve la respuesta real del LLM (por ejemplo, "Hello!").
* **Comprendiendo los Callbacks:** Esto resalta cómo los callbacks `before_` actúan como **guardianes**, permitiéndole interceptar la ejecución *antes* de un paso importante y potencialmente prevenirlo basándose en verificaciones (como estado, validación de entrada, permisos).


### After Agent Callback

**Cuándo:** Se llama *inmediatamente después* de que el método `_run_async_impl` (o `_run_live_impl`) del agente se complete exitosamente. *No* se ejecuta si el agente fue omitido debido a que `before_agent_callback` retornó contenido o si `end_invocation` fue establecido durante la ejecución del agente.

**Propósito:** Útil para tareas de limpieza, validación post-ejecución, registrar la finalización de la actividad de un agente, modificar el estado final, o aumentar/reemplazar la salida final del agente.

??? "Code"
    === "Python"

        ```python
        --8<-- "examples/python/snippets/callbacks/after_agent_callback.py"
        ```

    === "Typescript"
        ```typescript
        --8<-- "examples/typescript/snippets/callbacks/after_agent_callback.ts"
        ```

    === "Go"

        ```go
        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:imports"


        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:after_agent_example"
        ```

    === "Java"

        ```java
        --8<-- "examples/java/snippets/src/main/java/callbacks/AfterAgentCallbackExample.java:init"
        ```


**Nota sobre el Ejemplo de `after_agent_callback`:**

* **Qué Muestra:** Este ejemplo demuestra el `after_agent_callback`. Este callback se ejecuta *justo después* de que la lógica de procesamiento principal del agente ha finalizado y producido su resultado, pero *antes* de que ese resultado se finalice y retorne.
* **Cómo Funciona:** La función callback (`modify_output_after_agent`) verifica un indicador (`add_concluding_note`) en el estado de la sesión.
    * Si el indicador es `True`, el callback retorna un *nuevo* objeto `types.Content`. Esto le dice al framework ADK que **reemplace** la salida original del agente con el contenido retornado por el callback.
    * Si el indicador es `False` (o no está configurado), el callback retorna `None` o un objeto vacío. Esto le dice al framework ADK que **use** la salida original generada por el agente.
*   **Resultado Esperado:** Verá dos escenarios:
    1. En la sesión *sin* el estado `add_concluding_note: True`, el callback permite que se use la salida original del agente ("Processing complete!").
    2. En la sesión *con* ese indicador de estado, el callback intercepta la salida original del agente y la reemplaza con su propio mensaje ("Concluding note added...").
* **Comprendiendo los Callbacks:** Esto resalta cómo los callbacks `after_` permiten **post-procesamiento** o **modificación**. Puede inspeccionar el resultado de un paso (la ejecución del agente) y decidir si dejarlo pasar, cambiarlo o reemplazarlo completamente basándose en su lógica.

## Callbacks de Interacción con LLM

Estos callbacks son específicos de `LlmAgent` y proporcionan ganchos alrededor de la interacción con el Modelo de Lenguaje Grande.

### Before Model Callback

**Cuándo:** Se llama justo antes de que la solicitud `generate_content_async` (o equivalente) se envíe al LLM dentro del flujo de un `LlmAgent`.

**Propósito:** Permite la inspección y modificación de la solicitud que va al LLM. Los casos de uso incluyen agregar instrucciones dinámicas, inyectar ejemplos few-shot basados en el estado, modificar la configuración del modelo, implementar barreras de protección (como filtros de profanidad), o implementar caché a nivel de solicitud.

**Efecto del Valor de Retorno:**
Si el callback retorna `None` (o un objeto `Maybe.empty()` en Java), el LLM continúa su flujo normal. Si el callback retorna un objeto `LlmResponse`, entonces la llamada al LLM es **omitida**. El `LlmResponse` retornado se usa directamente como si viniera del modelo. Esto es poderoso para implementar barreras de protección o caché.

??? "Code"
    === "Python"

        ```python
        --8<-- "examples/python/snippets/callbacks/before_model_callback.py"
        ```

    === "Typescript"
        ```typescript
        --8<-- "examples/typescript/snippets/callbacks/before_model_callback.ts"
        ```

    === "Go"

        ```go
        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:imports"


        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:before_model_example"
        ```

    === "Java"

        ```java
        --8<-- "examples/java/snippets/src/main/java/callbacks/BeforeModelCallbackExample.java:init"
        ```

### After Model Callback

**Cuándo:** Se llama justo después de que se recibe una respuesta (`LlmResponse`) del LLM, antes de que sea procesada más a fondo por el agente invocador.

**Propósito:** Permite la inspección o modificación de la respuesta cruda del LLM. Los casos de uso incluyen

* registrar salidas del modelo,
* reformatear respuestas,
* censurar información sensible generada por el modelo,
* analizar datos estructurados de la respuesta del LLM y almacenarlos en `callback_context.state`
* o manejar códigos de error específicos.

??? "Code"
    === "Python"

        ```python
        --8<-- "examples/python/snippets/callbacks/after_model_callback.py"
        ```

    === "Typescript"
        ```typescript
        --8<-- "examples/typescript/snippets/callbacks/after_model_callback.ts"
        ```

    === "Go"

        ```go
        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:imports"


        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:after_model_example"
        ```

    === "Java"

        ```java
        --8<-- "examples/java/snippets/src/main/java/callbacks/AfterModelCallbackExample.java:init"
        ```

## Callbacks de Ejecución de Herramientas

Estos callbacks también son específicos de `LlmAgent` y se activan alrededor de la ejecución de herramientas (incluyendo `FunctionTool`, `AgentTool`, etc.) que el LLM podría solicitar.

### Before Tool Callback

**Cuándo:** Se llama justo antes de que se invoque el método `run_async` de una herramienta específica, después de que el LLM ha generado una llamada de función para ella.

**Propósito:** Permite la inspección y modificación de los argumentos de la herramienta, realizar verificaciones de autorización antes de la ejecución, registrar intentos de uso de herramientas, o implementar caché a nivel de herramienta.

**Efecto del Valor de Retorno:**

1. Si el callback retorna `None` (o un objeto `Maybe.empty()` en Java), el método `run_async` de la herramienta se ejecuta con los `args` (potencialmente modificados).
2. Si se retorna un diccionario (o `Map` en Java), el método `run_async` de la herramienta es **omitido**. El diccionario retornado se usa directamente como el resultado de la llamada a la herramienta. Esto es útil para caché o sobrescribir el comportamiento de la herramienta.


??? "Code"
    === "Python"

        ```python
        --8<-- "examples/python/snippets/callbacks/before_tool_callback.py"
        ```

    === "Typescript"
        ```typescript
        --8<-- "examples/typescript/snippets/callbacks/before_tool_callback.ts"
        ```

    === "Go"

        ```go
        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:imports"
        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:tool_defs"
        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:before_tool_example"
        ```

    === "Java"

        ```java
        --8<-- "examples/java/snippets/src/main/java/callbacks/BeforeToolCallbackExample.java:init"
        ```


### After Tool Callback

**Cuándo:** Se llama justo después de que el método `run_async` de la herramienta se completa exitosamente.

**Propósito:** Permite la inspección y modificación del resultado de la herramienta antes de que se envíe de vuelta al LLM (potencialmente después de la sumarización). Útil para registrar resultados de herramientas, post-procesar o formatear resultados, o guardar partes específicas del resultado en el estado de la sesión.

**Efecto del Valor de Retorno:**

1. Si el callback retorna `None` (o un objeto `Maybe.empty()` en Java), se usa el `tool_response` original.
2. Si se retorna un nuevo diccionario, **reemplaza** el `tool_response` original. Esto permite modificar o filtrar el resultado visto por el LLM.

??? "Code"
    === "Python"

        ```python
        --8<-- "examples/python/snippets/callbacks/after_tool_callback.py"
        ```

    === "Typescript"
        ```typescript
        --8<-- "examples/typescript/snippets/callbacks/after_tool_callback.ts"
        ```

    === "Go"

        ```go
        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:imports"
        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:tool_defs"
        --8<-- "examples/go/snippets/callbacks/types_of_callbacks/main.go:after_tool_example"
        ```

    === "Java"

        ```java
        --8<-- "examples/java/snippets/src/main/java/callbacks/AfterToolCallbackExample.java:init"
        ```