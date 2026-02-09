# State: El Bloc de Notas de la Sesión

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Dentro de cada `Session` (nuestro hilo de conversación), el atributo **`state`** actúa como el bloc de notas dedicado del agente para esa interacción específica. Mientras que `session.events` contiene el historial completo, `session.state` es donde el agente almacena y actualiza los detalles dinámicos necesarios *durante* la conversación.

## ¿Qué es `session.state`?

Conceptualmente, `session.state` es una colección (diccionario o Map) que contiene pares clave-valor. Está diseñado para información que el agente necesita recordar o rastrear para hacer efectiva la conversación actual:

* **Personalizar la Interacción:** Recordar preferencias del usuario mencionadas anteriormente (ej., `'user_preference_theme': 'dark'`).
* **Rastrear el Progreso de Tareas:** Mantener un seguimiento de los pasos en un proceso de múltiples turnos (ej., `'booking_step': 'confirm_payment'`).
* **Acumular Información:** Construir listas o resúmenes (ej., `'shopping_cart_items': ['book', 'pen']`).
* **Tomar Decisiones Informadas:** Almacenar banderas o valores que influyen en la próxima respuesta (ej., `'user_is_authenticated': True`).

### Características Clave de `State`

1. **Estructura: Pares Clave-Valor Serializables**

    * Los datos se almacenan como `key: value`.
    * **Claves:** Siempre cadenas de texto (`str`). Usa nombres claros (ej., `'departure_city'`, `'user:language_preference'`).
    * **Valores:** Deben ser **serializables**. Esto significa que pueden guardarse y cargarse fácilmente por el `SessionService`. Usa tipos básicos en los lenguajes específicos (Python/Go/Java/TypeScript) como cadenas, números, booleanos y listas o diccionarios simples que contengan *solo* estos tipos básicos. (Ver documentación de la API para detalles precisos).
    * **⚠️ Evita Objetos Complejos:** **No almacenes objetos no serializables** (instancias de clases personalizadas, funciones, conexiones, etc.) directamente en el estado. Almacena identificadores simples si es necesario, y recupera el objeto complejo en otro lugar.

2. **Mutabilidad: Cambia**

    * Se espera que el contenido del `state` cambie a medida que la conversación evoluciona.

3. **Persistencia: Depende del `SessionService`**

    * Si el estado sobrevive a los reinicios de la aplicación depende del servicio elegido:

      * `InMemorySessionService`: **No Persistente.** El estado se pierde al reiniciar.
      * `DatabaseSessionService` / `VertexAiSessionService`: **Persistente.** El estado se guarda de forma confiable.

!!! Note
    Los parámetros específicos o nombres de métodos para las primitivas pueden variar ligeramente según el lenguaje del SDK (ej., `session.state['current_intent'] = 'book_flight'` en Python, `context.State().Set("current_intent", "book_flight")` en Go, `session.state().put("current_intent", "book_flight)` en Java, o `context.state.set("current_intent", "book_flight")` en TypeScript). Consulta la documentación de la API específica del lenguaje para detalles.

### Organizando el State con Prefijos: El Alcance Importa

Los prefijos en las claves del estado definen su alcance y comportamiento de persistencia, especialmente con servicios persistentes:

* **Sin Prefijo (Estado de Sesión):**

    * **Alcance:** Específico a la sesión *actual* (`id`).
    * **Persistencia:** Solo persiste si el `SessionService` es persistente (`Database`, `VertexAI`).
    * **Casos de Uso:** Rastrear el progreso dentro de la tarea actual (ej., `'current_booking_step'`), banderas temporales para esta interacción (ej., `'needs_clarification'`).
    * **Ejemplo:** `session.state['current_intent'] = 'book_flight'`

