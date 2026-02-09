# Contexto

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

En el Agent Development Kit (ADK), "contexto" se refiere al paquete crucial de información disponible para tu agente y sus herramientas durante operaciones específicas. Piénsalo como el conocimiento de fondo y los recursos necesarios para manejar una tarea actual o un turno de conversación de manera efectiva.

Los agentes a menudo necesitan más que solo el último mensaje del usuario para funcionar bien. El contexto es esencial porque permite:

1. **Mantener Estado:** Recordar detalles a través de múltiples pasos en una conversación (por ejemplo, preferencias del usuario, cálculos previos, elementos en un carrito de compras). Esto se gestiona principalmente a través del **estado de sesión**.
2. **Pasar Datos:** Compartir información descubierta o generada en un paso (como una llamada LLM o la ejecución de una herramienta) con los pasos subsiguientes. El estado de sesión es clave aquí también.
3. **Acceder a Servicios:** Interactuar con capacidades del framework como:
    * **Almacenamiento de Artefactos:** Guardar o cargar archivos o blobs de datos (como PDFs, imágenes, archivos de configuración) asociados con la sesión.
    * **Memoria:** Buscar información relevante de interacciones pasadas o fuentes de conocimiento externas conectadas al usuario.
    * **Autenticación:** Solicitar y recuperar credenciales necesarias por las herramientas para acceder a APIs externas de forma segura.
4. **Identidad y Seguimiento:** Saber qué agente se está ejecutando actualmente (`agent.name`) e identificar de manera única el ciclo actual de solicitud-respuesta (`invocation_id`) para logging y depuración.
5. **Acciones Específicas de Herramientas:** Habilitar operaciones especializadas dentro de las herramientas, como solicitar autenticación o buscar en memoria, que requieren acceso a los detalles de la interacción actual.

La pieza central que mantiene toda esta información junta para un solo ciclo completo de solicitud-de-usuario-a-respuesta-final (una **invocación**) es el `InvocationContext`. Sin embargo, típicamente no crearás ni gestionarás este objeto directamente. El framework ADK lo crea cuando una invocación comienza (por ejemplo, vía `runner.run_async`) y pasa la información contextual relevante de forma implícita a tu código de agente, callbacks y herramientas.

=== "Python"

    ```python
    # Pseudocódigo Conceptual: Cómo el framework proporciona contexto (Lógica Interna)

    # runner = Runner(agent=my_root_agent, session_service=..., artifact_service=...)
    # user_message = types.Content(...)
    # session = session_service.get_session(...) # O crear nueva

    # --- Dentro de runner.run_async(...) ---
    # 1. El framework crea el contexto principal para esta ejecución específica
    # invocation_context = InvocationContext(
    #     invocation_id="unique-id-for-this-run",
    #     session=session,
    #     user_content=user_message,
    #     agent=my_root_agent, # El agente inicial
    #     session_service=session_service,
    #     artifact_service=artifact_service,
    #     memory_service=memory_service,
    #     # ... otros campos necesarios ...
    # )
    #
    # 2. El framework llama al método run del agente, pasando el contexto implícitamente
    #    (La firma del método del agente lo recibirá, ej., runAsyncImpl(InvocationContext invocationContext))
    # await my_root_agent.run_async(invocation_context)
    #   --- Fin Lógica Interna ---
    #
    # Como desarrollador, trabajas con los objetos de contexto proporcionados en los argumentos de método.
    ```

=== "TypeScript"

    ```typescript
    /* Pseudocódigo Conceptual: Cómo el framework proporciona contexto (Lógica Interna) */

    const runner = new InMemoryRunner({ agent: myRootAgent });
    const session = await runner.sessionService.createSession({ ... });
    const userMessage = createUserContent(...);

    // --- Dentro de runner.runAsync(...) ---
    // 1. El framework crea el contexto principal para esta ejecución específica
    const invocationContext = new InvocationContext({
      invocationId: "unique-id-for-this-run",
      session: session,
      userContent: userMessage,
      agent: myRootAgent, // El agente inicial
      sessionService: runner.sessionService,
      pluginManager: runner.pluginManager,
      // ... otros campos necesarios ...
    });
    //
    // 2. El framework llama al método run del agente, pasando el contexto implícitamente
    await myRootAgent.runAsync(invocationContext);
    //   --- Fin Lógica Interna ---

    // Como desarrollador, trabajas con los objetos de contexto proporcionados en los argumentos de método.
    ```

=== "Go"

    ```go
    /* Pseudocódigo Conceptual: Cómo el framework proporciona contexto (Lógica Interna) */
    --8<-- "examples/go/snippets/context/main.go:conceptual_runner_example"
    ```

=== "Java"

    ```java
    /* Pseudocódigo Conceptual: Cómo el framework proporciona contexto (Lógica Interna) */
    InMemoryRunner runner = new InMemoryRunner(agent);
    Session session = runner
        .sessionService()
        .createSession(runner.appName(), USER_ID, initialState, SESSION_ID )
        .blockingGet();

    try (Scanner scanner = new Scanner(System.in, StandardCharsets.UTF_8)) {
      while (true) {
        System.out.print("\nYou > ");
      }
      String userInput = scanner.nextLine();
      if ("quit".equalsIgnoreCase(userInput)) {
        break;
      }
      Content userMsg = Content.fromParts(Part.fromText(userInput));
      Flowable<Event> events = runner.runAsync(session.userId(), session.id(), userMsg);
      System.out.print("\nAgent > ");
      events.blockingForEach(event -> System.out.print(event.stringifyContent()));
    }
    ```

## Los Diferentes Tipos de Contexto

Mientras `InvocationContext` actúa como el contenedor interno integral, ADK proporciona objetos de contexto especializados adaptados a situaciones específicas. Esto asegura que tengas las herramientas y permisos adecuados para la tarea en cuestión sin necesitar manejar la complejidad completa del contexto interno en todas partes. Aquí están los diferentes "sabores" que encontrarás:

