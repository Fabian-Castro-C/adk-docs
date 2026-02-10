# Sistemas Multi-Agente en ADK

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

A medida que las aplicaciones agénticas crecen en complejidad, estructurarlas como un único agente monolítico puede volverse difícil de desarrollar, mantener y razonar. El Agent Development Kit (ADK) soporta la construcción de aplicaciones sofisticadas mediante la composición de múltiples instancias distintas de `BaseAgent` en un **Sistema Multi-Agente (MAS)**.

En ADK, un sistema multi-agente es una aplicación donde diferentes agentes, a menudo formando una jerarquía, colaboran o coordinan para lograr un objetivo mayor. Estructurar tu aplicación de esta manera ofrece ventajas significativas, incluyendo modularidad mejorada, especialización, reusabilidad, mantenibilidad, y la capacidad de definir flujos de control estructurados usando agentes de flujo de trabajo dedicados.

Puedes componer varios tipos de agentes derivados de `BaseAgent` para construir estos sistemas:

* **Agentes LLM:** Agentes impulsados por modelos de lenguaje grande. (Ver [Agentes LLM](llm-agents.md))
* **Agentes de Flujo de Trabajo:** Agentes especializados (`SequentialAgent`, `ParallelAgent`, `LoopAgent`) diseñados para gestionar el flujo de ejecución de sus sub-agentes. (Ver [Agentes de Flujo de Trabajo](workflow-agents/index.md))
* **Agentes personalizados:** Tus propios agentes que heredan de `BaseAgent` con lógica especializada no-LLM. (Ver [Agentes Personalizados](custom-agents.md))

Las siguientes secciones detallan las primitivas centrales de ADK—como la jerarquía de agentes, agentes de flujo de trabajo, y mecanismos de interacción—que te permiten construir y gestionar estos sistemas multi-agente de manera efectiva.

## 1. Primitivas de ADK para Composición de Agentes { #adk-primitives-for-agent-composition }

ADK proporciona bloques de construcción centrales—primitivas—que te permiten estructurar y gestionar interacciones dentro de tu sistema multi-agente.

!!! Note
    Los parámetros específicos o nombres de métodos para las primitivas pueden variar ligeramente según el lenguaje del SDK (por ejemplo, `sub_agents` en Python, `subAgents` en Java). Consulta la documentación de la API específica del lenguaje para más detalles.

### 1.1. Jerarquía de Agentes (Agente padre, Sub Agentes) { #agent-hierarchy-parent-agent-sub-agents }

La base para estructurar sistemas multi-agente es la relación padre-hijo definida en `BaseAgent`.