* **Prefijo `user:` (Estado de Usuario):**

    * **Alcance:** Vinculado al `user_id`, compartido entre *todas* las sesiones para ese usuario (dentro del mismo `app_name`).
    * **Persistencia:** Persistente con `Database` o `VertexAI`. (Almacenado por `InMemory` pero se pierde al reiniciar).
    * **Casos de Uso:** Preferencias de usuario (ej., `'user:theme'`), detalles de perfil (ej., `'user:name'`).
    * **Ejemplo:** `session.state['user:preferred_language'] = 'fr'`

* **Prefijo `app:` (Estado de Aplicación):**

    * **Alcance:** Vinculado al `app_name`, compartido entre *todos* los usuarios y sesiones para esa aplicación.
    * **Persistencia:** Persistente con `Database` o `VertexAI`. (Almacenado por `InMemory` pero se pierde al reiniciar).
    * **Casos de Uso:** Configuraciones globales (ej., `'app:api_endpoint'`), plantillas compartidas.
    * **Ejemplo:** `session.state['app:global_discount_code'] = 'SAVE10'`

* **Prefijo `temp:` (Estado de Invocación Temporal):**

    * **Alcance:** Específico a la **invocación** actual (el proceso completo desde que un agente recibe la entrada del usuario hasta generar la salida final para esa entrada).
    * **Persistencia:** **No Persistente.** Se descarta después de que la invocación se completa y no se traslada a la siguiente.
    * **Casos de Uso:** Almacenar cálculos intermedios, banderas o datos pasados entre llamadas a herramientas dentro de una sola invocación.
    * **Cuándo No Usar:** Para información que debe persistir entre diferentes invocaciones, como preferencias de usuario, resúmenes del historial de conversación o datos acumulados.
    * **Ejemplo:** `session.state['temp:raw_api_response'] = {...}`

!!! note "Sub-Agentes y Contexto de Invocación"
    Cuando un agente padre llama a un sub-agente (ej., usando `SequentialAgent` o `ParallelAgent`), pasa su `InvocationContext` al sub-agente. Esto significa que toda la cadena de llamadas de agentes comparte el mismo ID de invocación y, por lo tanto, el mismo estado `temp:`.

**Cómo lo Ve el Agente:** Tu código del agente interactúa con el estado *combinado* a través de la colección única `session.state` (dict/Map). El `SessionService` maneja la obtención/fusión del estado desde el almacenamiento subyacente correcto basándose en los prefijos.

### Accediendo al Estado de Sesión en las Instrucciones del Agente

Al trabajar con instancias de `LlmAgent`, puedes inyectar directamente valores del estado de sesión en la cadena de instrucción del agente usando una sintaxis de plantillas simple. Esto te permite crear instrucciones dinámicas y conscientes del contexto sin depender únicamente de directivas en lenguaje natural.

#### Usando Plantillas `{key}`

Para inyectar un valor del estado de sesión, encierra la clave de la variable de estado deseada entre llaves: `{key}`. El framework reemplazará automáticamente este marcador de posición con el valor correspondiente de `session.state` antes de pasar la instrucción al LLM.

**Ejemplo:**

=== "Python"

    ```python
    from google.adk.agents import LlmAgent

    story_generator = LlmAgent(
        name="StoryGenerator",
        model="gemini-2.0-flash",
        instruction="""Write a short story about a cat, focusing on the theme: {topic}."""
    )

    # Asumiendo que session.state['topic'] está configurado como "friendship", el LLM
    # recibirá la siguiente instrucción:
    # "Write a short story about a cat, focusing on the theme: friendship."
    ```

=== "TypeScript"

    ```typescript
    import { LlmAgent } from "@google/adk";

    const storyGenerator = new LlmAgent({
        name: "StoryGenerator",
        model: "gemini-2.5-flash",
        instruction: "Write a short story about a cat, focusing on the theme: {topic}."
    });

    // Asumiendo que session.state['topic'] está configurado como "friendship", el LLM
    // recibirá la siguiente instrucción:
    // "Write a short story about a cat, focusing on the theme: friendship."
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/sessions/instruction_template/instruction_template_example.go:key_template"
    ```

