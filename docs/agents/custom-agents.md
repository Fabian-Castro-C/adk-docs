# Agentes personalizados

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Los agentes personalizados proporcionan la máxima flexibilidad en ADK, permitiéndote definir **lógica de orquestación arbitraria** heredando directamente de `BaseAgent` e implementando tu propio flujo de control. Esto va más allá de los patrones predefinidos de `SequentialAgent`, `LoopAgent` y `ParallelAgent`, permitiéndote construir flujos de trabajo agénticos altamente específicos y complejos.

!!! warning "Concepto Avanzado"

    Construir agentes personalizados implementando directamente `_run_async_impl` (o su equivalente en otros lenguajes) proporciona un control poderoso pero es más complejo que usar los tipos predefinidos `LlmAgent` o `WorkflowAgent` estándar. Recomendamos comprender esos tipos de agentes fundamentales primero antes de abordar la lógica de orquestación personalizada.

## Introducción: Más Allá de los Flujos de Trabajo Predefinidos

### ¿Qué es un Agente Personalizado?

Un Agente Personalizado es esencialmente cualquier clase que crees que hereda de `google.adk.agents.BaseAgent` e implementa su lógica de ejecución central dentro del método asíncrono `_run_async_impl`. Tienes control completo sobre cómo este método llama a otros agentes (sub-agentes), gestiona el estado y maneja eventos.

!!! Note
    El nombre específico del método para implementar la lógica asíncrona central de un agente puede variar ligeramente según el lenguaje del SDK (por ejemplo, `runAsyncImpl` en Java, `_run_async_impl` en Python, o `runAsyncImpl` en TypeScript). Consulta la documentación de la API específica del lenguaje para más detalles.

### ¿Por Qué Usarlos?

Mientras que los [Agentes de Flujo de Trabajo](workflow-agents/index.md) estándar (`SequentialAgent`, `LoopAgent`, `ParallelAgent`) cubren patrones de orquestación comunes, necesitarás un agente Personalizado cuando tus requisitos incluyan:

* **Lógica Condicional:** Ejecutar diferentes sub-agentes o tomar diferentes rutas basándose en condiciones en tiempo de ejecución o los resultados de pasos anteriores.
* **Gestión de Estado Compleja:** Implementar lógica intrincada para mantener y actualizar el estado a lo largo del flujo de trabajo más allá del simple paso secuencial.
* **Integraciones Externas:** Incorporar llamadas a APIs externas, bases de datos o bibliotecas personalizadas directamente dentro del control de flujo de orquestación.
* **Selección Dinámica de Agentes:** Elegir qué sub-agente(s) ejecutar a continuación basándose en la evaluación dinámica de la situación o entrada.
* **Patrones de Flujo de Trabajo Únicos:** Implementar lógica de orquestación que no se ajusta a las estructuras estándar secuenciales, paralelas o de bucle.


![intro_components.png](../assets/custom-agent-flow.png)


## Implementando Lógica Personalizada:

El núcleo de cualquier agente personalizado es el método donde defines su comportamiento asíncrono único. Este método te permite orquestar sub-agentes y gestionar el flujo de ejecución.

=== "Python"

      El corazón de cualquier agente personalizado es el método `_run_async_impl`. Aquí es donde defines su comportamiento único.

      * **Firma:** `async def _run_async_impl(self, ctx: InvocationContext) -> AsyncGenerator[Event, None]:`
      * **Generador Asíncrono:** Debe ser una función `async def` y retornar un `AsyncGenerator`. Esto le permite hacer `yield` de eventos producidos por sub-agentes o su propia lógica de vuelta al ejecutor.
      * **`ctx` (InvocationContext):** Proporciona acceso a información crucial en tiempo de ejecución, más importante aún `ctx.session.state`, que es la forma principal de compartir datos entre pasos orquestados por tu agente personalizado.

=== "TypeScript"

    El corazón de cualquier agente personalizado es el método `runAsyncImpl`. Aquí es donde defines su comportamiento único.

    *   **Firma:** `async* runAsyncImpl(ctx: InvocationContext): AsyncGenerator<Event, void, undefined>`
    *   **Generador Asíncrono:** Debe ser una función generadora `async` (`async*`).
    *   **`ctx` (InvocationContext):** Proporciona acceso a información crucial en tiempo de ejecución, más importante aún `ctx.session.state`, que es la forma principal de compartir datos entre pasos orquestados por tu agente personalizado.