1.  **`InvocationContext`**
    *   **Dónde se Usa:** Recibido como el argumento `ctx` directamente dentro de los métodos de implementación principales de un agente (`_run_async_impl`, `_run_live_impl`).
    *   **Propósito:** Proporciona acceso al estado *completo* de la invocación actual. Este es el objeto de contexto más integral.
    *   **Contenidos Clave:** Acceso directo a `session` (incluyendo `state` y `events`), la instancia actual del `agent`, `invocation_id`, `user_content` inicial, referencias a servicios configurados (`artifact_service`, `memory_service`, `session_service`), y campos relacionados con modos live/streaming.
    *   **Caso de Uso:** Usado principalmente cuando la lógica central del agente necesita acceso directo a la sesión general o servicios, aunque a menudo las interacciones de estado y artefactos se delegan a callbacks/herramientas que usan sus propios contextos. También se usa para controlar la invocación en sí (por ejemplo, configurando `ctx.end_invocation = True`).

    === "Python"

        ```python
        # Pseudocódigo: Implementación de agente recibiendo InvocationContext
        from google.adk.agents import BaseAgent
        from google.adk.agents.invocation_context import InvocationContext
        from google.adk.events import Event
        from typing import AsyncGenerator

        class MyAgent(BaseAgent):
            async def _run_async_impl(self, ctx: InvocationContext) -> AsyncGenerator[Event, None]:
                # Ejemplo de acceso directo
                agent_name = ctx.agent.name
                session_id = ctx.session.id
                print(f"Agent {agent_name} running in session {session_id} for invocation {ctx.invocation_id}")
                # ... lógica del agente usando ctx ...
                yield # ... evento ...
        ```

    === "TypeScript"

        ```typescript
        // Pseudocódigo: Implementación de agente recibiendo InvocationContext
        import { BaseAgent, InvocationContext, Event } from '@google/adk';

        class MyAgent extends BaseAgent {
          async *runAsyncImpl(ctx: InvocationContext): AsyncGenerator<Event, void, undefined> {
            // Ejemplo de acceso directo
            const agentName = ctx.agent.name;
            const sessionId = ctx.session.id;
            console.log(`Agent ${agentName} running in session ${sessionId} for invocation ${ctx.invocationId}`);
            // ... lógica del agente usando ctx ...
            yield; // ... evento ...
          }
        }
        ```

    === "Go"

        ```go
        import (
        	"google.golang.org/adk/agent"
        	"google.golang.org/adk/session"
        )

        --8<-- "examples/go/snippets/context/main.go:invocation_context_agent"
        ```

    === "Java"

        ```java
        // Pseudocódigo: Implementación de agente recibiendo InvocationContext
        import com.google.adk.agents.BaseAgent;
        import com.google.adk.agents.InvocationContext;

            LlmAgent root_agent =
                LlmAgent.builder()
                    .model("gemini-***")
                    .name("sample_agent")
                    .description("Answers user questions.")
                    .instruction(
                        """
                        provide instruction for the agent here.
                        """
                    )
                    .tools(sampleTool)
                    .outputKey("YOUR_KEY")
                    .build();

            ConcurrentMap<String, Object> initialState = new ConcurrentHashMap<>();
            initialState.put("YOUR_KEY", "");

            InMemoryRunner runner = new InMemoryRunner(agent);
            Session session =
                  runner
                      .sessionService()
                      .createSession(runner.appName(), USER_ID, initialState, SESSION_ID )
                      .blockingGet();

           try (Scanner scanner = new Scanner(System.in, StandardCharsets.UTF_8)) {
                while (true) {
                  System.out.print("\nYou > ");
                  String userInput = scanner.nextLine();

                  if ("quit".equalsIgnoreCase(userInput)) {
                    break;
                  }

                  Content userMsg = Content.fromParts(Part.fromText(userInput));
                  Flowable<Event> events =
                          runner.runAsync(session.userId(), session.id(), userMsg);

                  System.out.print("\nAgent > ");
                  events.blockingForEach(event ->
                          System.out.print(event.stringifyContent()));
              }

            protected Flowable<Event> runAsyncImpl(InvocationContext invocationContext) {
                // Ejemplo de acceso directo
                String agentName = invocationContext.agent.name
                String sessionId = invocationContext.session.id
                String invocationId = invocationContext.invocationId
                System.out.println("Agent " + agent_name + " running in session " + session_id + " for invocation " + invocationId)
                // ... lógica del agente usando ctx ...
            }
        ```

2.  **`ReadonlyContext`**
    *   **Dónde se Usa:** Proporcionado en escenarios donde solo se necesita acceso de lectura a información básica y la mutación no está permitida (por ejemplo, funciones `InstructionProvider`). También es la clase base para otros contextos.
    *   **Propósito:** Ofrece una vista segura y de solo lectura de detalles contextuales fundamentales.
    *   **Contenidos Clave:** `invocation_id`, `agent_name`, y una *vista* de solo lectura del `state` actual.

    === "Python"

        ```python
        # Pseudocódigo: Proveedor de instrucciones recibiendo ReadonlyContext
        from google.adk.agents.readonly_context import ReadonlyContext

        def my_instruction_provider(context: ReadonlyContext) -> str:
            # Ejemplo de acceso solo lectura
            user_tier = context.state().get("user_tier", "standard") # Puede leer estado
            # context.state['new_key'] = 'value' # Esto típicamente causaría un error o sería inefectivo
            return f"Process the request for a {user_tier} user."
        ```

    === "TypeScript"

        ```typescript
        // Pseudocódigo: Proveedor de instrucciones recibiendo ReadonlyContext
        import { ReadonlyContext } from '@google/adk';

        function myInstructionProvider(context: ReadonlyContext): string {
          // Ejemplo de acceso solo lectura
          // El objeto state es de solo lectura
          const userTier = context.state.get('user_tier') ?? 'standard';
          // context.state.set('new_key', 'value'); // Esto fallaría o lanzaría un error
          return `Process the request for a ${userTier} user.`;
        }
        ```

    === "Go"

        ```go
        import "google.golang.org/adk/agent"

        --8<-- "examples/go/snippets/context/main.go:readonly_context_instruction"
        ```

    === "Java"

        ```java
        // Pseudocódigo: Proveedor de instrucciones recibiendo ReadonlyContext
        import com.google.adk.agents.ReadonlyContext;

        public String myInstructionProvider(ReadonlyContext context){
            // Ejemplo de acceso solo lectura
            String userTier = context.state().get("user_tier", "standard");
            context.state().put('new_key', 'value'); // Esto típicamente causaría un error
            return "Process the request for a " + userTier + " user."
        }
        ```