#### Consideraciones Importantes

* Existencia de Clave: Asegúrate de que la clave que referencias en la cadena de instrucción exista en session.state. Si la clave falta, el agente arrojará un error. Para usar una clave que puede o no estar presente, puedes incluir un signo de interrogación (?) después de la clave (ej. {topic?}).
* Tipos de Datos: El valor asociado con la clave debe ser una cadena de texto o un tipo que pueda convertirse fácilmente a una cadena.
* Escapado: Si necesitas usar llaves literales en tu instrucción (ej., para formato JSON), necesitarás escaparlas.

#### Evitando la Inyección de Estado con `InstructionProvider`

En algunos casos, podrías querer usar `{{` y `}}` literalmente en tus instrucciones sin activar el mecanismo de inyección de estado. Por ejemplo, podrías estar escribiendo instrucciones para un agente que ayuda con un lenguaje de plantillas que usa la misma sintaxis.

Para lograr esto, puedes proporcionar una función al parámetro `instruction` en lugar de una cadena. Esta función se llama `InstructionProvider`. Cuando usas un `InstructionProvider`, el ADK no intentará inyectar estado, y tu cadena de instrucción se pasará al modelo tal como está.

La función `InstructionProvider` recibe un objeto `ReadonlyContext`, que puedes usar para acceder al estado de sesión u otra información contextual si necesitas construir la instrucción dinámicamente.

=== "Python"

    ```python
    from google.adk.agents import LlmAgent
    from google.adk.agents.readonly_context import ReadonlyContext

    # Esto es un InstructionProvider
    def my_instruction_provider(context: ReadonlyContext) -> str:
        # Opcionalmente puedes usar el contexto para construir la instrucción
        # Para este ejemplo, devolveremos una cadena estática con llaves literales.
        return "This is an instruction with {{literal_braces}} that will not be replaced."

    agent = LlmAgent(
        model="gemini-2.0-flash",
        name="template_helper_agent",
        instruction=my_instruction_provider
    )
    ```

=== "TypeScript"

    ```typescript
    import { LlmAgent, ReadonlyContext } from "@google/adk";

    // Esto es un InstructionProvider
    function myInstructionProvider(context: ReadonlyContext): string {
        // Opcionalmente puedes usar el contexto para construir la instrucción
        // Para este ejemplo, devolveremos una cadena estática con llaves literales.
        return "This is an instruction with {{literal_braces}} that will not be replaced.";
    }

    const agent = new LlmAgent({
        model: "gemini-2.5-flash",
        name: "template_helper_agent",
        instruction: myInstructionProvider
    });
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/sessions/instruction_provider/instruction_provider_example.go:bypass_state_injection"
    ```

Si quieres tanto usar un `InstructionProvider` *como* inyectar estado en tus instrucciones, puedes usar la función de utilidad `inject_session_state`.

=== "Python"

    ```python
    from google.adk.agents import LlmAgent
    from google.adk.agents.readonly_context import ReadonlyContext
    from google.adk.utils import instructions_utils

    async def my_dynamic_instruction_provider(context: ReadonlyContext) -> str:
        template = "This is a {adjective} instruction with {{literal_braces}}."
        # Esto inyectará la variable de estado 'adjective' pero dejará las llaves literales.
        return await instructions_utils.inject_session_state(template, context)

    agent = LlmAgent(
        model="gemini-2.0-flash",
        name="dynamic_template_helper_agent",
        instruction=my_dynamic_instruction_provider
    )
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/sessions/instruction_provider/instruction_provider_example.go:manual_state_injection"
    ```

**Beneficios de la Inyección Directa**

* Claridad: Hace explícito qué partes de la instrucción son dinámicas y basadas en el estado de sesión.
* Confiabilidad: Evita depender del LLM para interpretar correctamente instrucciones en lenguaje natural para acceder al estado.
* Mantenibilidad: Simplifica las cadenas de instrucción y reduce el riesgo de errores al actualizar nombres de variables de estado.