=== "Go"

    En Go, implementas el método `Run` como parte de una estructura que satisface la interfaz `agent.Agent`. La lógica real es típicamente un método en la estructura de tu agente personalizado.

    *   **Firma:** `Run(ctx agent.InvocationContext) iter.Seq2[*session.Event, error]`
    *   **Iterador:** El método `Run` retorna un iterador (`iter.Seq2`) que produce eventos y errores. Esta es la forma estándar de manejar resultados en streaming de la ejecución de un agente.
    *   **`ctx` (InvocationContext):** El `agent.InvocationContext` proporciona acceso a la sesión, incluyendo el estado, y otra información crucial en tiempo de ejecución.
    *   **Estado de la Sesión:** Puedes acceder al estado de la sesión a través de `ctx.Session().State()`.

=== "Java"

    El corazón de cualquier agente personalizado es el método `runAsyncImpl`, que sobrescribes desde `BaseAgent`.

    *   **Firma:** `protected Flowable<Event> runAsyncImpl(InvocationContext ctx)`
    *   **Flujo Reactivo (`Flowable`):** Debe retornar un `io.reactivex.rxjava3.core.Flowable<Event>`. Este `Flowable` representa un flujo de eventos que será producido por la lógica del agente personalizado, a menudo combinando o transformando múltiples `Flowable` de sub-agentes.
    *   **`ctx` (InvocationContext):** Proporciona acceso a información crucial en tiempo de ejecución, más importante aún `ctx.session().state()`, que es un `java.util.concurrent.ConcurrentMap<String, Object>`. Esta es la forma principal de compartir datos entre pasos orquestados por tu agente personalizado.

**Capacidades Clave dentro del Método Asíncrono Central:**

=== "Python"

    1. **Llamando a Sub-Agentes:** Invocas sub-agentes (que típicamente se almacenan como atributos de instancia como `self.my_llm_agent`) usando su método `run_async` y haces yield de sus eventos:

          ```python
          async for event in self.some_sub_agent.run_async(ctx):
              # Opcionalmente inspeccionar o registrar el evento
              yield event # Pasar el evento hacia arriba
          ```

    2. **Gestionando el Estado:** Lee y escribe en el diccionario de estado de la sesión (`ctx.session.state`) para pasar datos entre llamadas a sub-agentes o tomar decisiones:

          ```python
          # Leer datos establecidos por un agente anterior
          previous_result = ctx.session.state.get("some_key")

          # Tomar una decisión basada en el estado
          if previous_result == "some_value":
              # ... llamar a un sub-agente específico ...
          else:
              # ... llamar a otro sub-agente ...

          # Almacenar un resultado para un paso posterior (a menudo hecho a través del output_key de un sub-agente)
          # ctx.session.state["my_custom_result"] = "calculated_value"
          ```

    3. **Implementando Flujo de Control:** Usa construcciones estándar de Python (`if`/`elif`/`else`, bucles `for`/`while`, `try`/`except`) para crear flujos de trabajo sofisticados, condicionales o iterativos que involucren tus sub-agentes.

=== "TypeScript"

    1.  **Llamando a Sub-Agentes:** Invocas sub-agentes (que típicamente se almacenan como propiedades de instancia como `this.myLlmAgent`) usando su método `run` y haces yield de sus eventos:

        ```typescript
        for await (const event of this.someSubAgent.runAsync(ctx)) {
            // Opcionalmente inspeccionar o registrar el evento
            yield event; // Pasar el evento hacia arriba al ejecutor
        }
        ```

    2.  **Gestionando el Estado:** Lee y escribe en el objeto de estado de la sesión (`ctx.session.state`) para pasar datos entre llamadas a sub-agentes o tomar decisiones:

        ```typescript
        // Leer datos establecidos por un agente anterior
        const previousResult = ctx.session.state['some_key'];

        // Tomar una decisión basada en el estado
        if (previousResult === 'some_value') {
          // ... llamar a un sub-agente específico ...
        } else {
          // ... llamar a otro sub-agente ...
        }

        // Almacenar un resultado para un paso posterior (a menudo hecho a través del outputKey de un sub-agente)
        // ctx.session.state['my_custom_result'] = 'calculated_value';
        ```

    3. **Implementando Flujo de Control:** Usa construcciones estándar de TypeScript/JavaScript (`if`/`else`, bucles `for`/`while`, `try`/`catch`) para crear flujos de trabajo sofisticados, condicionales o iterativos que involucren tus sub-agentes.