3.  **`CallbackContext`**
    *   **Dónde se Usa:** Pasado como `callback_context` a callbacks del ciclo de vida del agente (`before_agent_callback`, `after_agent_callback`) y callbacks de interacción con el modelo (`before_model_callback`, `after_model_callback`).
    *   **Propósito:** Facilita inspeccionar y modificar estado, interactuar con artefactos, y acceder a detalles de invocación *específicamente dentro de callbacks*.
    *   **Capacidades Clave (Agrega a `ReadonlyContext`):**
        *   **Propiedad `state` Mutable:** Permite leer *y escribir* al estado de sesión. Los cambios hechos aquí (`callback_context.state['key'] = value`) se rastrean y asocian con el evento generado por el framework después del callback.
        *   **Métodos de Artefactos:** Métodos `load_artifact(filename)` y `save_artifact(filename, part)` para interactuar con el `artifact_service` configurado.
        *   Acceso directo a `user_content`.

    === "Python"

        ```python
        # Pseudocódigo: Callback recibiendo CallbackContext
        from google.adk.agents.callback_context import CallbackContext
        from google.adk.models import LlmRequest
        from google.genai import types
        from typing import Optional

        def my_before_model_cb(callback_context: CallbackContext, request: LlmRequest) -> Optional[types.Content]:
            # Ejemplo de lectura/escritura de estado
            call_count = callback_context.state.get("model_calls", 0)
            callback_context.state["model_calls"] = call_count + 1 # Modificar estado

            # Opcionalmente cargar un artefacto
            # config_part = callback_context.load_artifact("model_config.json")
            print(f"Preparing model call #{call_count + 1} for invocation {callback_context.invocation_id}")
            return None # Permitir que la llamada al modelo proceda
        ```

    === "TypeScript"

        ```typescript
        // Pseudocódigo: Callback recibiendo CallbackContext
        import { CallbackContext, LlmRequest } from '@google/adk';
        import { Content } from '@google/genai';

        function myBeforeModelCb(callbackContext: CallbackContext, request: LlmRequest): Content | undefined {
          // Ejemplo de lectura/escritura de estado
          const callCount = (callbackContext.state.get('model_calls') as number) || 0;
          callbackContext.state.set('model_calls', callCount + 1); // Modificar estado

          // Opcionalmente cargar un artefacto
          // const configPart = await callbackContext.loadArtifact('model_config.json');
          console.log(`Preparing model call #${callCount + 1} for invocation ${callbackContext.invocationId}`);
          return undefined; // Permitir que la llamada al modelo proceda
        }
        ```

    === "Go"

        ```go
        import (
        	"google.golang.org/adk/agent"
        	"google.golang.org/adk/model"
        )

        --8<-- "examples/go/snippets/context/main.go:callback_context_callback"
        ```

    === "Java"

        ```java
        // Pseudocódigo: Callback recibiendo CallbackContext
        import com.google.adk.agents.CallbackContext;
        import com.google.adk.models.LlmRequest;
        import com.google.genai.types.Content;
        import java.util.Optional;

        public Maybe<LlmResponse> myBeforeModelCb(CallbackContext callbackContext, LlmRequest request){
            // Ejemplo de lectura/escritura de estado
            callCount = callbackContext.state().get("model_calls", 0)
            callbackContext.state().put("model_calls") = callCount + 1 # Modificar estado

            // Opcionalmente cargar un artefacto
            // Maybe<Part> configPart = callbackContext.loadArtifact("model_config.json");
            System.out.println("Preparing model call " + callCount + 1);
            return Maybe.empty(); // Permitir que la llamada al modelo proceda
        }
        ```

4.  **`ToolContext`**
    *   **Dónde se Usa:** Pasado como `tool_context` a las funciones que respaldan `FunctionTool`s y a callbacks de ejecución de herramientas (`before_tool_callback`, `after_tool_callback`).
    *   **Propósito:** Proporciona todo lo que `CallbackContext` hace, más métodos especializados esenciales para la ejecución de herramientas, como manejar autenticación, buscar en memoria y listar artefactos.
    *   **Capacidades Clave (Agrega a `CallbackContext`):**
        *   **Métodos de Autenticación:** `request_credential(auth_config)` para activar un flujo de autenticación, y `get_auth_response(auth_config)` para recuperar credenciales proporcionadas por el usuario/sistema.
        *   **Listado de Artefactos:** `list_artifacts()` para descubrir artefactos disponibles en la sesión.
        *   **Búsqueda en Memoria:** `search_memory(query)` para consultar el `memory_service` configurado.
        *   **Propiedad `function_call_id`:** Identifica la llamada de función específica del LLM que activó esta ejecución de herramienta, crucial para vincular solicitudes o respuestas de autenticación correctamente.
        *   **Propiedad `actions`:** Acceso directo al objeto `EventActions` para este paso, permitiendo que la herramienta señale cambios de estado, solicitudes de autenticación, etc.

    === "Python"

        ```python
        # Pseudocódigo: Función de herramienta recibiendo ToolContext
        from google.adk.tools import ToolContext
        from typing import Dict, Any

        # Asumir que esta función está envuelta por un FunctionTool
        def search_external_api(query: str, tool_context: ToolContext) -> Dict[str, Any]:
            api_key = tool_context.state.get("api_key")
            if not api_key:
                # Definir configuración de autenticación requerida
                # auth_config = AuthConfig(...)
                # tool_context.request_credential(auth_config) # Solicitar credenciales
                # Usar la propiedad 'actions' para señalar que se ha hecho la solicitud de autenticación
                # tool_context.actions.requested_auth_configs[tool_context.function_call_id] = auth_config
                return {"status": "Auth Required"}

            # Usar la API key...
            print(f"Tool executing for query '{query}' using API key. Invocation: {tool_context.invocation_id}")

            # Opcionalmente buscar en memoria o listar artefactos
            # relevant_docs = tool_context.search_memory(f"info related to {query}")
            # available_files = tool_context.list_artifacts()

            return {"result": f"Data for {query} fetched."}
        ```

    === "TypeScript"

        ```typescript
        // Pseudocódigo: Función de herramienta recibiendo ToolContext
        import { ToolContext } from '@google/adk';

        // __Asumir que esta función está envuelta por un FunctionTool__
        function searchExternalApi(query: string, toolContext: ToolContext): { [key: string]: string } {
          const apiKey = toolContext.state.get('api_key') as string;
          if (!apiKey) {
             // Definir configuración de autenticación requerida
             // const authConfig = new AuthConfig(...);
             // toolContext.requestCredential(authConfig); // Solicitar credenciales
             // La propiedad 'actions' ahora se actualiza automáticamente por requestCredential
             return { status: 'Auth Required' };
          }

          // Usar la API key...
          console.log(`Tool executing for query '${query}' using API key. Invocation: ${toolContext.invocationId}`);

          // Opcionalmente buscar en memoria o listar artefactos
          // Nota: acceder a servicios como memoria/artefactos es típicamente async en TS,
          // así que necesitarías marcar esta función como 'async' si los reutilizaras.
          // toolContext.searchMemory(`info related to ${query}`).then(...)
          // toolContext.listArtifacts().then(...)

          return { result: `Data for ${query} fetched.` };
        }
        ```

    === "Go"

        ```go
        import "google.golang.org/adk/tool"

        --8<-- "examples/go/snippets/context/main.go:tool_context_tool"
        ```

    === "Java"

        ```java
        // Pseudocódigo: Función de herramienta recibiendo ToolContext
        import com.google.adk.tools.ToolContext;
        import java.util.HashMap;
        import java.util.Map;

        // Asumir que esta función está envuelta por un FunctionTool
        public Map<String, Object> searchExternalApi(String query, ToolContext toolContext){
            String apiKey = toolContext.state.get("api_key");
            if(apiKey.isEmpty()){
                // Definir configuración de autenticación requerida
                // authConfig = AuthConfig(...);
                // toolContext.requestCredential(authConfig); # Solicitar credenciales
                // Usar la propiedad 'actions' para señalar que se ha hecho la solicitud de autenticación
                ...
                return Map.of("status", "Auth Required");

            // Usar la API key...
            System.out.println("Tool executing for query " + query + " using API key. ");

            // Opcionalmente listar artefactos
            // Single<List<String>> availableFiles = toolContext.listArtifacts();

            return Map.of("result", "Data for " + query + " fetched");
        }
        ```

Entender estos diferentes objetos de contexto y cuándo usarlos es clave para gestionar efectivamente el estado, acceder a servicios y controlar el flujo de tu aplicación ADK. La siguiente sección detallará tareas comunes que puedes realizar usando estos contextos.

## Tareas Comunes Usando Contexto

Ahora que entiendes los diferentes objetos de contexto, enfoquémonos en cómo usarlos para tareas comunes al construir tus agentes y herramientas.

### Acceder a Información

Frecuentemente necesitarás leer información almacenada dentro del contexto.

*   **Leer Estado de Sesión:** Acceder datos guardados en pasos previos o configuraciones de nivel usuario/app. Usa acceso tipo diccionario en la propiedad `state`.

    === "Python"

        ```python
        # Pseudocódigo: En una función de Herramienta
        from google.adk.tools import ToolContext

        def my_tool(tool_context: ToolContext, **kwargs):
            user_pref = tool_context.state.get("user_display_preference", "default_mode")
            api_endpoint = tool_context.state.get("app:api_endpoint") # Leer estado a nivel app

            if user_pref == "dark_mode":
                # ... aplicar lógica de modo oscuro ...
                pass
            print(f"Using API endpoint: {api_endpoint}")
            # ... resto de la lógica de la herramienta ...

        # Pseudocódigo: En una función de Callback
        from google.adk.agents.callback_context import CallbackContext

        def my_callback(callback_context: CallbackContext, **kwargs):
            last_tool_result = callback_context.state.get("temp:last_api_result") # Leer estado temporal
            if last_tool_result:
                print(f"Found temporary result from last tool: {last_tool_result}")
            # ... lógica del callback ...
        ```

    === "TypeScript"

        ```typescript
        // Pseudocódigo: En una función de Herramienta
        import { ToolContext } from '@google/adk';

        async function myTool(toolContext: ToolContext) {
          const userPref = toolContext.state.get('user_display_preference', 'default_mode');
          const apiEndpoint = toolContext.state.get('app:api_endpoint'); // Leer estado a nivel app

          if (userPref === 'dark_mode') {
            // ... aplicar lógica de modo oscuro ...
          }
          console.log(`Using API endpoint: ${apiEndpoint}`);
          // ... resto de la lógica de la herramienta ...
        }

        // Pseudocódigo: En una función de Callback
        import { CallbackContext } from '@google/adk';

        function myCallback(callbackContext: CallbackContext) {
          const lastToolResult = callbackContext.state.get('temp:last_api_result'); // Leer estado temporal
          if (lastToolResult) {
            console.log(`Found temporary result from last tool: ${lastToolResult}`);
          }
          // ... lógica del callback ...
        }
        ```

    === "Go"

        ```go
        import (
        	"google.golang.org/adk/agent"
        	"google.golang.org/adk/session"
            "google.golang.org/adk/tool"
        	"google.golang.org/genai"
        )

        --8<-- "examples/go/snippets/context/main.go:accessing_state_tool"

        --8<-- "examples/go/snippets/context/main.go:accessing_state_callback"
        ```

    === "Java"

        ```java
        // Pseudocódigo: En una función de Herramienta
        import com.google.adk.tools.ToolContext;

        public void myTool(ToolContext toolContext){
           String userPref = toolContext.state().get("user_display_preference");
           String apiEndpoint = toolContext.state().get("app:api_endpoint"); // Leer estado a nivel app
           if(userPref.equals("dark_mode")){
                // ... aplicar lógica de modo oscuro ...
                pass
            }
           System.out.println("Using API endpoint: " + api_endpoint);
           // ... resto de la lógica de la herramienta ...
        }


        // Pseudocódigo: En una función de Callback
        import com.google.adk.agents.CallbackContext;

            public void myCallback(CallbackContext callbackContext){
                String lastToolResult = (String) callbackContext.state().get("temp:last_api_result"); // Leer estado temporal
            }
            if(!(lastToolResult.isEmpty())){
                System.out.println("Found temporary result from last tool: " + lastToolResult);
            }
            // ... lógica del callback ...
        ```

*   **Obtener Identificadores Actuales:** Útil para logging o lógica personalizada basada en la operación actual.

    === "Python"

        ```python
        # Pseudocódigo: En cualquier contexto (ToolContext mostrado)
        from google.adk.tools import ToolContext

        def log_tool_usage(tool_context: ToolContext, **kwargs):
            agent_name = tool_context.agent_name
            inv_id = tool_context.invocation_id
            func_call_id = getattr(tool_context, 'function_call_id', 'N/A') # Específico a ToolContext

            print(f"Log: Invocation={inv_id}, Agent={agent_name}, FunctionCallID={func_call_id} - Tool Executed.")
        ```

    === "TypeScript"

        ```typescript
        // Pseudocódigo: En cualquier contexto (ToolContext mostrado)
        import { ToolContext } from '@google/adk';

        function logToolUsage(toolContext: ToolContext) {
          const agentName = toolContext.agentName;
          const invId = toolContext.invocationId;
          const functionCallId = toolContext.functionCallId ?? 'N/A'; // Específico a ToolContext

          console.log(`Log: Invocation=${invId}, Agent=${agentName}, FunctionCallID=${functionCallId} - Tool Executed.`);
        }
        ```

    === "Go"

        ```go
        import "google.golang.org/adk/tool"

        --8<-- "examples/go/snippets/context/main.go:accessing_ids"
        ```

    === "Java"

        ```java
        // Pseudocódigo: En cualquier contexto (ToolContext mostrado)
         import com.google.adk.tools.ToolContext;

         public void logToolUsage(ToolContext toolContext){
                    String agentName = toolContext.agentName;
                    String invId = toolContext.invocationId;
                    String functionCallId = toolContext.functionCallId().get(); // Específico a ToolContext
                    System.out.println("Log: Invocation= " + invId &+ " Agent= " + agentName);
                }
        ```

*   **Acceder a la Entrada Inicial del Usuario:** Referirse de vuelta al mensaje que inició la invocación actual.

    === "Python"

        ```python
        # Pseudocódigo: En un Callback
        from google.adk.agents.callback_context import CallbackContext

        def check_initial_intent(callback_context: CallbackContext, **kwargs):
            initial_text = "N/A"
            if callback_context.user_content and callback_context.user_content.parts:
                initial_text = callback_context.user_content.parts[0].text or "Non-text input"

            print(f"This invocation started with user input: '{initial_text}'")

        # Pseudocódigo: En el _run_async_impl de un Agente
        # async def _run_async_impl(self, ctx: InvocationContext) -> AsyncGenerator[Event, None]:
        #     if ctx.user_content and ctx.user_content.parts:
        #         initial_text = ctx.user_content.parts[0].text
        #         print(f"Agent logic remembering initial query: {initial_text}")
        #     ...
        ```

    === "TypeScript"

        ```typescript
        // Pseudocódigo: En un Callback
        import { CallbackContext } from '@google/adk';

        function checkInitialIntent(callbackContext: CallbackContext) {
          let initialText = 'N/A';
          const userContent = callbackContext.userContent;
          if (userContent?.parts?.length) {
            initialText = userContent.parts[0].text ?? 'Non-text input';
          }

          console.log(`This invocation started with user input: '${initialText}'`);
        }
        ```

    === "Go"

        ```go
        import (
        	"google.golang.org/adk/agent"
        	"google.golang.org/genai"
        )

        --8<-- "examples/go/snippets/context/main.go:accessing_initial_user_input"
        ```

    === "Java"

        ```java
        // Pseudocódigo: En un Callback
        import com.google.adk.agents.CallbackContext;

        public void checkInitialIntent(CallbackContext callbackContext){
            String initialText = "N/A";
            if((!(callbackContext.userContent().isEmpty())) && (!(callbackContext.userContent().parts.isEmpty()))){
                initialText = cbx.userContent().get().parts().get().get(0).text().get();
                ...
                System.out.println("This invocation started with user input: " + initialText)
            }
        }
        ```

### Gestionar Estado

El estado es crucial para la memoria y el flujo de datos. Cuando modificas el estado usando `CallbackContext` o `ToolContext`, los cambios se rastrean y persisten automáticamente por el framework.

*   **Cómo Funciona:** Escribir a `callback_context.state['my_key'] = my_value` o `tool_context.state['my_key'] = my_value` agrega este cambio al `EventActions.state_delta` asociado con el evento del paso actual. El `SessionService` luego aplica estos deltas al persistir el evento.

*  **Pasar Datos Entre Herramientas**

    === "Python"

        ```python
        # Pseudocódigo: Herramienta 1 - Obtiene ID de usuario
        from google.adk.tools import ToolContext
        import uuid

        def get_user_profile(tool_context: ToolContext) -> dict:
            user_id = str(uuid.uuid4()) # Simular obtención de ID
            # Guardar el ID en el estado para la siguiente herramienta
            tool_context.state["temp:current_user_id"] = user_id
            return {"profile_status": "ID generated"}

        # Pseudocódigo: Herramienta 2 - Usa ID de usuario del estado
        def get_user_orders(tool_context: ToolContext) -> dict:
            user_id = tool_context.state.get("temp:current_user_id")
            if not user_id:
                return {"error": "User ID not found in state"}

            print(f"Fetching orders for user ID: {user_id}")
            # ... lógica para obtener órdenes usando user_id ...
            return {"orders": ["order123", "order456"]}
        ```

    === "TypeScript"

        ```typescript
        // Pseudocódigo: Herramienta 1 - Obtiene ID de usuario
        import { ToolContext } from '@google/adk';
        import { v4 as uuidv4 } from 'uuid';

        function getUserProfile(toolContext: ToolContext): Record<string, string> {
          const userId = uuidv4(); // Simular obtención de ID
          // Guardar el ID en el estado para la siguiente herramienta
          toolContext.state.set('temp:current_user_id', userId);
          return { profile_status: 'ID generated' };
        }

        // Pseudocódigo: Herramienta 2 - Usa ID de usuario del estado
        function getUserOrders(toolContext: ToolContext): Record<string, string | string[]> {
          const userId = toolContext.state.get('temp:current_user_id');
          if (!userId) {
            return { error: 'User ID not found in state' };
          }

          console.log(`Fetching orders for user ID: ${userId}`);
          // ... lógica para obtener órdenes usando user_id ...
          return { orders: ['order123', 'order456'] };
        }
        ```

    === "Go"

        ```go
        import "google.golang.org/adk/tool"

        --8<-- "examples/go/snippets/context/main.go:passing_data_tool1"

        --8<-- "examples/go/snippets/context/main.go:passing_data_tool2"
        ```

    === "Java"

        ```java
        // Pseudocódigo: Herramienta 1 - Obtiene ID de usuario
        import com.google.adk.tools.ToolContext;
        import java.util.UUID;

        public Map<String, String> getUserProfile(ToolContext toolContext){
            String userId = UUID.randomUUID().toString();
            // Guardar el ID en el estado para la siguiente herramienta
            toolContext.state().put("temp:current_user_id", user_id);
            return Map.of("profile_status", "ID generated");
        }

        // Pseudocódigo: Herramienta 2 - Usa ID de usuario del estado
        public  Map<String, String> getUserOrders(ToolContext toolContext){
            String userId = toolContext.state().get("temp:current_user_id");
            if(userId.isEmpty()){
                return Map.of("error", "User ID not found in state");
            }
            System.out.println("Fetching orders for user id: " + userId);
             // ... lógica para obtener órdenes usando user_id ...
            return Map.of("orders", "order123");
        }
        ```

*   **Actualizar Preferencias del Usuario:**

    === "Python"

        ```python
        # Pseudocódigo: Herramienta o Callback identifica una preferencia
        from google.adk.tools import ToolContext # O CallbackContext

        def set_user_preference(tool_context: ToolContext, preference: str, value: str) -> dict:
            # Usar prefijo 'user:' para estado a nivel usuario (si se usa un SessionService persistente)
            state_key = f"user:{preference}"
            tool_context.state[state_key] = value
            print(f"Set user preference '{preference}' to '{value}'")
            return {"status": "Preference updated"}
        ```

    === "TypeScript"

        ```typescript
        // Pseudocódigo: Herramienta o Callback identifica una preferencia
        import { ToolContext } from '@google/adk'; // O CallbackContext

        function setUserPreference(toolContext: ToolContext, preference: string, value: string): Record<string, string> {
          // Usar prefijo 'user:' para estado a nivel usuario (si se usa un SessionService persistente)
          const stateKey = `user:${preference}`;
          toolContext.state.set(stateKey, value);
          console.log(`Set user preference '${preference}' to '${value}'`);
          return { status: 'Preference updated' };
        }
        ```

    === "Go"

        ```go
        import "google.golang.org/adk/tool"

        --8<-- "examples/go/snippets/context/main.go:updating_preferences"
        ```

    === "Java"

        ```java
        // Pseudocódigo: Herramienta o Callback identifica una preferencia
        import com.google.adk.tools.ToolContext; // O CallbackContext

        public Map<String, String> setUserPreference(ToolContext toolContext, String preference, String value){
            // Usar prefijo 'user:' para estado a nivel usuario (si se usa un SessionService persistente)
            String stateKey = "user:" + preference;
            toolContext.state().put(stateKey, value);
            System.out.println("Set user preference '" + preference + "' to '" + value + "'");
            return Map.of("status", "Preference updated");
        }
        ```

*   **Prefijos de Estado:** Mientras que el estado básico es específico de la sesión, prefijos como `app:` y `user:` pueden usarse con implementaciones persistentes de `SessionService` (como `DatabaseSessionService` o `VertexAiSessionService`) para indicar un alcance más amplio (a nivel app o usuario a través de sesiones). `temp:` puede denotar datos solo relevantes dentro de la invocación actual.

### Trabajar con Artefactos

Usa artefactos para manejar archivos o blobs de datos grandes asociados con la sesión. Caso de uso común: procesar documentos cargados.

*   **Ejemplo de Flujo de Resumidor de Documentos:**

    1.  **Ingerir Referencia (por ejemplo, en una Herramienta de Configuración o Callback):** Guardar la *ruta o URI* del documento, no el contenido completo, como un artefacto.

        === "Python"

               ```python
               # Pseudocódigo: En un callback o herramienta inicial
               from google.adk.agents.callback_context import CallbackContext # O ToolContext
               from google.genai import types

               def save_document_reference(context: CallbackContext, file_path: str) -> None:
                   # Asumir que file_path es algo como "gs://my-bucket/docs/report.pdf" o "/local/path/to/report.pdf"
                   try:
                       # Crear un Part conteniendo el texto de la ruta/URI
                       artifact_part = types.Part(text=file_path)
                       version = context.save_artifact("document_to_summarize.txt", artifact_part)
                       print(f"Saved document reference '{file_path}' as artifact version {version}")
                       # Almacenar el nombre del archivo en el estado si es necesario para otras herramientas
                       context.state["temp:doc_artifact_name"] = "document_to_summarize.txt"
                   except ValueError as e:
                       print(f"Error saving artifact: {e}") # Por ejemplo, Servicio de artefactos no configurado
                   except Exception as e:
                       print(f"Unexpected error saving artifact reference: {e}")

               # Ejemplo de uso:
               # save_document_reference(callback_context, "gs://my-bucket/docs/report.pdf")
               ```

        === "TypeScript"

               ```typescript
               // Pseudocódigo: En un callback o herramienta inicial
               import { CallbackContext } from '@google/adk'; // O ToolContext
               import type { Part } from '@google/genai';

               async function saveDocumentReference(context: CallbackContext, filePath: string) {
                 // Asumir que filePath es algo como "gs://my-bucket/docs/report.pdf" o "/local/path/to/report.pdf"
                 try {
                   // Crear un Part conteniendo el texto de la ruta/URI
                   const artifactPart: Part = { text: filePath };
                   const version = await context.saveArtifact('document_to_summarize.txt', artifactPart);
                   console.log(`Saved document reference '${filePath}' as artifact version ${version}`);
                   // Almacenar el nombre del archivo en el estado si es necesario para otras herramientas
                   context.state.set('temp:doc_artifact_name', 'document_to_summarize.txt');
                 } catch (e) {
                   console.error(`Unexpected error saving artifact reference: ${e}`);
                 }
               }

               // Ejemplo de uso:
               // saveDocumentReference(callbackContext, "gs://my-bucket/docs/report.pdf");
               ```

        === "Go"

            ```go
            import (
            	"google.golang.org/adk/tool"
            	"google.golang.org/genai"
            )

            --8<-- "examples/go/snippets/context/main.go:artifacts_save_ref"
            ```

        === "Java"

               ```java
               // Pseudocódigo: En un callback o herramienta inicial
               import com.google.adk.agents.CallbackContext;
               import com.google.genai.types.Content;
               import com.google.genai.types.Part;


               pubic void saveDocumentReference(CallbackContext context, String filePath){
                   // Asumir que file_path es algo como "gs://my-bucket/docs/report.pdf" o "/local/path/to/report.pdf"
                   try{
                       // Crear un Part conteniendo el texto de la ruta/URI
                       Part artifactPart = types.Part(filePath)
                       Optional<Integer> version = context.saveArtifact("document_to_summarize.txt", artifactPart)
                       System.out.println("Saved document reference" + filePath + " as artifact version " + version);
                       // Almacenar el nombre del archivo en el estado si es necesario para otras herramientas
                       context.state().put("temp:doc_artifact_name", "document_to_summarize.txt");
                   } catch(Exception e){
                       System.out.println("Unexpected error saving artifact reference: " + e);
                   }
               }

               // Ejemplo de uso:
               // saveDocumentReference(context, "gs://my-bucket/docs/report.pdf")
               ```

    2.  **Herramienta Resumidora:** Cargar el artefacto para obtener la ruta/URI, leer el contenido real del documento usando bibliotecas apropiadas, resumir y devolver el resultado.

        === "Python"

            ```python
            # Pseudocódigo: En la función de herramienta Resumidora
            from google.adk.tools import ToolContext
            from google.genai import types
            # Asumir que bibliotecas como google.cloud.storage o open integrado están disponibles
            # Asumir que existe una función 'summarize_text'
            # from my_summarizer_lib import summarize_text

            def summarize_document_tool(tool_context: ToolContext) -> dict:
                artifact_name = tool_context.state.get("temp:doc_artifact_name")
                if not artifact_name:
                    return {"error": "Document artifact name not found in state."}

                try:
                    # 1. Cargar el part del artefacto conteniendo la ruta/URI
                    artifact_part = tool_context.load_artifact(artifact_name)
                    if not artifact_part or not artifact_part.text:
                        return {"error": f"Could not load artifact or artifact has no text path: {artifact_name}"}

                    file_path = artifact_part.text
                    print(f"Loaded document reference: {file_path}")

                    # 2. Leer el contenido real del documento (fuera del contexto ADK)
                    document_content = ""
                    if file_path.startswith("gs://"):
                        # Ejemplo: Usar biblioteca cliente GCS para descargar/leer
                        # from google.cloud import storage
                        # client = storage.Client()
                        # blob = storage.Blob.from_string(file_path, client=client)
                        # document_content = blob.download_as_text() # O bytes dependiendo del formato
                        pass # Reemplazar con lógica real de lectura GCS
                    elif file_path.startswith("/"):
                         # Ejemplo: Usar sistema de archivos local
                         with open(file_path, 'r', encoding='utf-8') as f:
                             document_content = f.read()
                    else:
                        return {"error": f"Unsupported file path scheme: {file_path}"}

                    # 3. Resumir el contenido
                    if not document_content:
                         return {"error": "Failed to read document content."}

                    # summary = summarize_text(document_content) # Llamar tu lógica de resumen
                    summary = f"Summary of content from {file_path}" # Marcador de posición

                    return {"summary": summary}

                except ValueError as e:
                     return {"error": f"Artifact service error: {e}"}
                except FileNotFoundError:
                     return {"error": f"Local file not found: {file_path}"}
                # except Exception as e: # Capturar excepciones específicas para GCS etc.
                #      return {"error": f"Error reading document {file_path}: {e}"}
            ```

        === "TypeScript"

            ```typescript
            // Pseudocódigo: En la función de herramienta Resumidora
            import { ToolContext } from '@google/adk';

            async function summarizeDocumentTool(toolContext: ToolContext): Promise<Record<string, string>> {
              const artifactName = toolContext.state.get('temp:doc_artifact_name') as string;
              if (!artifactName) {
                return { error: 'Document artifact name not found in state.' };
              }

              try {
                // 1. Cargar el part del artefacto conteniendo la ruta/URI
                const artifactPart = await toolContext.loadArtifact(artifactName);
                if (!artifactPart?.text) {
                  return { error: `Could not load artifact or artifact has no text path: ${artifactName}` };
                }

                const filePath = artifactPart.text;
                console.log(`Loaded document reference: ${filePath}`);

                // 2. Leer el contenido real del documento (fuera del contexto ADK)
                let documentContent = '';
                if (filePath.startsWith('gs://')) {
                  // Ejemplo: Usar biblioteca cliente GCS para descargar/leer
                  // const storage = new Storage();
                  // const bucket = storage.bucket('my-bucket');
                  // const file = bucket.file(filePath.replace('gs://my-bucket/', ''));
                  // const [contents] = await file.download();
                  // documentContent = contents.toString();
                } else if (filePath.startsWith('/')) {
                  // Ejemplo: Usar sistema de archivos local
                  // import { readFile } from 'fs/promises';
                  // documentContent = await readFile(filePath, 'utf8');
                } else {
                  return { error: `Unsupported file path scheme: ${filePath}` };
                }

                // 3. Resumir el contenido
                if (!documentContent) {
                   return { error: 'Failed to read document content.' };
                }

                // const summary = summarizeText(documentContent); // Llamar tu lógica de resumen
                const summary = `Summary of content from ${filePath}`; // Marcador de posición

                return { summary };

              } catch (e) {
                 return { error: `Error processing artifact: ${e}` };
              }
            }
            ```

        === "Go"

            ```go
            import "google.golang.org/adk/tool"

            --8<-- "examples/go/snippets/context/main.go:artifacts_summarize"
            ```

        === "Java"

            ```java
            // Pseudocódigo: En la función de herramienta Resumidora
            import com.google.adk.tools.ToolContext;
            import com.google.genai.types.Content;
            import com.google.genai.types.Part;

            public Map<String, String> summarizeDocumentTool(ToolContext toolContext){
                String artifactName = toolContext.state().get("temp:doc_artifact_name");
                if(artifactName.isEmpty()){
                    return Map.of("error", "Document artifact name not found in state.");
                }
                try{
                    // 1. Cargar el part del artefacto conteniendo la ruta/URI
                    Maybe<Part> artifactPart = toolContext.loadArtifact(artifactName);
                    if((artifactPart == null) || (artifactPart.text().isEmpty())){
                        return Map.of("error", "Could not load artifact or artifact has no text path: " + artifactName);
                    }
                    filePath = artifactPart.text();
                    System.out.println("Loaded document reference: " + filePath);

                    // 2. Leer el contenido real del documento (fuera del contexto ADK)
                    String documentContent = "";
                    if(filePath.startsWith("gs://")){
                        // Ejemplo: Usar biblioteca cliente GCS para descargar/leer en documentContent
                        pass; // Reemplazar con lógica real de lectura GCS
                    } else if(){
                        // Ejemplo: Usar sistema de archivos local para descargar/leer en documentContent
                    } else{
                        return Map.of("error", "Unsupported file path scheme: " + filePath);
                    }

                    // 3. Resumir el contenido
                    if(documentContent.isEmpty()){
                        return Map.of("error", "Failed to read document content.");
                    }

                    // summary = summarizeText(documentContent) // Llamar tu lógica de resumen
                    summary = "Summary of content from " + filePath; // Marcador de posición

                    return Map.of("summary", summary);
                } catch(IllegalArgumentException e){
                    return Map.of("error", "Artifact service error " + filePath + e);
                } catch(FileNotFoundException e){
                    return Map.of("error", "Local file not found " + filePath + e);
                } catch(Exception e){
                    return Map.of("error", "Error reading document " + filePath + e);
                }
            }
            ```

*   **Listar Artefactos:** Descubrir qué archivos están disponibles.

    === "Python"

        ```python
        # Pseudocódigo: En una función de herramienta
        from google.adk.tools import ToolContext

        def check_available_docs(tool_context: ToolContext) -> dict:
            try:
                artifact_keys = tool_context.list_artifacts()
                print(f"Available artifacts: {artifact_keys}")
                return {"available_docs": artifact_keys}
            except ValueError as e:
                return {"error": f"Artifact service error: {e}"}
        ```

    === "TypeScript"

        ```typescript
        // Pseudocódigo: En una función de herramienta
        import { ToolContext } from '@google/adk';

        async function checkAvailableDocs(toolContext: ToolContext): Promise<Record<string, string[] | string>> {
          try {
            const artifactKeys = await toolContext.listArtifacts();
            console.log(`Available artifacts: ${artifactKeys}`);
            return { available_docs: artifactKeys };
          } catch (e) {
            return { error: `Artifact service error: ${e}` };
          }
        }
        ```

    === "Go"

        ```go
        import "google.golang.org/adk/tool"

        --8<-- "examples/go/snippets/context/main.go:artifacts_list"
        ```

    === "Java"

        ```java
        // Pseudocódigo: En una función de herramienta
        import com.google.adk.tools.ToolContext;

        public Map<String, String> checkAvailableDocs(ToolContext toolContext){
            try{
                Single<List<String>> artifactKeys = toolContext.listArtifacts();
                System.out.println("Available artifacts" + artifactKeys.tostring());
                return Map.of("availableDocs", "artifactKeys");
            } catch(IllegalArgumentException e){
                return Map.of("error", "Artifact service error: " + e);
            }
        }
        ```

### Manejar Autenticación de Herramientas

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

Gestionar de manera segura API keys u otras credenciales necesarias para las herramientas.

=== "Python"

    ```python
    # Pseudocódigo: Herramienta que requiere autenticación
    from google.adk.tools import ToolContext
    from google.adk.auth import AuthConfig # Asumir que se define un AuthConfig apropiado

    # Definir tu configuración de autenticación requerida (por ejemplo, OAuth, API Key)
    MY_API_AUTH_CONFIG = AuthConfig(...)
    AUTH_STATE_KEY = "user:my_api_credential" # Clave para almacenar la credencial recuperada

    def call_secure_api(tool_context: ToolContext, request_data: str) -> dict:
        # 1. Verificar si la credencial ya existe en el estado
        credential = tool_context.state.get(AUTH_STATE_KEY)

        if not credential:
            # 2. Si no, solicitarla
            print("Credential not found, requesting...")
            try:
                tool_context.request_credential(MY_API_AUTH_CONFIG)
                # El framework maneja el yield del evento. La ejecución de la herramienta se detiene aquí para este turno.
                return {"status": "Authentication required. Please provide credentials."}
            except ValueError as e:
                return {"error": f"Auth error: {e}"} # por ejemplo, function_call_id faltante
            except Exception as e:
                return {"error": f"Failed to request credential: {e}"}

        # 3. Si la credencial existe (podría ser de un turno previo después de la solicitud)
        #    o si esta es una llamada subsiguiente después de que el flujo de autenticación se completó externamente
        try:
            # Opcionalmente, re-validar/recuperar si es necesario, o usar directamente
            # Esto podría recuperar la credencial si el flujo externo acaba de completarse
            auth_credential_obj = tool_context.get_auth_response(MY_API_AUTH_CONFIG)
            api_key = auth_credential_obj.api_key # O access_token, etc.

            # Almacenarla de vuelta en el estado para futuras llamadas dentro de la sesión
            tool_context.state[AUTH_STATE_KEY] = auth_credential_obj.model_dump() # Persistir credencial recuperada

            print(f"Using retrieved credential to call API with data: {request_data}")
            # ... Hacer la llamada real a la API usando api_key ...
            api_result = f"API result for {request_data}"

            return {"result": api_result}
        except Exception as e:
            # Manejar errores al recuperar/usar la credencial
            print(f"Error using credential: {e}")
            # Tal vez limpiar la clave de estado si la credencial es inválida?
            # tool_context.state[AUTH_STATE_KEY] = None
            return {"error": "Failed to use credential"}
    ```

=== "TypeScript"

    ```typescript
    // Pseudocódigo: Herramienta que requiere autenticación
    import { ToolContext } from '@google/adk'; // AuthConfig de ADK o personalizado

    // Definir una interfaz local AuthConfig ya que no es exportada públicamente por ADK
    interface AuthConfig {
      credentialKey: string;
      authScheme: { type: string }; // Representación mínima para el ejemplo
      // Agregar otras propiedades si se vuelven relevantes para el ejemplo
    }

    // Definir tu configuración de autenticación requerida (por ejemplo, OAuth, API Key)
    const MY_API_AUTH_CONFIG: AuthConfig = {
      credentialKey: 'my-api-key', // Ejemplo de clave
      authScheme: { type: 'api-key' }, // Ejemplo de tipo de esquema
    };
    const AUTH_STATE_KEY = 'user:my_api_credential'; // Clave para almacenar la credencial