**Relación con Otros Métodos de Acceso al Estado**

Este método de inyección directo es específico para las instrucciones de LlmAgent. Consulta la siguiente sección para más información sobre otros métodos de acceso al estado.

### Cómo se Actualiza el Estado: Métodos Recomendados

!!! note "La Forma Correcta de Modificar el Estado"
    Cuando necesitas cambiar el estado de sesión, el método correcto y más seguro es **modificar directamente el objeto `state` en el `Context`** proporcionado a tu función (ej., `callback_context.state['my_key'] = 'new_value'`). Esto se considera "manipulación directa del estado" de la manera correcta, ya que el framework rastrea automáticamente estos cambios.

    Esto es críticamente diferente de modificar directamente el `state` en un objeto `Session` que obtienes del `SessionService` (ej., `my_session.state['my_key'] = 'new_value'`). **Debes evitar esto**, ya que evita el seguimiento de eventos del ADK y puede llevar a la pérdida de datos. La sección "Advertencia" al final de esta página tiene más detalles sobre esta distinción importante.

El estado **siempre** debe actualizarse como parte de agregar un `Event` al historial de la sesión usando `session_service.append_event()`. Esto asegura que los cambios se rastreen, la persistencia funcione correctamente y las actualizaciones sean seguras para hilos.

**1\. La Manera Fácil: `output_key` (para Respuestas de Texto del Agente)**

Este es el método más simple para guardar la respuesta de texto final de un agente directamente en el estado. Al definir tu `LlmAgent`, especifica el `output_key`:

=== "Python"

    ```py
    from google.adk.agents import LlmAgent
    from google.adk.sessions import InMemorySessionService, Session
    from google.adk.runners import Runner
    from google.genai.types import Content, Part

    # Definir agente con output_key
    greeting_agent = LlmAgent(
        name="Greeter",
        model="gemini-2.0-flash", # Usa un modelo válido
        instruction="Generate a short, friendly greeting.",
        output_key="last_greeting" # Guardar respuesta en state['last_greeting']
    )

    # --- Configurar Runner y Session ---
    app_name, user_id, session_id = "state_app", "user1", "session1"
    session_service = InMemorySessionService()
    runner = Runner(
        agent=greeting_agent,
        app_name=app_name,
        session_service=session_service
    )
    session = await session_service.create_session(app_name=app_name,
                                        user_id=user_id,
                                        session_id=session_id)
    print(f"Initial state: {session.state}")

    # --- Ejecutar el Agente ---
    # Runner maneja la llamada a append_event, que usa el output_key
    # para crear automáticamente el state_delta.
    user_message = Content(parts=[Part(text="Hello")])
    for event in runner.run(user_id=user_id,
                            session_id=session_id,
                            new_message=user_message):
        if event.is_final_response():
          print(f"Agent responded.") # El texto de respuesta también está en event.content

    # --- Verificar el Estado Actualizado ---
    updated_session = await session_service.get_session(app_name=APP_NAME, user_id=USER_ID, session_id=session_id)
    print(f"State after agent run: {updated_session.state}")
    # La salida esperada podría incluir: {'last_greeting': 'Hello there! How can I help you today?'}
    ```