=== "Go"

    1. **Llamando a Sub-Agentes:** Invocas sub-agentes llamando a su método `Run`.

          ```go
          // Ejemplo: Ejecutando un sub-agente y haciendo yield de sus eventos
          for event, err := range someSubAgent.Run(ctx) {
              if err != nil {
                  // Manejar o propagar el error
                  return
              }
              // Hacer yield del evento hacia arriba al llamador
              if !yield(event, nil) {
                return
              }
          }
          ```

    2. **Gestionando el Estado:** Lee y escribe en el estado de la sesión para pasar datos entre llamadas a sub-agentes o tomar decisiones.
          ```go
          // El `ctx` (`agent.InvocationContext`) se pasa directamente a la función `Run` de tu agente.
          // Leer datos establecidos por un agente anterior
          previousResult, err := ctx.Session().State().Get("some_key")
          if err != nil {
              // Manejar casos donde la clave podría no existir aún
          }

          // Tomar una decisión basada en el estado
          if val, ok := previousResult.(string); ok && val == "some_value" {
              // ... llamar a un sub-agente específico ...
          } else {
              // ... llamar a otro sub-agente ...
          }

          // Almacenar un resultado para un paso posterior
          if err := ctx.Session().State().Set("my_custom_result", "calculated_value"); err != nil {
              // Manejar error
          }
          ```

    3. **Implementando Flujo de Control:** Usa construcciones estándar de Go (`if`/`else`, bucles `for`/`switch`, goroutines, canales) para crear flujos de trabajo sofisticados, condicionales o iterativos que involucren tus sub-agentes.

=== "Java"

    1. **Llamando a Sub-Agentes:** Invocas sub-agentes (que típicamente se almacenan como atributos de instancia u objetos) usando su método de ejecución asíncrono y retornas sus flujos de eventos:

           Típicamente encadenas `Flowable`s de sub-agentes usando operadores de RxJava como `concatWith`, `flatMapPublisher` o `concatArray`.

           ```java
           // Ejemplo: Ejecutando un sub-agente
           // return someSubAgent.runAsync(ctx);

           // Ejemplo: Ejecutando sub-agentes secuencialmente
           Flowable<Event> firstAgentEvents = someSubAgent1.runAsync(ctx)
               .doOnNext(event -> System.out.println("Event from agent 1: " + event.id()));

           Flowable<Event> secondAgentEvents = Flowable.defer(() ->
               someSubAgent2.runAsync(ctx)
                   .doOnNext(event -> System.out.println("Event from agent 2: " + event.id()))
           );

           return firstAgentEvents.concatWith(secondAgentEvents);
           ```
           El `Flowable.defer()` se usa a menudo para etapas subsiguientes si su ejecución depende de la finalización o estado después de etapas previas.

    2. **Gestionando el Estado:** Lee y escribe en el estado de la sesión para pasar datos entre llamadas a sub-agentes o tomar decisiones. El estado de la sesión es un `java.util.concurrent.ConcurrentMap<String, Object>` obtenido mediante `ctx.session().state()`.

        ```java
        // Leer datos establecidos por un agente anterior
        Object previousResult = ctx.session().state().get("some_key");

        // Tomar una decisión basada en el estado
        if ("some_value".equals(previousResult)) {
            // ... lógica para incluir el Flowable de un sub-agente específico ...
        } else {
            // ... lógica para incluir el Flowable de otro sub-agente ...
        }

        // Almacenar un resultado para un paso posterior (a menudo hecho a través del output_key de un sub-agente)
        // ctx.session().state().put("my_custom_result", "calculated_value");
        ```

    3. **Implementando Flujo de Control:** Usa construcciones estándar del lenguaje (`if`/`else`, bucles, `try`/`catch`) combinadas con operadores reactivos (RxJava) para crear flujos de trabajo sofisticados.

          *   **Condicional:** `Flowable.defer()` para elegir a qué `Flowable` suscribirse basándose en una condición, o `filter()` si estás filtrando eventos dentro de un flujo.
          *   **Iterativo:** Operadores como `repeat()`, `retry()`, o estructurando tu cadena `Flowable` para llamar recursivamente partes de sí misma basándose en condiciones (a menudo gestionado con `flatMapPublisher` o `concatMap`).