* **Estableciendo la Jerarquía:** Creas una estructura de árbol pasando una lista de instancias de agentes al argumento `sub_agents` al inicializar un agente padre. ADK automáticamente establece el atributo `parent_agent` en cada agente hijo durante la inicialización.
* **Regla de Padre Único:** Una instancia de agente solo puede ser agregada como sub-agente una vez. Intentar asignar un segundo padre resultará en un `ValueError`.
* **Importancia:** Esta jerarquía define el alcance para [Agentes de Flujo de Trabajo](#workflow-agents-as-orchestrators) e influye en los objetivos potenciales para Delegación Impulsada por LLM. Puedes navegar la jerarquía usando `agent.parent_agent` o encontrar descendientes usando `agent.find_agent(name)`.

=== "Python"

    ```python
    # Ejemplo Conceptual: Definiendo Jerarquía
    from google.adk.agents import LlmAgent, BaseAgent


    # Define agentes individuales
    greeter = LlmAgent(name="Greeter", model="gemini-2.0-flash")
    task_doer = BaseAgent(name="TaskExecutor") # Agente personalizado no-LLM


    # Crea agente padre y asigna hijos vía sub_agents
    coordinator = LlmAgent(
        name="Coordinator",
        model="gemini-2.0-flash",
        description="I coordinate greetings and tasks.",
        sub_agents=[ # Asigna sub_agents aquí
            greeter,
            task_doer
        ]
    )


    # El framework automáticamente establece:
    # assert greeter.parent_agent == coordinator
    # assert task_doer.parent_agent == coordinator
    ```

=== "Typescript"

    ```typescript
    // Ejemplo Conceptual: Definiendo Jerarquía
    import { LlmAgent, BaseAgent, InvocationContext } from '@google/adk';
    import type { Event, createEventActions } from '@google/adk';

    class TaskExecutorAgent extends BaseAgent {
      async *runAsyncImpl(context: InvocationContext): AsyncGenerator<Event, void, void> {
        yield {
          id: 'event-1',
          invocationId: context.invocationId,
          author: this.name,
          content: { parts: [{ text: 'Task completed!' }] },
          actions: createEventActions(),
          timestamp: Date.now(),
        };
      }
      async *runLiveImpl(context: InvocationContext): AsyncGenerator<Event, void, void> {
        this.runAsyncImpl(context);
      }
    }

    // Define agentes individuales
    const greeter = new LlmAgent({name: 'Greeter', model: 'gemini-2.5-flash'});
    const taskDoer = new TaskExecutorAgent({name: 'TaskExecutor'}); // Agente personalizado no-LLM

    // Crea agente padre y asigna hijos vía subAgents
    const coordinator = new LlmAgent({
        name: 'Coordinator',
        model: 'gemini-2.5-flash',
        description: 'I coordinate greetings and tasks.',
        subAgents: [ // Asigna subAgents aquí
            greeter,
            taskDoer
        ],
    });

    // El framework automáticamente establece:
    // console.assert(greeter.parentAgent === coordinator);
    // console.assert(taskDoer.parentAgent === coordinator);
    ```

=== "Go"

    ```go
    import (
        "google.golang.org/adk/agent"
        "google.golang.org/adk/agent/llmagent"
    )

    --8<-- "examples/go/snippets/agents/multi-agent/main.go:hierarchy"
    ```

=== "Java"

    ```java
    // Ejemplo Conceptual: Definiendo Jerarquía
    import com.google.adk.agents.SequentialAgent;
    import com.google.adk.agents.LlmAgent;


    // Define agentes individuales
    LlmAgent greeter = LlmAgent.builder().name("Greeter").model("gemini-2.0-flash").build();
    SequentialAgent taskDoer = SequentialAgent.builder().name("TaskExecutor").subAgents(...).build(); // Sequential Agent


    // Crea agente padre y asigna sub_agents
    LlmAgent coordinator = LlmAgent.builder()
        .name("Coordinator")
        .model("gemini-2.0-flash")
        .description("I coordinate greetings and tasks")
        .subAgents(greeter, taskDoer) // Asigna sub_agents aquí
        .build();


    // El framework automáticamente establece:
    // assert greeter.parentAgent().equals(coordinator);
    // assert taskDoer.parentAgent().equals(coordinator);
    ```

### 1.2. Agentes de Flujo de Trabajo como Orquestadores { #workflow-agents-as-orchestrators }

ADK incluye agentes especializados derivados de `BaseAgent` que no realizan tareas por sí mismos sino que orquestan el flujo de ejecución de sus `sub_agents`.

* **[`SequentialAgent`](workflow-agents/sequential-agents.md):** Ejecuta sus `sub_agents` uno tras otro en el orden en que están listados.
    * **Contexto:** Pasa el *mismo* [`InvocationContext`](../runtime/index.md) secuencialmente, permitiendo a los agentes pasar resultados fácilmente vía estado compartido.

=== "Python"

    ```python
    # Ejemplo Conceptual: Pipeline Secuencial
    from google.adk.agents import SequentialAgent, LlmAgent

    step1 = LlmAgent(name="Step1_Fetch", output_key="data") # Guarda salida en state['data']
    step2 = LlmAgent(name="Step2_Process", instruction="Process data from {data}.")

    pipeline = SequentialAgent(name="MyPipeline", sub_agents=[step1, step2])
    # Cuando pipeline se ejecuta, Step2 puede acceder al state['data'] establecido por Step1.
    ```

=== "Typescript"

    ```typescript
    // Ejemplo Conceptual: Pipeline Secuencial
    import { SequentialAgent, LlmAgent } from '@google/adk';

    const step1 = new LlmAgent({name: 'Step1_Fetch', outputKey: 'data'}); // Guarda salida en state['data']
    const step2 = new LlmAgent({name: 'Step2_Process', instruction: 'Process data from {data}.'});

    const pipeline = new SequentialAgent({name: 'MyPipeline', subAgents: [step1, step2]});
    // Cuando pipeline se ejecuta, Step2 puede acceder al state['data'] establecido por Step1.
    ```

=== "Go"

    ```go
    import (
        "google.golang.org/adk/agent"
        "google.golang.org/adk/agent/llmagent"
        "google.golang.org/adk/agent/workflowagents/sequentialagent"
    )

    --8<-- "examples/go/snippets/agents/multi-agent/main.go:sequential-pipeline"
    ```

=== "Java"

    ```java
    // Ejemplo Conceptual: Pipeline Secuencial
    import com.google.adk.agents.SequentialAgent;
    import com.google.adk.agents.LlmAgent;

    LlmAgent step1 = LlmAgent.builder().name("Step1_Fetch").outputKey("data").build(); // Guarda salida en state.get("data")
    LlmAgent step2 = LlmAgent.builder().name("Step2_Process").instruction("Process data from {data}.").build();

    SequentialAgent pipeline = SequentialAgent.builder().name("MyPipeline").subAgents(step1, step2).build();
    // Cuando pipeline se ejecuta, Step2 puede acceder al state.get("data") establecido por Step1.
    ```

* **[`ParallelAgent`](workflow-agents/parallel-agents.md):** Ejecuta sus `sub_agents` en paralelo. Los eventos de los sub-agentes pueden estar intercalados.
    * **Contexto:** Modifica el `InvocationContext.branch` para cada agente hijo (por ejemplo, `ParentBranch.ChildName`), proporcionando una ruta contextual distinta que puede ser útil para aislar el historial en algunas implementaciones de memoria.
    * **Estado:** A pesar de las diferentes ramas, todos los hijos paralelos acceden al *mismo* `session.state` compartido, permitiéndoles leer el estado inicial y escribir resultados (usa claves distintas para evitar condiciones de carrera).

=== "Python"

    ```python
    # Ejemplo Conceptual: Ejecución Paralela
    from google.adk.agents import ParallelAgent, LlmAgent

    fetch_weather = LlmAgent(name="WeatherFetcher", output_key="weather")
    fetch_news = LlmAgent(name="NewsFetcher", output_key="news")

    gatherer = ParallelAgent(name="InfoGatherer", sub_agents=[fetch_weather, fetch_news])
    # Cuando gatherer se ejecuta, WeatherFetcher y NewsFetcher se ejecutan concurrentemente.
    # Un agente subsecuente podría leer state['weather'] y state['news'].
    ```

=== "Typescript"

    ```typescript
    // Ejemplo Conceptual: Ejecución Paralela
    import { ParallelAgent, LlmAgent } from '@google/adk';

    const fetchWeather = new LlmAgent({name: 'WeatherFetcher', outputKey: 'weather'});
    const fetchNews = new LlmAgent({name: 'NewsFetcher', outputKey: 'news'});

    const gatherer = new ParallelAgent({name: 'InfoGatherer', subAgents: [fetchWeather, fetchNews]});
    // Cuando gatherer se ejecuta, WeatherFetcher y NewsFetcher se ejecutan concurrentemente.
    // Un agente subsecuente podría leer state['weather'] y state['news'].
    ```

=== "Go"

    ```go
    import (
        "google.golang.org/adk/agent"
        "google.golang.org/adk/agent/llmagent"
        "google.golang.org/adk/agent/workflowagents/parallelagent"
    )

    --8<-- "examples/go/snippets/agents/multi-agent/main.go:parallel-execution"
    ```

=== "Java"

    ```java
    // Ejemplo Conceptual: Ejecución Paralela
    import com.google.adk.agents.LlmAgent;
    import com.google.adk.agents.ParallelAgent;


    LlmAgent fetchWeather = LlmAgent.builder()
        .name("WeatherFetcher")
        .outputKey("weather")
        .build();


    LlmAgent fetchNews = LlmAgent.builder()
        .name("NewsFetcher")
        .instruction("news")
        .build();


    ParallelAgent gatherer = ParallelAgent.builder()
        .name("InfoGatherer")
        .subAgents(fetchWeather, fetchNews)
        .build();


    // Cuando gatherer se ejecuta, WeatherFetcher y NewsFetcher se ejecutan concurrentemente.
    // Un agente subsecuente podría leer state['weather'] y state['news'].
    ```

  * **[`LoopAgent`](workflow-agents/loop-agents.md):** Ejecuta sus `sub_agents` secuencialmente en un bucle.
      * **Terminación:** El bucle se detiene si se alcanza el `max_iterations` opcional, o si cualquier sub-agente retorna un [`Event`](../events/index.md) con `escalate=True` en sus Event Actions.
      * **Contexto y Estado:** Pasa el *mismo* `InvocationContext` en cada iteración, permitiendo que los cambios de estado (por ejemplo, contadores, banderas) persistan a través de los bucles.

=== "Python"

      ```python
      # Ejemplo Conceptual: Bucle con Condición
      from google.adk.agents import LoopAgent, LlmAgent, BaseAgent
      from google.adk.events import Event, EventActions
      from google.adk.agents.invocation_context import InvocationContext
      from typing import AsyncGenerator

      class CheckCondition(BaseAgent): # Agente personalizado para verificar estado
          async def _run_async_impl(self, ctx: InvocationContext) -> AsyncGenerator[Event, None]:
              status = ctx.session.state.get("status", "pending")
              is_done = (status == "completed")
              yield Event(author=self.name, actions=EventActions(escalate=is_done)) # Escala si está hecho

      process_step = LlmAgent(name="ProcessingStep") # Agente que podría actualizar state['status']

      poller = LoopAgent(
          name="StatusPoller",
          max_iterations=10,
          sub_agents=[process_step, CheckCondition(name="Checker")]
      )
      # Cuando poller se ejecuta, ejecuta process_step luego Checker repetidamente
      # hasta que Checker escala (state['status'] == 'completed') o pasen 10 iteraciones.
      ```

=== "Typescript"

    ```typescript
    // Ejemplo Conceptual: Bucle con Condición
    import { LoopAgent, LlmAgent, BaseAgent, InvocationContext } from '@google/adk';
    import type { Event, createEventActions, EventActions } from '@google/adk';

    class CheckConditionAgent extends BaseAgent { // Agente personalizado para verificar estado
        async *runAsyncImpl(ctx: InvocationContext): AsyncGenerator<Event> {
            const status = ctx.session.state['status'] || 'pending';
            const isDone = status === 'completed';
            yield createEvent({ author: 'check_condition', actions: createEventActions({ escalate: isDone }) });
        }

        async *runLiveImpl(ctx: InvocationContext): AsyncGenerator<Event> {
            // Esto no está implementado.
        }
    };

    const processStep = new LlmAgent({name: 'ProcessingStep'}); // Agente que podría actualizar state['status']

    const poller = new LoopAgent({
        name: 'StatusPoller',
        maxIterations: 10,
        // Ejecuta sus sub_agents secuencialmente en un bucle
        subAgents: [processStep, new CheckConditionAgent ({name: 'Checker'})]
    });
    // Cuando poller se ejecuta, ejecuta processStep luego Checker repetidamente
    // hasta que Checker escala (state['status'] === 'completed') o pasen 10 iteraciones.
    ```

=== "Go"

    ```go
    import (
        "iter"
        "google.golang.org/adk/agent"
        "google.golang.org/adk/agent/llmagent"
        "google.golang.org/adk/agent/workflowagents/loopagent"
        "google.golang.org/adk/session"
    )

    --8<-- "examples/go/snippets/agents/multi-agent/main.go:loop-with-condition"
    ```

      ```

=== "Java"

    ```java
    // Ejemplo Conceptual: Bucle con Condición
    // Agente personalizado para verificar estado y potencialmente escalar
    public static class CheckConditionAgent extends BaseAgent {
      public CheckConditionAgent(String name, String description) {
        super(name, description, List.of(), null, null);
      }


      @Override
      protected Flowable<Event> runAsyncImpl(InvocationContext ctx) {
        String status = (String) ctx.session().state().getOrDefault("status", "pending");
        boolean isDone = "completed".equalsIgnoreCase(status);

        // Emite un evento que señala escalar (salir del bucle) si la condición se cumple.
        // Si no está hecho, la bandera escalate será falsa o ausente, y el bucle continúa.
        Event checkEvent = Event.builder()
                .author(name())
                .id(Event.generateEventId()) // Importante dar a los eventos IDs únicos
                .actions(EventActions.builder().escalate(isDone).build()) // Escala si está hecho
                .build();
        return Flowable.just(checkEvent);
      }
    }


    // Agente que podría actualizar state.put("status")
    LlmAgent processingStepAgent = LlmAgent.builder().name("ProcessingStep").build();
    // Instancia de agente personalizado para verificar la condición
    CheckConditionAgent conditionCheckerAgent = new CheckConditionAgent(
        "ConditionChecker",
        "Checks if the status is 'completed'."
    );
    LoopAgent poller = LoopAgent.builder().name("StatusPoller").maxIterations(10).subAgents(processingStepAgent, conditionCheckerAgent).build();
    // Cuando poller se ejecuta, ejecuta processingStepAgent luego conditionCheckerAgent repetidamente
    // hasta que Checker escala (state.get("status") == "completed") o pasen 10 iteraciones.
    ```

### 1.3. Mecanismos de Interacción y Comunicación { #interaction-communication-mechanisms }

Los agentes dentro de un sistema a menudo necesitan intercambiar datos o activar acciones entre sí. ADK facilita esto a través de:

#### a) Estado de Sesión Compartido (`session.state`)

La forma más fundamental para que los agentes que operan dentro de la misma invocación (y por lo tanto comparten el mismo objeto [`Session`](/sessions/session/) vía el `InvocationContext`) se comuniquen pasivamente.

* **Mecanismo:** Un agente (o su herramienta/callback) escribe un valor (`context.state['data_key'] = processed_data`), y un agente subsecuente lo lee (`data = context.state.get('data_key')`). Los cambios de estado son rastreados vía [`CallbackContext`](../callbacks/index.md).
* **Conveniencia:** La propiedad `output_key` en [`LlmAgent`](llm-agents.md) automáticamente guarda el texto de respuesta final del agente (o salida estructurada) en la clave de estado especificada.
* **Naturaleza:** Comunicación asíncrona, pasiva. Ideal para pipelines orquestados por `SequentialAgent` o pasando datos a través de iteraciones de `LoopAgent`.
* **Ver También:** [Gestión de Estado](../sessions/state.md)

!!! note "Contexto de Invocación y Estado `temp:`"
    Cuando un agente padre invoca un sub-agente, pasa el mismo `InvocationContext`. Esto significa que comparten el mismo estado temporal (`temp:`), que es ideal para pasar datos que solo son relevantes para el turno actual.

=== "Python"

    ```python
    # Ejemplo Conceptual: Usando output_key y leyendo estado
    from google.adk.agents import LlmAgent, SequentialAgent


    agent_A = LlmAgent(name="AgentA", instruction="Find the capital of France.", output_key="capital_city")
    agent_B = LlmAgent(name="AgentB", instruction="Tell me about the city stored in {capital_city}.")


    pipeline = SequentialAgent(name="CityInfo", sub_agents=[agent_A, agent_B])
    # AgentA se ejecuta, guarda "Paris" en state['capital_city'].
    # AgentB se ejecuta, su procesador de instrucciones lee state['capital_city'] para obtener "Paris".
    ```

=== "Typescript"

    ```typescript
    // Ejemplo Conceptual: Usando outputKey y leyendo estado
    import { LlmAgent, SequentialAgent } from '@google/adk';

    const agentA = new LlmAgent({name: 'AgentA', instruction: 'Find the capital of France.', outputKey: 'capital_city'});
    const agentB = new LlmAgent({name: 'AgentB', instruction: 'Tell me about the city stored in {capital_city}.'});

    const pipeline = new SequentialAgent({name: 'CityInfo', subAgents: [agentA, agentB]});
    // AgentA se ejecuta, guarda "Paris" en state['capital_city'].
    // AgentB se ejecuta, su procesador de instrucciones lee state['capital_city'] para obtener "Paris".
    ```

=== "Go"

    ```go
    import (
        "google.golang.org/adk/agent"
        "google.golang.org/adk/agent/llmagent"
        "google.golang.org/adk/agent/workflowagents/sequentialagent"
    )

    --8<-- "examples/go/snippets/agents/multi-agent/main.go:output-key-state"
    ```

=== "Java"

    ```java
    // Ejemplo Conceptual: Usando outputKey y leyendo estado
    import com.google.adk.agents.LlmAgent;
    import com.google.adk.agents.SequentialAgent;


    LlmAgent agentA = LlmAgent.builder()
        .name("AgentA")
        .instruction("Find the capital of France.")
        .outputKey("capital_city")
        .build();


    LlmAgent agentB = LlmAgent.builder()
        .name("AgentB")
        .instruction("Tell me about the city stored in {capital_city}.")
        .outputKey("capital_city")
        .build();


    SequentialAgent pipeline = SequentialAgent.builder().name("CityInfo").subAgents(agentA, agentB).build();
    // AgentA se ejecuta, guarda "Paris" en state('capital_city').
    // AgentB se ejecuta, su procesador de instrucciones lee state.get("capital_city") para obtener "Paris".
    ```

#### b) Delegación Impulsada por LLM (Transferencia de Agente)

Aprovecha la comprensión de un [`LlmAgent`](llm-agents.md) para enrutar dinámicamente tareas a otros agentes apropiados dentro de la jerarquía.

* **Mecanismo:** El LLM del agente genera una llamada de función específica: `transfer_to_agent(agent_name='target_agent_name')`.
* **Manejo:** El `AutoFlow`, usado por defecto cuando hay sub-agentes presentes o la transferencia no está deshabilitada, intercepta esta llamada. Identifica el agente objetivo usando `root_agent.find_agent()` y actualiza el `InvocationContext` para cambiar el foco de ejecución.
* **Requiere:** El `LlmAgent` que llama necesita `instructions` claras sobre cuándo transferir, y los agentes objetivo potenciales necesitan `description`s distintas para que el LLM tome decisiones informadas. El alcance de transferencia (padre, sub-agente, hermanos) puede configurarse en el `LlmAgent`.
* **Naturaleza:** Enrutamiento dinámico, flexible basado en la interpretación del LLM.

=== "Python"

    ```python
    # Configuración Conceptual: Transferencia LLM
    from google.adk.agents import LlmAgent


    booking_agent = LlmAgent(name="Booker", description="Handles flight and hotel bookings.")
    info_agent = LlmAgent(name="Info", description="Provides general information and answers questions.")


    coordinator = LlmAgent(
        name="Coordinator",
        model="gemini-2.0-flash",
        instruction="You are an assistant. Delegate booking tasks to Booker and info requests to Info.",
        description="Main coordinator.",
        # AutoFlow se usa típicamente implícitamente aquí
        sub_agents=[booking_agent, info_agent]
    )
    # Si coordinator recibe "Book a flight", su LLM debería generar:
    # FunctionCall(name='transfer_to_agent', args={'agent_name': 'Booker'})
    # El framework ADK luego enruta la ejecución a booking_agent.
    ```

=== "Typescript"

    ```typescript
    // Configuración Conceptual: Transferencia LLM
    import { LlmAgent } from '@google/adk';

    const bookingAgent = new LlmAgent({name: 'Booker', description: 'Handles flight and hotel bookings.'});
    const infoAgent = new LlmAgent({name: 'Info', description: 'Provides general information and answers questions.'});

    const coordinator = new LlmAgent({
        name: 'Coordinator',
        model: 'gemini-2.5-flash',
        instruction: 'You are an assistant. Delegate booking tasks to Booker and info requests to Info.',
        description: 'Main coordinator.',
        // AutoFlow se usa típicamente implícitamente aquí
        subAgents: [bookingAgent, infoAgent]
    });
    // Si coordinator recibe "Book a flight", su LLM debería generar:
    // {functionCall: {name: 'transfer_to_agent', args: {agent_name: 'Booker'}}}
    // El framework ADK luego enruta la ejecución a bookingAgent.
    ```

=== "Go"

    ```go
    import (
        "google.golang.org/adk/agent/llmagent"
    )

    --8<-- "examples/go/snippets/agents/multi-agent/main.go:llm-transfer"
    ```

=== "Java"

    ```java
    // Configuración Conceptual: Transferencia LLM
    import com.google.adk.agents.LlmAgent;


    LlmAgent bookingAgent = LlmAgent.builder()
        .name("Booker")
        .description("Handles flight and hotel bookings.")
        .build();


    LlmAgent infoAgent = LlmAgent.builder()
        .name("Info")
        .description("Provides general information and answers questions.")
        .build();


    // Define el agente coordinador
    LlmAgent coordinator = LlmAgent.builder()
        .name("Coordinator")
        .model("gemini-2.0-flash") // O tu modelo deseado
        .instruction("You are an assistant. Delegate booking tasks to Booker and info requests to Info.")
        .description("Main coordinator.")
        // AutoFlow se usará por defecto (implícitamente) porque subAgents están presentes
        // y la transferencia no está deshabilitada.
        .subAgents(bookingAgent, infoAgent)
        .build();

    // Si coordinator recibe "Book a flight", su LLM debería generar:
    // FunctionCall.builder.name("transferToAgent").args(ImmutableMap.of("agent_name", "Booker")).build()
    // El framework ADK luego enruta la ejecución a bookingAgent.
    ```

#### c) Invocación Explícita (`AgentTool`)

Permite a un [`LlmAgent`](llm-agents.md) tratar a otra instancia de `BaseAgent` como una función o [Herramienta](../tools/index.md) invocable.

* **Mecanismo:** Envuelve la instancia del agente objetivo en `AgentTool` e inclúyelo en la lista de `tools` del `LlmAgent` padre. `AgentTool` genera una declaración de función correspondiente para el LLM.
* **Manejo:** Cuando el LLM padre genera una llamada de función dirigida al `AgentTool`, el framework ejecuta `AgentTool.run_async`. Este método ejecuta el agente objetivo, captura su respuesta final, reenvía cualquier cambio de estado/artefacto de vuelta al contexto del padre, y retorna la respuesta como el resultado de la herramienta.
* **Naturaleza:** Invocación síncrona (dentro del flujo del padre), explícita, controlada como cualquier otra herramienta.
* **(Nota:** `AgentTool` necesita ser importado y usado explícitamente).

=== "Python"

    ```python
    # Configuración Conceptual: Agente como una Herramienta
    from google.adk.agents import LlmAgent, BaseAgent
    from google.adk.tools import agent_tool
    from pydantic import BaseModel


    # Define un agente objetivo (podría ser LlmAgent o BaseAgent personalizado)
    class ImageGeneratorAgent(BaseAgent): # Ejemplo de agente personalizado
        name: str = "ImageGen"
        description: str = "Generates an image based on a prompt."
        # ... lógica interna ...
        async def _run_async_impl(self, ctx): # Lógica de ejecución simplificada
            prompt = ctx.session.state.get("image_prompt", "default prompt")
            # ... generar bytes de imagen ...
            image_bytes = b"..."
            yield Event(author=self.name, content=types.Content(parts=[types.Part.from_bytes(image_bytes, "image/png")]))


    image_agent = ImageGeneratorAgent()
    image_tool = agent_tool.AgentTool(agent=image_agent) # Envuelve el agente


    # Agente padre usa el AgentTool
    artist_agent = LlmAgent(
        name="Artist",
        model="gemini-2.0-flash",
        instruction="Create a prompt and use the ImageGen tool to generate the image.",
        tools=[image_tool] # Incluye el AgentTool
    )
    # El LLM Artist genera un prompt, luego llama:
    # FunctionCall(name='ImageGen', args={'image_prompt': 'a cat wearing a hat'})
    # El framework llama image_tool.run_async(...), que ejecuta ImageGeneratorAgent.
    # El Part de imagen resultante se retorna al agente Artist como resultado de la herramienta.
    ```

=== "Typescript"

    ```typescript
    // Configuración Conceptual: Agente como una Herramienta
    import { LlmAgent, BaseAgent, AgentTool, InvocationContext } from '@google/adk';
    import type { Part, createEvent, Event } from '@google/genai';

    // Define un agente objetivo (podría ser LlmAgent o BaseAgent personalizado)
    class ImageGeneratorAgent extends BaseAgent { // Ejemplo de agente personalizado
        constructor() {
            super({name: 'ImageGen', description: 'Generates an image based on a prompt.'});
        }
        // ... lógica interna ...
        async *runAsyncImpl(ctx: InvocationContext): AsyncGenerator<Event> { // Lógica de ejecución simplificada
            const prompt = ctx.session.state['image_prompt'] || 'default prompt';
            // ... generar bytes de imagen ...
            const imageBytes = new Uint8Array(); // placeholder
            const imagePart: Part = {inlineData: {data: Buffer.from(imageBytes).toString('base64'), mimeType: 'image/png'}};
            yield createEvent({content: {parts: [imagePart]}});
        }

        async *runLiveImpl(ctx: InvocationContext): AsyncGenerator<Event, void, void> {
            // No implementado para este agente.
        }
    }

    const imageAgent = new ImageGeneratorAgent();
    const imageTool = new AgentTool({agent: imageAgent}); // Envuelve el agente

    // Agente padre usa el AgentTool
    const artistAgent = new LlmAgent({
        name: 'Artist',
        model: 'gemini-2.5-flash',
        instruction: 'Create a prompt and use the ImageGen tool to generate the image.',
        tools: [imageTool] // Incluye el AgentTool
    });
    // El LLM Artist genera un prompt, luego llama:
    // {functionCall: {name: 'ImageGen', args: {image_prompt: 'a cat wearing a hat'}}}
    // El framework llama imageTool.runAsync(...), que ejecuta ImageGeneratorAgent.
    // El Part de imagen resultante se retorna al agente Artist como resultado de la herramienta.
    ```

=== "Go"

    ```go
    import (
        "fmt"
        "iter"
        "google.golang.org/adk/agent"
        "google.golang.org/adk/agent/llmagent"
        "google.golang.org/adk/model"
        "google.golang.org/adk/session"
        "google.golang.org/adk/tool"
        "google.golang.org/adk/tool/agenttool"
        "google.golang.org/genai"
    )

    --8<-- "examples/go/snippets/agents/multi-agent/main.go:agent-as-tool"
    ```

=== "Java"

    ```java
    // Configuración Conceptual: Agente como una Herramienta
    import com.google.adk.agents.BaseAgent;
    import com.google.adk.agents.LlmAgent;
    import com.google.adk.tools.AgentTool;

    // Ejemplo de agente personalizado (podría ser LlmAgent o BaseAgent personalizado)
    public class ImageGeneratorAgent extends BaseAgent  {


      public ImageGeneratorAgent(String name, String description) {
        super(name, description, List.of(), null, null);
      }


      // ... lógica interna ...
      @Override
      protected Flowable<Event> runAsyncImpl(InvocationContext invocationContext) { // Lógica de ejecución simplificada
        invocationContext.session().state().get("image_prompt");
        // Generar bytes de imagen
        // ...


        Event responseEvent = Event.builder()
            .author(this.name())
            .content(Content.fromParts(Part.fromText("...")))
            .build();


        return Flowable.just(responseEvent);
      }


      @Override
      protected Flowable<Event> runLiveImpl(InvocationContext invocationContext) {
        return null;
      }
    }

    // Envuelve el agente usando AgentTool
    ImageGeneratorAgent imageAgent = new ImageGeneratorAgent("image_agent", "generates images");
    AgentTool imageTool = AgentTool.create(imageAgent);


    // Agente padre usa el AgentTool
    LlmAgent artistAgent = LlmAgent.builder()
            .name("Artist")
            .model("gemini-2.0-flash")
            .instruction(
                    "You are an artist. Create a detailed prompt for an image and then " +
                            "use the 'ImageGen' tool to generate the image. " +
                            "The 'ImageGen' tool expects a single string argument named 'request' " +
                            "containing the image prompt. The tool will return a JSON string in its " +
                            "'result' field, containing 'image_base64', 'mime_type', and 'status'."
            )
            .description("An agent that can create images using a generation tool.")
            .tools(imageTool) // Incluye el AgentTool
            .build();


    // El LLM Artist genera un prompt, luego llama:
    // FunctionCall(name='ImageGen', args={'imagePrompt': 'a cat wearing a hat'})
    // El framework llama imageTool.runAsync(...), que ejecuta ImageGeneratorAgent.
    // El Part de imagen resultante se retorna al agente Artist como resultado de la herramienta.
    ```

Estas primitivas proporcionan la flexibilidad para diseñar interacciones multi-agente que van desde flujos de trabajo secuenciales estrechamente acoplados hasta redes de delegación dinámicas impulsadas por LLM.

## 2. Patrones Comunes Multi-Agente usando Primitivas de ADK { #common-multi-agent-patterns-using-adk-primitives }

Combinando las primitivas de composición de ADK, puedes implementar varios patrones establecidos para colaboración multi-agente.

### Patrón Coordinador/Despachador

* **Estructura:** Un [`LlmAgent`](llm-agents.md) central (Coordinador) gestiona varios `sub_agents` especializados.
* **Objetivo:** Enrutar solicitudes entrantes al agente especialista apropiado.
* **Primitivas de ADK Usadas:**
    * **Jerarquía:** El Coordinador tiene especialistas listados en `sub_agents`.
    * **Interacción:** Principalmente usa **Delegación Impulsada por LLM** (requiere `description`s claras en sub-agentes e `instruction` apropiada en el Coordinador) o **Invocación Explícita (`AgentTool`)** (el Coordinador incluye especialistas envueltos en `AgentTool` en sus `tools`).

=== "Python"

    ```python
    # Código Conceptual: Coordinador usando Transferencia LLM
    from google.adk.agents import LlmAgent


    billing_agent = LlmAgent(name="Billing", description="Handles billing inquiries.")
    support_agent = LlmAgent(name="Support", description="Handles technical support requests.")


    coordinator = LlmAgent(
        name="HelpDeskCoordinator",
        model="gemini-2.0-flash",
        instruction="Route user requests: Use Billing agent for payment issues, Support agent for technical problems.",
        description="Main help desk router.",
        # allow_transfer=True a menudo es implícito con sub_agents en AutoFlow
        sub_agents=[billing_agent, support_agent]
    )
    # Usuario pregunta "My payment failed" -> El LLM del Coordinador debería llamar transfer_to_agent(agent_name='Billing')
    # Usuario pregunta "I can't log in" -> El LLM del Coordinador debería llamar transfer_to_agent(agent_name='Support')
    ```

=== "Typescript"

    ```typescript
    // Código Conceptual: Coordinador usando Transferencia LLM
    import { LlmAgent } from '@google/adk';

    const billingAgent = new LlmAgent({name: 'Billing', description: 'Handles billing inquiries.'});
    const supportAgent = new LlmAgent({name: 'Support', description: 'Handles technical support requests.'});

    const coordinator = new LlmAgent({
        name: 'HelpDeskCoordinator',
        model: 'gemini-2.5-flash',
        instruction: 'Route user requests: Use Billing agent for payment issues, Support agent for technical problems.',
        description: 'Main help desk router.',
        // allowTransfer=true a menudo es implícito con subAgents en AutoFlow
        subAgents: [billingAgent, supportAgent]
    });
    // Usuario pregunta "My payment failed" -> El LLM del Coordinador debería llamar {functionCall: {name: 'transfer_to_agent', args: {agent_name: 'Billing'}}}
    // Usuario pregunta "I can't log in" -> El LLM del Coordinador debería llamar {functionCall: {name: 'transfer_to_agent', args: {agent_name: 'Support'}}}
    ```

=== "Go"

    ```go
    import (
        "google.golang.org/adk/agent"
        "google.golang.org/adk/agent/llmagent"
    )

    --8<-- "examples/go/snippets/agents/multi-agent/main.go:coordinator-pattern"
    ```

=== "Java"

    ```java
    // Código Conceptual: Coordinador usando Transferencia LLM
    import com.google.adk.agents.LlmAgent;

    LlmAgent billingAgent = LlmAgent.builder()
        .name("Billing")
        .description("Handles billing inquiries and payment issues.")
        .build();

    LlmAgent supportAgent = LlmAgent.builder()
        .name("Support")
        .description("Handles technical support requests and login problems.")
        .build();

    LlmAgent coordinator = LlmAgent.builder()
        .name("HelpDeskCoordinator")
        .model("gemini-2.0-flash")
        .instruction("Route user requests: Use Billing agent for payment issues, Support agent for technical problems.")
        .description("Main help desk router.")
        .subAgents(billingAgent, supportAgent)
        // La transferencia de agente es implícita con sub agents en el Autoflow, a menos que se especifique
        // usando .disallowTransferToParent o disallowTransferToPeers
        .build();

    // Usuario pregunta "My payment failed" -> El LLM del Coordinador debería llamar
    // transferToAgent(agentName='Billing')
    // Usuario pregunta "I can't log in" -> El LLM del Coordinador debería llamar
    // transferToAgent(agentName='Support')
    ```

### Patrón Pipeline Secuencial

* **Estructura:** Un [`SequentialAgent`](workflow-agents/sequential-agents.md) contiene `sub_agents` ejecutados en un orden fijo.
* **Objetivo:** Implementar un proceso de múltiples pasos donde la salida de un paso alimenta al siguiente.
* **Primitivas de ADK Usadas:**
    * **Flujo de Trabajo:** `SequentialAgent` define el orden.
    * **Comunicación:** Principalmente usa **Estado de Sesión Compartido**. Los agentes anteriores escriben resultados (a menudo vía `output_key`), los agentes posteriores leen esos resultados de `context.state`.

=== "Python"

    ```python
    # Código Conceptual: Pipeline de Datos Secuencial
    from google.adk.agents import SequentialAgent, LlmAgent


    validator = LlmAgent(name="ValidateInput", instruction="Validate the input.", output_key="validation_status")
    processor = LlmAgent(name="ProcessData", instruction="Process data if {validation_status} is 'valid'.", output_key="result")
    reporter = LlmAgent(name="ReportResult", instruction="Report the result from {result}.")


    data_pipeline = SequentialAgent(
        name="DataPipeline",
        sub_agents=[validator, processor, reporter]
    )
    # validator se ejecuta -> guarda en state['validation_status']
    # processor se ejecuta -> lee state['validation_status'], guarda en state['result']
    # reporter se ejecuta -> lee state['result']
    ```

=== "Typescript"

    ```typescript
    // Código Conceptual: Pipeline de Datos Secuencial
    import { SequentialAgent, LlmAgent } from '@google/adk';

    const validator = new LlmAgent({name: 'ValidateInput', instruction: 'Validate the input.', outputKey: 'validation_status'});
    const processor = new LlmAgent({name: 'ProcessData', instruction: 'Process data if {validation_status} is "valid".', outputKey: 'result'});
    const reporter = new LlmAgent({name: 'ReportResult', instruction: 'Report the result from {result}.'});

    const dataPipeline = new SequentialAgent({
        name: 'DataPipeline',
        subAgents: [validator, processor, reporter]
    });
    // validator se ejecuta -> guarda en state['validation_status']
    // processor se ejecuta -> lee state['validation_status'], guarda en state['result']
    // reporter se ejecuta -> lee state['result']
    ```

=== "Go"

    ```go
    import (
        "google.golang.org/adk/agent"
        "google.golang.org/adk/agent/llmagent"
        "google.golang.org/adk/agent/workflowagents/sequentialagent"
    )

    --8<-- "examples/go/snippets/agents/multi-agent/main.go:sequential-pipeline-pattern"
    ```

=== "Java"

    ```java
    // Código Conceptual: Pipeline de Datos Secuencial
    import com.google.adk.agents.SequentialAgent;


    LlmAgent validator = LlmAgent.builder()
        .name("ValidateInput")
        .instruction("Validate the input")
        .outputKey("validation_status") // Guarda su salida de texto principal en session.state["validation_status"]
        .build();


    LlmAgent processor = LlmAgent.builder()
        .name("ProcessData")
        .instruction("Process data if {validation_status} is 'valid'")
        .outputKey("result") // Guarda su salida de texto principal en session.state["result"]
        .build();


    LlmAgent reporter = LlmAgent.builder()
        .name("ReportResult")
        .instruction("Report the result from {result}")
        .build();


    SequentialAgent dataPipeline = SequentialAgent.builder()
        .name("DataPipeline")
        .subAgents(validator, processor, reporter)
        .build();


    // validator se ejecuta -> guarda en state['validation_status']
    // processor se ejecuta -> lee state['validation_status'], guarda en state['result']
    // reporter se ejecuta -> lee state['result']
    ```

### Patrón Paralelo Fan-Out/Gather

* **Estructura:** Un [`ParallelAgent`](workflow-agents/parallel-agents.md) ejecuta múltiples `sub_agents` concurrentemente, a menudo seguido por un agente posterior (en un `SequentialAgent`) que agrega resultados.
* **Objetivo:** Ejecutar tareas independientes simultáneamente para reducir latencia, luego combinar sus salidas.
* **Primitivas de ADK Usadas:**
    * **Flujo de Trabajo:** `ParallelAgent` para ejecución concurrente (Fan-Out). A menudo anidado dentro de un `SequentialAgent` para manejar el paso de agregación subsecuente (Gather).
    * **Comunicación:** Los sub-agentes escriben resultados en claves distintas en **Estado de Sesión Compartido**. El agente "Gather" subsecuente lee múltiples claves de estado.

=== "Python"

    ```python
    # Código Conceptual: Recopilación de Información Paralela
    from google.adk.agents import SequentialAgent, ParallelAgent, LlmAgent


    fetch_api1 = LlmAgent(name="API1Fetcher", instruction="Fetch data from API 1.", output_key="api1_data")
    fetch_api2 = LlmAgent(name="API2Fetcher", instruction="Fetch data from API 2.", output_key="api2_data")


    gather_concurrently = ParallelAgent(
        name="ConcurrentFetch",
        sub_agents=[fetch_api1, fetch_api2]
    )


    synthesizer = LlmAgent(
        name="Synthesizer",
        instruction="Combine results from {api1_data} and {api2_data}."
    )


    overall_workflow = SequentialAgent(
        name="FetchAndSynthesize",
        sub_agents=[gather_concurrently, synthesizer] # Ejecuta fetch paralelo, luego sintetiza
    )
    # fetch_api1 y fetch_api2 se ejecutan concurrentemente, guardando en estado.
    # synthesizer se ejecuta después, leyendo state['api1_data'] y state['api2_data'].
    ```

=== "Typescript"

    ```typescript
    // Código Conceptual: Recopilación de Información Paralela
    import { SequentialAgent, ParallelAgent, LlmAgent } from '@google/adk';

    const fetchApi1 = new LlmAgent({name: 'API1Fetcher', instruction: 'Fetch data from API 1.', outputKey: 'api1_data'});
    const fetchApi2 = new LlmAgent({name: 'API2Fetcher', instruction: 'Fetch data from API 2.', outputKey: 'api2_data'});

    const gatherConcurrently = new ParallelAgent({
        name: 'ConcurrentFetch',
        subAgents: [fetchApi1, fetchApi2]
    });

    const synthesizer = new LlmAgent({
        name: 'Synthesizer',
        instruction: 'Combine results from {api1_data} and {api2_data}.'
    });

    const overallWorkflow = new SequentialAgent({
        name: 'FetchAndSynthesize',
        subAgents: [gatherConcurrently, synthesizer] // Ejecuta fetch paralelo, luego sintetiza
    });
    // fetchApi1 y fetchApi2 se ejecutan concurrentemente, guardando en estado.
    // synthesizer se ejecuta después, leyendo state['api1_data'] y state['api2_data'].
    ```

=== "Go"

    ```go
    import (
        "google.golang.org/adk/agent"
        "google.golang.org/adk/agent/llmagent"
        "google.golang.org/adk/agent/workflowagents/parallelagent"
        "google.golang.org/adk/agent/workflowagents/sequentialagent"
    )

    --8<-- "examples/go/snippets/agents/multi-agent/main.go:parallel-gather-pattern"
    ```

=== "Java"

    ```java
    // Código Conceptual: Recopilación de Información Paralela
    import com.google.adk.agents.LlmAgent;
    import com.google.adk.agents.ParallelAgent;
    import com.google.adk.agents.SequentialAgent;

    LlmAgent fetchApi1 = LlmAgent.builder()
        .name("API1Fetcher")
        .instruction("Fetch data from API 1.")
        .outputKey("api1_data")
        .build();

    LlmAgent fetchApi2 = LlmAgent.builder()
        .name("API2Fetcher")
        .instruction("Fetch data from API 2.")
        .outputKey("api2_data")
        .build();

    ParallelAgent gatherConcurrently = ParallelAgent.builder()
        .name("ConcurrentFetcher")
        .subAgents(fetchApi2, fetchApi1)
        .build();

    LlmAgent synthesizer = LlmAgent.builder()
        .name("Synthesizer")
        .instruction("Combine results from {api1_data} and {api2_data}.")
        .build();

    SequentialAgent overallWorfklow = SequentialAgent.builder()
        .name("FetchAndSynthesize") // Ejecuta fetch paralelo, luego sintetiza
        .subAgents(gatherConcurrently, synthesizer)
        .build();

    // fetch_api1 y fetch_api2 se ejecutan concurrentemente, guardando en estado.
    // synthesizer se ejecuta después, leyendo state['api1_data'] y state['api2_data'].
    ```

### Descomposición Jerárquica de Tareas

* **Estructura:** Un árbol multinivel de agentes donde los agentes de nivel superior descomponen objetivos complejos y delegan sub-tareas a agentes de nivel inferior.
* **Objetivo:** Resolver problemas complejos descomponiéndolos recursivamente en pasos más simples y ejecutables.
* **Primitivas de ADK Usadas:**
    * **Jerarquía:** Estructura multinivel `parent_agent`/`sub_agents`.
    * **Interacción:** Principalmente **Delegación Impulsada por LLM** o **Invocación Explícita (`AgentTool`)** usada por agentes padre para asignar tareas a sub-agentes. Los resultados se retornan hacia arriba en la jerarquía (vía respuestas de herramientas o estado).

=== "Python"

    ```python
    # Código Conceptual: Tarea de Investigación Jerárquica
    from google.adk.agents import LlmAgent
    from google.adk.tools import agent_tool


    # Agentes de bajo nivel tipo herramienta
    web_searcher = LlmAgent(name="WebSearch", description="Performs web searches for facts.")
    summarizer = LlmAgent(name="Summarizer", description="Summarizes text.")


    # Agente de nivel medio combinando herramientas
    research_assistant = LlmAgent(
        name="ResearchAssistant",
        model="gemini-2.0-flash",
        description="Finds and summarizes information on a topic.",
        tools=[agent_tool.AgentTool(agent=web_searcher), agent_tool.AgentTool(agent=summarizer)]
    )


    # Agente de alto nivel delegando investigación
    report_writer = LlmAgent(
        name="ReportWriter",
        model="gemini-2.0-flash",
        instruction="Write a report on topic X. Use the ResearchAssistant to gather information.",
        tools=[agent_tool.AgentTool(agent=research_assistant)]
        # Alternativamente, podría usar Transferencia LLM si research_assistant es un sub_agent
    )
    # Usuario interactúa con ReportWriter.
    # ReportWriter llama a la herramienta ResearchAssistant.
    # ResearchAssistant llama a las herramientas WebSearch y Summarizer.
    # Los resultados fluyen de vuelta hacia arriba.
    ```

=== "Typescript"

    ```typescript
    // Código Conceptual: Tarea de Investigación Jerárquica
    import { LlmAgent, AgentTool } from '@google/adk';

    // Agentes de bajo nivel tipo herramienta
    const webSearcher = new LlmAgent({name: 'WebSearch', description: 'Performs web searches for facts.'});
    const summarizer = new LlmAgent({name: 'Summarizer', description: 'Summarizes text.'});

    // Agente de nivel medio combinando herramientas
    const researchAssistant = new LlmAgent({
        name: 'ResearchAssistant',
        model: 'gemini-2.5-flash',
        description: 'Finds and summarizes information on a topic.',
        tools: [new AgentTool({agent: webSearcher}), new AgentTool({agent: summarizer})]
    });

    // Agente de alto nivel delegando investigación
    const reportWriter = new LlmAgent({
        name: 'ReportWriter',
        model: 'gemini-2.5-flash',
        instruction: 'Write a report on topic X. Use the ResearchAssistant to gather information.',
        tools: [new AgentTool({agent: researchAssistant})]
        // Alternativamente, podría usar Transferencia LLM si researchAssistant es un subAgent
    });
    // Usuario interactúa con ReportWriter.
    // ReportWriter llama a la herramienta ResearchAssistant.
    // ResearchAssistant llama a las herramientas WebSearch y Summarizer.
    // Los resultados fluyen de vuelta hacia arriba.
    ```

=== "Go"

    ```go
    import (
        "google.golang.org/adk/agent/llmagent"
        "google.golang.org/adk/tool"
        "google.golang.org/adk/tool/agenttool"
    )

    --8<-- "examples/go/snippets/agents/multi-agent/main.go:hierarchical-pattern"
    ```

=== "Java"

    ```java
    // Código Conceptual: Tarea de Investigación Jerárquica
    import com.google.adk.agents.LlmAgent;
    import com.google.adk.tools.AgentTool;


    // Agentes de bajo nivel tipo herramienta
    LlmAgent webSearcher = LlmAgent.builder()
        .name("WebSearch")
        .description("Performs web searches for facts.")
        .build();


    LlmAgent summarizer = LlmAgent.builder()
        .name("Summarizer")
        .description("Summarizes text.")
        .build();


    // Agente de nivel medio combinando herramientas
    LlmAgent researchAssistant = LlmAgent.builder()
        .name("ResearchAssistant")
        .model("gemini-2.0-flash")
        .description("Finds and summarizes information on a topic.")
        .tools(AgentTool.create(webSearcher), AgentTool.create(summarizer))
        .build();


    // Agente de alto nivel delegando investigación
    LlmAgent reportWriter = LlmAgent.builder()
        .name("ReportWriter")
        .model("gemini-2.0-flash")
        .instruction("Write a report on topic X. Use the ResearchAssistant to gather information.")
        .tools(AgentTool.create(researchAssistant))
        // Alternativamente, podría usar Transferencia LLM si research_assistant es un subAgent
        .build();


    // Usuario interactúa con ReportWriter.
    // ReportWriter llama a la herramienta ResearchAssistant.
    // ResearchAssistant llama a las herramientas WebSearch y Summarizer.
    // Los resultados fluyen de vuelta hacia arriba.
    ```

### Patrón Revisión/Crítica (Generador-Crítico)

* **Estructura:** Típicamente involucra dos agentes dentro de un [`SequentialAgent`](workflow-agents/sequential-agents.md): un Generador y un Crítico/Revisor.
* **Objetivo:** Mejorar la calidad o validez de la salida generada teniendo un agente dedicado para revisarla.
* **Primitivas de ADK