=== "TypeScript"

    ```typescript
    import { LlmAgent, Runner, InMemorySessionService, isFinalResponse } from "@google/adk";
    import { Content } from "@google/genai";

    // Definir agente con outputKey
    const greetingAgent = new LlmAgent({
        name: "Greeter",
        model: "gemini-2.5-flash",
        instruction: "Generate a short, friendly greeting.",
        outputKey: "last_greeting" // Guardar respuesta en state['last_greeting']
    });

    // --- Configurar Runner y Session ---
    const appName = "state_app";
    const userId = "user1";
    const sessionId = "session1";
    const sessionService = new InMemorySessionService();
    const runner = new Runner({
        agent: greetingAgent,
        appName: appName,
        sessionService: sessionService
    });
    const session = await sessionService.createSession({
        appName,
        userId,
        sessionId
    });
    console.log(`Initial state: ${JSON.stringify(session.state)}`);

    // --- Ejecutar el Agente ---
    // Runner maneja la llamada a appendEvent, que usa el outputKey
    // para crear automáticamente el stateDelta.
    const userMessage: Content = { parts: [{ text: "Hello" }] };
    for await (const event of runner.runAsync({
        userId,
        sessionId,
        newMessage: userMessage
    })) {
        if (isFinalResponse(event)) {
          console.log("Agent responded."); // El texto de respuesta también está en event.content
        }
    }

    // --- Verificar el Estado Actualizado ---
    const updatedSession = await sessionService.getSession({ appName, userId, sessionId });
    console.log(`State after agent run: ${JSON.stringify(updatedSession?.state)}`);
    // La salida esperada podría incluir: {"last_greeting":"Hello there! How can I help you today?"}
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/sessions/state_example/state_example.go:greeting"
    ```

=== "Java"

    ```java
    --8<-- "examples/java/snippets/src/main/java/state/GreetingAgentExample.java:full_code"
    ```

Detrás de escena, el `Runner` usa el `output_key` para crear los `EventActions` necesarios con un `state_delta` y llama a `append_event`.

**2\. La Manera Estándar: `EventActions.state_delta` (para Actualizaciones Complejas)**

Para escenarios más complejos (actualizar múltiples claves, valores no string, alcances específicos como `user:` o `app:`, o actualizaciones no vinculadas directamente al texto final del agente), construyes manualmente el `state_delta` dentro de `EventActions`.

=== "Python"

    ```py
    from google.adk.sessions import InMemorySessionService, Session
    from google.adk.events import Event, EventActions
    from google.genai.types import Part, Content
    import time

    # --- Configuración ---
    session_service = InMemorySessionService()
    app_name, user_id, session_id = "state_app_manual", "user2", "session2"
    session = await session_service.create_session(
        app_name=app_name,
        user_id=user_id,
        session_id=session_id,
        state={"user:login_count": 0, "task_status": "idle"}
    )
    print(f"Initial state: {session.state}")

    # --- Definir Cambios de Estado ---
    current_time = time.time()
    state_changes = {
        "task_status": "active",              # Actualizar estado de sesión
        "user:login_count": session.state.get("user:login_count", 0) + 1, # Actualizar estado de usuario
        "user:last_login_ts": current_time,   # Agregar estado de usuario
        "temp:validation_needed": True        # Agregar estado temporal (será descartado)
    }

    # --- Crear Event con Actions ---
    actions_with_update = EventActions(state_delta=state_changes)
    # Este evento podría representar una acción interna del sistema, no solo una respuesta del agente
    system_event = Event(
        invocation_id="inv_login_update",
        author="system", # O 'agent', 'tool', etc.
        actions=actions_with_update,
        timestamp=current_time
        # content podría ser None o representar la acción tomada
    )

    # --- Agregar el Event (Esto actualiza el estado) ---
    await session_service.append_event(session, system_event)
    print("`append_event` called with explicit state delta.")

    # --- Verificar el Estado Actualizado ---
    updated_session = await session_service.get_session(app_name=app_name,
                                                user_id=user_id,
                                                session_id=session_id)
    print(f"State after event: {updated_session.state}")
    # Esperado: {'user:login_count': 1, 'task_status': 'active', 'user:last_login_ts': <timestamp>}
    # Nota: 'temp:validation_needed' NO está presente.
    ```