## Gestionando Sub-Agentes y Estado

Típicamente, un agente personalizado orquesta otros agentes (como `LlmAgent`, `LoopAgent`, etc.).

* **Inicialización:** Usualmente pasas instancias de estos sub-agentes al constructor de tu agente personalizado y los almacenas como campos/atributos de instancia (por ejemplo, `this.story_generator = story_generator_instance` o `self.story_generator = story_generator_instance`). Esto los hace accesibles dentro de la lógica de ejecución asíncrona central del agente personalizado (como el método `_run_async_impl`).
* **Lista de Sub Agentes:** Al inicializar el `BaseAgent` usando su constructor `super()`, debes pasar una lista de `sub agents`. Esta lista le dice al framework ADK sobre los agentes que son parte de la jerarquía inmediata de este agente personalizado. Es importante para características del framework como gestión del ciclo de vida, introspección y potencialmente capacidades de enrutamiento futuras, incluso si tu lógica de ejecución central (`_run_async_impl`) llama a los agentes directamente a través de `self.xxx_agent`. Incluye los agentes que tu lógica personalizada invoca directamente en el nivel superior.
* **Estado:** Como se mencionó, `ctx.session.state` es la forma estándar en que los sub-agentes (especialmente `LlmAgent`s usando `output key`) comunican resultados de vuelta al orquestador y cómo el orquestador pasa las entradas necesarias hacia abajo.

## Ejemplo de Patrón de Diseño: `StoryFlowAgent`

Ilustremos el poder de los agentes personalizados con un ejemplo de patrón: un flujo de trabajo de generación de contenido multi-etapa con lógica condicional.

**Objetivo:** Crear un sistema que genere una historia, la refine iterativamente a través de crítica y revisión, realice verificaciones finales y, crucialmente, *regenere la historia si la verificación de tono final falla*.

**¿Por Qué Personalizado?** El requisito central que impulsa la necesidad de un agente personalizado aquí es la **regeneración condicional basada en la verificación de tono**. Los agentes de flujo de trabajo estándar no tienen ramificación condicional incorporada basada en el resultado de la tarea de un sub-agente. Necesitamos lógica personalizada (`if tone == "negative": ...`) dentro del orquestador.

---

### Parte 1: Inicialización simplificada del agente personalizado { #part-1-simplified-custom-agent-initialization }

=== "Python"

    Definimos el `StoryFlowAgent` heredando de `BaseAgent`. En `__init__`, almacenamos los sub-agentes necesarios (pasados) como atributos de instancia y le decimos al framework `BaseAgent` sobre los agentes de nivel superior que este agente personalizado orquestará directamente.

    ```python
    --8<-- "examples/python/snippets/agents/custom-agent/storyflow_agent.py:init"
    ```

=== "TypeScript"

    Definimos el `StoryFlowAgent` extendiendo `BaseAgent`. En su constructor:
    1.  Creamos cualquier agente compuesto interno (como `LoopAgent` o `SequentialAgent`).
    2.  Pasamos la lista de todos los sub-agentes de nivel superior al constructor `super()`.
    3.  Almacenamos los sub-agentes (pasados o creados internamente) como propiedades de instancia (por ejemplo, `this.storyGenerator`) para que puedan ser accedidos en la lógica `runImpl` personalizada.

    ```typescript
    --8<-- "examples/typescript/snippets/agents/custom-agent/storyflow_agent.ts:init"
    ```

=== "Go"

    Definimos la estructura `StoryFlowAgent` y un constructor. En el constructor, almacenamos los sub-agentes necesarios y le decimos al framework `BaseAgent` sobre los agentes de nivel superior que este agente personalizado orquestará directamente.

    ```go
    --8<-- "examples/go/snippets/agents/custom-agent/storyflow_agent.go:init"
    ```