=== "TypeScript"

    ```typescript
    import { InMemorySessionService, createEvent, createEventActions } from "@google/adk";

    // --- Configuración ---
    const sessionService = new InMemorySessionService();
    const appName = "state_app_manual";
    const userId = "user2";
    const sessionId = "session2";
    const session = await sessionService.createSession({
        appName,
        userId,
        sessionId,
        state: { "user:login_count": 0, "task_status": "idle" }
    });
    console.log(`Initial state: ${JSON.stringify(session.state)}`);

    // --- Definir Cambios de Estado ---
    const currentTime = Date.now();
    const stateChanges = {
        "task_status": "active",              // Actualizar estado de sesión
        "user:login_count": (session.state["user:login_count"] as number || 0) + 1, // Actualizar estado de usuario
        "user:last_login_ts": currentTime,   // Agregar estado de usuario
        "temp:validation_needed": true        // Agregar estado temporal (será descartado)
    };

    // --- Crear Event con Actions ---
    const actionsWithUpdate = createEventActions({
        stateDelta: stateChanges,
    });
    // Este evento podría representar una acción interna del sistema, no solo una respuesta del agente
    const systemEvent = createEvent({
        invocationId: "inv_login_update",
        author: "system", // O 'agent', 'tool', etc.
        actions: actionsWithUpdate,
        timestamp: currentTime
        // content podría ser null o representar la acción tomada
    });

    // --- Agregar el Event (Esto actualiza el estado) ---
    await sessionService.appendEvent({ session, event: systemEvent });
    console.log("`appendEvent` called with explicit state delta.");

    // --- Verificar el Estado Actualizado ---
    const updatedSession = await sessionService.getSession({
        appName,
        userId,
        sessionId
    });
    console.log(`State after event: ${JSON.stringify(updatedSession?.state)}`);
    // Esperado: {"user:login_count":1,"task_status":"active","user:last_login_ts":<timestamp>}
    // Nota: 'temp:validation_needed' NO está presente.
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/sessions/state_example/state_example.go:manual"
    ```

=== "Java"

    ```java
    --8<-- "examples/java/snippets/src/main/java/state/ManualStateUpdateExample.java:full_code"
    ```

**3. Mediante `CallbackContext` o `ToolContext` (Recomendado para Callbacks y Tools)**

Modificar el estado dentro de callbacks de agente (ej., `on_before_agent_call`, `on_after_agent_call`) o funciones de herramientas se hace mejor usando el atributo `state` del `CallbackContext` o `ToolContext` proporcionado a tu función.

*   `callback_context.state['my_key'] = my_value`
*   `tool_context.state['my_key'] = my_value`

Estos objetos de contexto están diseñados específicamente para gestionar cambios de estado dentro de sus respectivos alcances de ejecución. Cuando modificas `context.state`, el framework del ADK asegura que estos cambios se capturen automáticamente y se enruten correctamente al `EventActions.state_delta` para el evento que está siendo generado por el callback o herramienta. Este delta es luego procesado por el `SessionService` cuando se agrega el evento, asegurando la persistencia y seguimiento adecuados.

Este método abstrae la creación manual de `EventActions` y `state_delta` para la mayoría de los escenarios comunes de actualización de estado dentro de callbacks y herramientas, haciendo tu código más limpio y menos propenso a errores.

Para más detalles completos sobre objetos de contexto, consulta la [documentación de Context](../context/index.md).

=== "Python"

    ```python
    # En un callback de agente o función de herramienta
    from google.adk.agents import CallbackContext # o ToolContext

    def my_callback_or_tool_function(context: CallbackContext, # O ToolContext
                                     # ... otros parámetros ...
                                    ):
        # Actualizar estado existente
        count = context.state.get("user_action_count", 0)
        context.state["user_action_count"] = count + 1

        # Agregar nuevo estado
        context.state["temp:last_operation_status"] = "success"

        # Los cambios de estado son automáticamente parte del state_delta del evento
        # ... resto de lógica del callback/herramienta ...
    ```