=== "Java"

    Definimos el `StoryFlowAgentExample` extendiendo `BaseAgent`. En su **constructor**, almacenamos las instancias de sub-agentes necesarias (pasadas como parámetros) como campos de instancia. Estos sub-agentes de nivel superior, que este agente personalizado orquestará directamente, también se pasan al constructor `super` de `BaseAgent` como una lista.

    ```java
    --8<-- "examples/java/snippets/src/main/java/agents/StoryFlowAgentExample.java:init"
    ```

---

### Parte 2: Definiendo la Lógica de Ejecución Personalizada { #part-2-defining-the-custom-execution-logic }

=== "Python"

    Este método orquesta los sub-agentes usando async/await estándar de Python y flujo de control.

    ```python
    --8<-- "examples/python/snippets/agents/custom-agent/storyflow_agent.py:executionlogic"
    ```
    **Explicación de la Lógica:**

    1. El `story_generator` inicial se ejecuta. Se espera que su salida esté en `ctx.session.state["current_story"]`.
    2. El `loop_agent` se ejecuta, que internamente llama al `critic` y `reviser` secuencialmente por `max_iterations` veces. Leen/escriben `current_story` y `criticism` desde/hacia el estado.
    3. El `sequential_agent` se ejecuta, llamando a `grammar_check` y luego `tone_check`, leyendo `current_story` y escribiendo `grammar_suggestions` y `tone_check_result` al estado.
    4. **Parte Personalizada:** La declaración `if` verifica el `tone_check_result` del estado. Si es "negative", el `story_generator` es llamado *nuevamente*, sobrescribiendo el `current_story` en el estado. De lo contrario, el flujo termina.

=== "TypeScript"

    El método `runImpl` orquesta los sub-agentes usando `async`/`await` estándar de TypeScript y flujo de control. El `runLiveImpl` también se agrega para manejar escenarios de transmisión en vivo.

    ```typescript
    --8<-- "examples/typescript/snippets/agents/custom-agent/storyflow_agent.ts:executionlogic"
    ```
    **Explicación de la Lógica:**

    1.  El `storyGenerator` inicial se ejecuta. Se espera que su salida esté en `ctx.session.state['current_story']`.
    2.  El `loopAgent` se ejecuta, que internamente llama al `critic` y `reviser` secuencialmente por `maxIterations` veces. Leen/escriben `current_story` y `criticism` desde/hacia el estado.
    3.  El `sequentialAgent` se ejecuta, llamando a `grammarCheck` y luego `toneCheck`, leyendo `current_story` y escribiendo `grammar_suggestions` y `tone_check_result` al estado.
    4.  **Parte Personalizada:** La declaración `if` verifica el `tone_check_result` del estado. Si es "negative", el `storyGenerator` es llamado *nuevamente*, sobrescribiendo el `current_story` en el estado. De lo contrario, el flujo termina.

=== "Go"

    El método `Run` orquesta los sub-agentes llamando a sus respectivos métodos `Run` en un bucle y haciendo yield de sus eventos.

    ```go
    --8<-- "examples/go/snippets/agents/custom-agent/storyflow_agent.go:executionlogic"
    ```
    **Explicación de la Lógica:**

    1. El `storyGenerator` inicial se ejecuta. Se espera que su salida esté en el estado de la sesión bajo la clave `"current_story"`.
    2. El `revisionLoopAgent` se ejecuta, que internamente llama al `critic` y `reviser` secuencialmente por `max_iterations` veces. Leen/escriben `current_story` y `criticism` desde/hacia el estado.
    3. El `postProcessorAgent` se ejecuta, llamando a `grammar_check` y luego `tone_check`, leyendo `current_story` y escribiendo `grammar_suggestions` y `tone_check_result` al estado.
    4. **Parte Personalizada:** El código verifica el `tone_check_result` del estado. Si es "negative", el `story_generator` es llamado *nuevamente*, sobrescribiendo el `current_story` en el estado. De lo contrario, el flujo termina.

=== "Java"

    El método `runAsyncImpl` orquesta los sub-agentes usando flujos Flowable de RxJava y operadores para flujo de control asíncrono.

    ```java
    --8<-- "examples/java/snippets/src/main/java/agents/StoryFlowAgentExample.java:executionlogic"
    ```
    **Explicación de la Lógica:**

    1. El Flowable inicial `storyGenerator.runAsync(invocationContext)` se ejecuta. Se espera que su salida esté en `invocationContext.session().state().get("current_story")`.
    2. El Flowable del `loopAgent` se ejecuta después (debido a `Flowable.concatArray` y `Flowable.defer`). El LoopAgent internamente llama a los sub-agentes `critic` y `reviser` secuencialmente hasta `maxIterations`. Leen/escriben `current_story` y `criticism` desde/hacia el estado.
    3. Luego, el Flowable del `sequentialAgent` se ejecuta. Llama a `grammar_check` y luego `tone_check`, leyendo `current_story` y escribiendo `grammar_suggestions` y `tone_check_result` al estado.
    4. **Parte Personalizada:** Después de que el sequentialAgent completa, la lógica dentro de un `Flowable.defer` verifica el "tone_check_result" de `invocationContext.session().state()`. Si es "negative", el Flowable del `storyGenerator` es *condicionalmente concatenado* y ejecutado nuevamente, sobrescribiendo "current_story". De lo contrario, se usa un Flowable vacío, y el flujo de trabajo general procede a completarse.

---

### Parte 3: Definiendo los Sub-Agentes LLM { #part-3-defining-the-llm-sub-agents }

Estas son definiciones estándar de `LlmAgent`, responsables de tareas específicas. Su parámetro `output key` es crucial para colocar resultados en el `session.state` donde otros agentes o el orquestador personalizado puedan acceder a ellos.

!!! tip "Inyección Directa de Estado en Instrucciones"
    Observa la instrucción del `story_generator`. La sintaxis `{var}` es un marcador de posición. Antes de que la instrucción sea enviada al LLM, el framework ADK reemplaza automáticamente (Ejemplo:`{topic}`) con el valor de `session.state['topic']`. Esta es la forma recomendada de proporcionar contexto a un agente, usando plantillas en las instrucciones. Para más detalles, consulta la [documentación de Estado](../sessions/state.md#accessing-session-state-in-agent-instructions).

=== "Python"

    ```python
    GEMINI_2_FLASH = "gemini-2.0-flash" # Definir constante de modelo
    --8<-- "examples/python/snippets/agents/custom-agent/storyflow_agent.py:llmagents"
    ```

=== "TypeScript"

    ```typescript
    --8<-- "examples/typescript/snippets/agents/custom-agent/storyflow_agent.ts:llmagents"
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/agents/custom-agent/storyflow_agent.go:llmagents"
    ```

=== "Java"

    ```java
    --8<-- "examples/java/snippets/src/main/java/agents/StoryFlowAgentExample.java:llmagents"
    ```

---

### Parte 4: Instanciando y Ejecutando el agente personalizado { #part-4-instantiating-and-running-the-custom-agent }

Finalmente, instancias tu `StoryFlowAgent` y usas el `Runner` como de costumbre.

=== "Python"

    ```python
    --8<-- "examples/python/snippets/agents/custom-agent/storyflow_agent.py:story_flow_agent"
    ```

=== "TypeScript"

    ```typescript
    --8<-- "examples/typescript/snippets/agents/custom-agent/storyflow_agent.ts:story_flow_agent"
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/agents/custom-agent/storyflow_agent.go:story_flow_agent"
    ```

=== "Java"

    ```java
    --8<-- "examples/java/snippets/src/main/java/agents/StoryFlowAgentExample.java:story_flow_agent"
    ```

*(Nota: El código ejecutable completo, incluyendo imports y lógica de ejecución, se puede encontrar enlazado abajo.)*

---

## Ejemplo de Código Completo

???+ "Storyflow Agent"

    === "Python"

        ```python
        # Código ejecutable completo para el ejemplo de StoryFlowAgent
        --8<-- "examples/python/snippets/agents/custom-agent/storyflow_agent.py"
        ```

    === "TypeScript"

        ```typescript
        // Código ejecutable completo para el ejemplo de StoryFlowAgent

        --8<-- "examples/typescript/snippets/agents/custom-agent/storyflow_agent.ts"
        ```

    === "Go"

        ```go
        # Código ejecutable completo para el ejemplo de StoryFlowAgent
        --8<-- "examples/go/snippets/agents/custom-agent/storyflow_agent.go:full_code"
        ```

    === "Java"

        ```java
        # Código ejecutable completo para el ejemplo de StoryFlowAgent
        --8<-- "examples/java/snippets/src/main/java/agents/StoryFlowAgentExample.java:full_code"
        ```