=== "TypeScript"

    ```typescript
    // En un callback de agente o función de herramienta
    import { CallbackContext } from "@google/adk"; // o ToolContext

    function myCallbackOrToolFunction(
        context: CallbackContext, // O ToolContext
        // ... otros parámetros ...
    ) {
        // Actualizar estado existente
        const count = context.state.get("user_action_count", 0);
        context.state.set("user_action_count", count + 1);

        // Agregar nuevo estado
        context.state.set("temp:last_operation_status", "success");

        // Los cambios de estado son automáticamente parte del stateDelta del evento
        // ... resto de lógica del callback/herramienta ...
    }
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/sessions/state_example/state_example.go:context"
    ```

=== "Java"

    ```java
    // En un callback de agente o método de herramienta
    import com.google.adk.agents.CallbackContext; // o ToolContext
    // ... otras importaciones ...

    public class MyAgentCallbacks {
        public void onAfterAgent(CallbackContext callbackContext) {
            // Actualizar estado existente
            Integer count = (Integer) callbackContext.state().getOrDefault("user_action_count", 0);
            callbackContext.state().put("user_action_count", count + 1);

            // Agregar nuevo estado
            callbackContext.state().put("temp:last_operation_status", "success");

            // Los cambios de estado son automáticamente parte del state_delta del evento
            // ... resto de lógica del callback ...
        }
    }
    ```

**Lo que Hace `append_event`:**

* Agrega el `Event` a `session.events`.
* Lee el `state_delta` de las `actions` del evento.
* Aplica estos cambios al estado gestionado por el `SessionService`, manejando correctamente prefijos y persistencia basándose en el tipo de servicio.
* Actualiza el `last_update_time` de la sesión.
* Asegura seguridad de hilos para actualizaciones concurrentes.

### ⚠️ Una Advertencia Sobre la Modificación Directa del Estado

Evita modificar directamente la colección `session.state` (diccionario/Map) en un objeto `Session` que fue obtenido directamente del `SessionService` (ej., mediante `session_service.get_session()` o `session_service.create_session()`) *fuera* del ciclo de vida gestionado de una invocación de agente (es decir, no a través de un `CallbackContext` o `ToolContext`). Por ejemplo, código como `retrieved_session = await session_service.get_session(...); retrieved_session.state['key'] = value` es problemático.

Las modificaciones de estado *dentro* de callbacks o herramientas usando `CallbackContext.state` o `ToolContext.state` son la forma correcta de asegurar que los cambios sean rastreados, ya que estos objetos de contexto manejan la integración necesaria con el sistema de eventos.

**Por qué la modificación directa (fuera de contextos) está fuertemente desaconsejada:**

1. **Evita el Historial de Eventos:** El cambio no se registra como un `Event`, perdiendo auditabilidad.
2. **Rompe la Persistencia:** Los cambios hechos de esta manera **probablemente NO serán guardados** por `DatabaseSessionService` o `VertexAiSessionService`. Estos dependen de `append_event` para activar el guardado.
3. **No es Seguro para Hilos:** Puede llevar a condiciones de carrera y actualizaciones perdidas.
4. **Ignora Timestamps/Lógica:** No actualiza `last_update_time` ni dispara lógica relacionada con eventos.

**Recomendación:** Mantente actualizando el estado mediante `output_key`, `EventActions.state_delta` (cuando creas eventos manualmente), o modificando la propiedad `state` de objetos `CallbackContext` o `ToolContext` cuando estés dentro de sus respectivos alcances. Estos métodos aseguran una gestión de estado confiable, rastreable y persistente. Usa el acceso directo a `session.state` (desde una sesión recuperada del `SessionService`) solo para *leer* el estado.

### Resumen de Mejores Prácticas para el Diseño del Estado

* **Minimalismo:** Almacena solo datos dinámicos esenciales.
* **Serialización:** Usa tipos básicos serializables.
* **Claves Descriptivas y Prefijos:** Usa nombres claros y prefijos apropiados (`user:`, `app:`, `temp:`, o ninguno).
* **Estructuras Planas:** Evita anidamiento profundo donde sea posible.
* **Flujo de Actualización Estándar:** Confía en `append_event`.