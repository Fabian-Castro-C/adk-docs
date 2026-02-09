# Bucle de Eventos del Runtime

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

El ADK Runtime es el motor subyacente que impulsa tu aplicación de agente durante las interacciones del usuario. Es el sistema que toma tus agentes, herramientas y callbacks definidos y orquesta su ejecución en respuesta a la entrada del usuario, gestionando el flujo de información, cambios de estado e interacciones con servicios externos como LLMs o almacenamiento.

Piensa en el Runtime como el **"motor"** de tu aplicación agéntica. Tú defines las partes (agentes, herramientas), y el Runtime maneja cómo se conectan y ejecutan juntos para cumplir con la solicitud de un usuario.

## Idea Central: El Bucle de Eventos

En su núcleo, el ADK Runtime opera sobre un **Bucle de Eventos**. Este bucle facilita una comunicación de ida y vuelta entre el componente `Runner` y tu "Lógica de Ejecución" definida (que incluye tus Agentes, las llamadas LLM que hacen, Callbacks y Herramientas).

![intro_components.png](../assets/event-loop.png)

En términos simples:

1. El `Runner` recibe una consulta del usuario y solicita al `Agent` principal que comience el procesamiento.
2. El `Agent` (y su lógica asociada) se ejecuta hasta que tiene algo que reportar (como una respuesta, una solicitud para usar una herramienta o un cambio de estado) – entonces **cede** o **emite** un `Event`.
3. El `Runner` recibe este `Event`, procesa cualquier acción asociada (como guardar cambios de estado a través de `Services`), y reenvía el evento hacia adelante (por ejemplo, a la interfaz de usuario).
4. Solo *después* de que el `Runner` ha procesado el evento, la lógica del `Agent` **se reanuda** desde donde se pausó, ahora potencialmente viendo los efectos de los cambios confirmados por el Runner.
5. Este ciclo se repite hasta que el agente no tiene más eventos que ceder para la consulta actual del usuario.

Este bucle dirigido por eventos es el patrón fundamental que gobierna cómo ADK ejecuta tu código de agente.

## El Latido: El Bucle de Eventos - Funcionamiento interno

El Bucle de Eventos es el patrón operativo central que define la interacción entre el `Runner` y tu código personalizado (Agentes, Herramientas, Callbacks, referidos colectivamente como "Lógica de Ejecución" o "Componentes de Lógica" en el documento de diseño). Establece una clara división de responsabilidades:

!!! Note
    Los nombres específicos de métodos y parámetros pueden variar ligeramente por lenguaje del SDK (por ejemplo, `agent.run_async(...)` en Python, `agent.Run(...)` en Go, `agent.runAsync(...)` en Java y TypeScript). Consulta la documentación de la API específica del lenguaje para más detalles.

### Rol del Runner (Orquestador)

El `Runner` actúa como el coordinador central para una única invocación de usuario. Sus responsabilidades en el bucle son:

1. **Iniciación:** Recibe la consulta del usuario final (`new_message`) y típicamente la añade al historial de la sesión a través del `SessionService`.
2. **Arranque:** Inicia el proceso de generación de eventos llamando al método de ejecución del agente principal (por ejemplo, `agent_to_run.run_async(...)`).
3. **Recibir y Procesar:** Espera a que la lógica del agente `yield` o `emit` un `Event`. Al recibir un evento, el Runner **lo procesa de inmediato**. Esto involucra:
      * Usar `Services` configurados (`SessionService`, `ArtifactService`, `MemoryService`) para confirmar cambios indicados en `event.actions` (como `state_delta`, `artifact_delta`).
      * Realizar otra contabilidad interna.
4. **Ceder hacia Arriba:** Reenvía el evento procesado hacia adelante (por ejemplo, a la aplicación llamadora o UI para renderizado).
5. **Iterar:** Señala a la lógica del agente que el procesamiento está completo para el evento cedido, permitiéndole reanudar y generar el *siguiente* evento.

*Bucle Conceptual del Runner:*

=== "Python"

    ```py
    # Vista simplificada de la lógica del bucle principal del Runner
    def run(new_query, ...) -> Generator[Event]:
        # 1. Añadir new_query al historial de eventos de la sesión (a través de SessionService)
        session_service.append_event(session, Event(author='user', content=new_query))

        # 2. Iniciar bucle de eventos llamando al agente
        agent_event_generator = agent_to_run.run_async(context)

        async for event in agent_event_generator:
            # 3. Procesar el evento generado y confirmar cambios
            session_service.append_event(session, event) # Confirma deltas de estado/artefactos, etc.
            # memory_service.update_memory(...) # Si aplica
            # artifact_service podría haber sido llamado ya a través del contexto durante la ejecución del agente

            # 4. Ceder evento para procesamiento upstream (por ejemplo, renderizado de UI)
            yield event
            # El Runner señala implícitamente que el generador del agente puede continuar después de ceder
    ```

=== "TypeScript"

    ```typescript
    // Vista simplificada de la lógica del bucle principal del Runner
    async * runAsync(newQuery: Content, ...): AsyncGenerator<Event, void, void> {
        // 1. Añadir newQuery al historial de eventos de la sesión (a través de SessionService)
        await sessionService.appendEvent({
            session,
            event: createEvent({author: 'user', content: newQuery})
        });

        // 2. Iniciar bucle de eventos llamando al agente
        const agentEventGenerator = agentToRun.runAsync(context);

        for await (const event of agentEventGenerator) {
            // 3. Procesar el evento generado y confirmar cambios
            // Confirma deltas de estado/artefactos, etc.
            await sessionService.appendEvent({session, event});
            // memoryService.updateMemory(...) // Si aplica
            // artifactService podría haber sido llamado ya a través del contexto durante la ejecución del agente

            // 4. Ceder evento para procesamiento upstream (por ejemplo, renderizado de UI)
            yield event;
            // El Runner señala implícitamente que el generador del agente puede continuar después de ceder
        }
    }
    ```

=== "Go"

    ```go
    // Vista conceptual simplificada de la lógica del bucle principal del Runner en Go
    func (r *Runner) RunConceptual(ctx context.Context, session *session.Session, newQuery *genai.Content) iter.Seq2[*Event, error] {
        return func(yield func(*Event, error) bool) {
            // 1. Añadir new_query al historial de eventos de la sesión (a través de SessionService)
            // ...
            userEvent := session.NewEvent(ctx.InvocationID()) // Simplificado para vista conceptual
            userEvent.Author = "user"
            userEvent.LLMResponse = model.LLMResponse{Content: newQuery}

            if _, err := r.sessionService.Append(ctx, &session.AppendRequest{Event: userEvent}); err != nil {
                yield(nil, err)
                return
            }

            // 2. Iniciar flujo de eventos llamando al agente
            // Asumiendo que agent.Run también retorna iter.Seq2[*Event, error]
            agentEventsAndErrs := r.agent.Run(ctx, &agent.RunRequest{Session: session, Input: newQuery})

            for event, err := range agentEventsAndErrs {
                if err != nil {
                    if !yield(event, err) { // Ceder evento incluso si hay error, luego detener
                        return
                    }
                    return // El agente terminó con un error
                }

                // 3. Procesar el evento generado y confirmar cambios
                // Solo confirmar evento no parcial a un servicio de sesión (como se ve en el código real)
                if !event.LLMResponse.Partial {
                    if _, err := r.sessionService.Append(ctx, &session.AppendRequest{Event: event}); err != nil {
                        yield(nil, err)
                        return
                    }
                }
                // memory_service.update_memory(...) // Si aplica
                // artifact_service podría haber sido llamado ya a través del contexto durante la ejecución del agente

                // 4. Ceder evento para procesamiento upstream
                if !yield(event, nil) {
                    return // El consumidor upstream se detuvo
                }
            }
            // El agente terminó exitosamente
        }
    }
    ```

=== "Java"

    ```java
    // Vista conceptual simplificada de la lógica del bucle principal del Runner en Java.
    public Flowable<Event> runConceptual(
        Session session,
        InvocationContext invocationContext,
        Content newQuery
        ) {

        // 1. Añadir new_query al historial de eventos de la sesión (a través de SessionService)
        // ...
        sessionService.appendEvent(session, userEvent).blockingGet();

        // 2. Iniciar flujo de eventos llamando al agente
        Flowable<Event> agentEventStream = agentToRun.runAsync(invocationContext);

        // 3. Procesar cada evento generado, confirmar cambios, y "ceder" o "emitir"
        return agentEventStream.map(event -> {
            // Esto muta el objeto session (añade evento, aplica stateDelta).
            // El valor de retorno de appendEvent (un Single<Event>) es conceptualmente
            // solo el evento mismo después del procesamiento.
            sessionService.appendEvent(session, event).blockingGet(); // Llamada bloqueante simplificada

            // memory_service.update_memory(...) // Si aplica - conceptual
            // artifact_service podría haber sido llamado ya a través del contexto durante la ejecución del agente

            // 4. "Ceder" evento para procesamiento upstream
            //    En RxJava, retornar el evento en map efectivamente lo cede al siguiente operador o suscriptor.
            return event;
        });
    }
    ```

### Rol de la Lógica de Ejecución (Agent, Tool, Callback)

Tu código dentro de agentes, herramientas y callbacks es responsable de la computación y toma de decisiones real. Su interacción con el bucle involucra:

1. **Ejecutar:** Ejecuta su lógica basada en el `InvocationContext` actual, incluyendo el estado de la sesión *tal como estaba cuando la ejecución se reanudó*.
2. **Ceder:** Cuando la lógica necesita comunicarse (enviar un mensaje, llamar a una herramienta, reportar un cambio de estado), construye un `Event` que contiene el contenido y acciones relevantes, y luego `cede` este evento de vuelta al `Runner`.
3. **Pausar:** Crucialmente, la ejecución de la lógica del agente **se pausa inmediatamente** después de la instrucción `yield` (o `return` en RxJava). Espera a que el `Runner` complete el paso 3 (procesamiento y confirmación).
4. **Reanudar:** *Solo después* de que el `Runner` ha procesado el evento cedido, la lógica del agente reanuda la ejecución desde la instrucción inmediatamente siguiente al `yield`.
5. **Ver Estado Actualizado:** Al reanudarse, la lógica del agente ahora puede acceder de manera confiable al estado de la sesión (`ctx.session.state`) reflejando los cambios que fueron confirmados por el `Runner` desde el evento *previamente cedido*.

*Lógica de Ejecución Conceptual:*

=== "Python"

    ```py
    # Vista simplificada de lógica dentro de Agent.run_async, callbacks o herramientas

    # ... código anterior se ejecuta basado en el estado actual ...

    # 1. Determinar que se necesita un cambio o salida, construir el evento
    # Ejemplo: Actualizando estado
    update_data = {'field_1': 'value_2'}
    event_with_state_change = Event(
        author=self.name,
        actions=EventActions(state_delta=update_data),
        content=types.Content(parts=[types.Part(text="Estado actualizado.")])
        # ... otros campos del evento ...
    )

    # 2. Ceder el evento al Runner para procesamiento y confirmación
    yield event_with_state_change
    # <<<<<<<<<<<< LA EJECUCIÓN SE PAUSA AQUÍ >>>>>>>>>>>>

    # <<<<<<<<<<<< EL RUNNER PROCESA Y CONFIRMA EL EVENTO >>>>>>>>>>>>

    # 3. Reanudar ejecución SOLO después de que el Runner terminó de procesar el evento anterior.
    # Ahora, el estado confirmado por el Runner está reflejado de manera confiable.
    # El código subsecuente puede asumir de manera segura que el cambio del evento cedido ocurrió.
    val = ctx.session.state['field_1']
    # aquí `val` está garantizado a ser "value_2" (asumiendo que el Runner confirmó exitosamente)
    print(f"Ejecución reanudada. El valor de field_1 es ahora: {val}")

    # ... código subsecuente continúa ...
    # Tal vez ceder otro evento más tarde...
    ```

=== "TypeScript"

    ```typescript
    // Vista simplificada de lógica dentro de Agent.runAsync, callbacks o herramientas

    // ... código anterior se ejecuta basado en el estado actual ...

    // 1. Determinar que se necesita un cambio o salida, construir el evento
    // Ejemplo: Actualizando estado
    const updateData = {'field_1': 'value_2'};
    const eventWithStateChange = createEvent({
        author: this.name,
        actions: createEventActions({stateDelta: updateData}),
        content: {parts: [{text: "Estado actualizado."}]}
        // ... otros campos del evento ...
    });

    // 2. Ceder el evento al Runner para procesamiento y confirmación
    yield eventWithStateChange;
    // <<<<<<<<<<<< LA EJECUCIÓN SE PAUSA AQUÍ >>>>>>>>>>>>

    // <<<<<<<<<<<< EL RUNNER PROCESA Y CONFIRMA EL EVENTO >>>>>>>>>>>>

    // 3. Reanudar ejecución SOLO después de que el Runner terminó de procesar el evento anterior.
    // Ahora, el estado confirmado por el Runner está reflejado de manera confiable.
    // El código subsecuente puede asumir de manera segura que el cambio del evento cedido ocurrió.
    const val = ctx.session.state['field_1'];
    // aquí `val` está garantizado a ser "value_2" (asumiendo que el Runner confirmó exitosamente)
    console.log(`Ejecución reanudada. El valor de field_1 es ahora: ${val}`);

    // ... código subsecuente continúa ...
    // Tal vez ceder otro evento más tarde...
    ```

=== "Go"

    ```go
    // Vista simplificada de lógica dentro de Agent.Run, callbacks o herramientas

    // ... código anterior se ejecuta basado en el estado actual ...

    // 1. Determinar que se necesita un cambio o salida, construir el evento
    // Ejemplo: Actualizando estado
    updateData := map[string]interface{}{"field_1": "value_2"}
    eventWithStateChange := &Event{
        Author: self.Name(),
        Actions: &EventActions{StateDelta: updateData},
        Content: genai.NewContentFromText("Estado actualizado.", "model"),
        // ... otros campos del evento ...
    }

    // 2. Ceder el evento al Runner para procesamiento y confirmación
    // En Go, esto se hace enviando el evento a un canal.
    eventsChan <- eventWithStateChange
    // <<<<<<<<<<<< LA EJECUCIÓN SE PAUSA AQUÍ (conceptualmente) >>>>>>>>>>>>
    // El Runner del otro lado del canal recibirá y procesará el evento.
    // La goroutine del agente podría continuar, pero el flujo lógico espera el siguiente input o paso.

    // <<<<<<<<<<<< EL RUNNER PROCESA Y CONFIRMA EL EVENTO >>>>>>>>>>>>

    // 3. Reanudar ejecución SOLO después de que el Runner terminó de procesar el evento anterior.
    // En una implementación real de Go, esto probablemente se manejaría por el agente recibiendo
    // un nuevo RunRequest o contexto indicando el siguiente paso. El estado actualizado
    // sería parte del objeto session en esa nueva solicitud.
    // Para este ejemplo conceptual, solo verificaremos el estado.
    val := ctx.State.Get("field_1")
    // aquí `val` está garantizado a ser "value_2" porque el Runner habría
    // actualizado el estado de la sesión antes de llamar al agente nuevamente.
    fmt.Printf("Ejecución reanudada. El valor de field_1 es ahora: %v\n", val)

    // ... código subsecuente continúa ...
    // Tal vez enviar otro evento al canal más tarde...
    ```

=== "Java"

    ```java
    // Vista simplificada de lógica dentro de Agent.runAsync, callbacks o herramientas
    // ... código anterior se ejecuta basado en el estado actual ...

    // 1. Determinar que se necesita un cambio o salida, construir el evento
    // Ejemplo: Actualizando estado
    ConcurrentMap<String, Object> updateData = new ConcurrentHashMap<>();
    updateData.put("field_1", "value_2");

    EventActions actions = EventActions.builder().stateDelta(updateData).build();
    Content eventContent = Content.builder().parts(Part.fromText("Estado actualizado.")).build();

    Event eventWithStateChange = Event.builder()
        .author(self.name())
        .actions(actions)
        .content(Optional.of(eventContent))
        // ... otros campos del evento ...
        .build();

    // 2. "Ceder" el evento. En RxJava, esto significa emitirlo al flujo.
    //    El Runner (o consumidor upstream) se suscribirá a este Flowable.
    //    Cuando el Runner reciba este evento, lo procesará (por ejemplo, llamará a sessionService.appendEvent).
    //    El 'appendEvent' en Java ADK muta el objeto 'Session' contenido en 'ctx' (InvocationContext).

    // <<<<<<<<<<<< PUNTO DE PAUSA CONCEPTUAL >>>>>>>>>>>>
    // En RxJava, la emisión de 'eventWithStateChange' ocurre, y luego el flujo
    // podría continuar con un operador 'flatMap' o 'concatMap' que representa
    // la lógica *después* de que el Runner ha procesado este evento.

    // Para modelar "reanudar ejecución SOLO después de que el Runner terminó de procesar":
    // El `appendEvent` del Runner es usualmente una operación async en sí misma (retorna Single<Event>).
    // El flujo del agente necesita estar estructurado de tal manera que la lógica subsecuente
    // que depende del estado confirmado se ejecute *después* de que ese `appendEvent` se complete.

    // Así es como el Runner típicamente lo orquesta:
    // Runner:
    //   agent.runAsync(ctx)
    //     .concatMapEager(eventFromAgent ->
    //         sessionService.appendEvent(ctx.session(), eventFromAgent) // Esto actualiza ctx.session().state()
    //             .toFlowable() // Emite el evento después de que se procesa
    //     )
    //     .subscribe(processedEvent -> { /* La UI renderiza processedEvent */ });

    // Entonces, dentro de la propia lógica del agente, si necesita hacer algo *después* de un evento que cedió
    // ha sido procesado y sus cambios de estado están reflejados en ctx.session().state(),
    // esa lógica subsecuente típicamente estaría en otro paso de su cadena reactiva.

    // Para este ejemplo conceptual, emitiremos el evento, y luego simularemos la "reanudación"
    // como una operación subsecuente en la cadena Flowable.

    return Flowable.just(eventWithStateChange) // Paso 2: Ceder el evento
        .concatMap(yieldedEvent -> {
            // <<<<<<<<<<<< EL RUNNER CONCEPTUALMENTE PROCESA Y CONFIRMA EL EVENTO >>>>>>>>>>>>
            // En este punto, en un runner real, ctx.session().appendEvent(yieldedEvent) habría sido llamado
            // por el Runner, y ctx.session().state() estaría actualizado.
            // Ya que estamos *dentro* de la lógica conceptual del agente tratando de modelar esto,
            // asumimos que la acción del Runner ha actualizado implícitamente nuestro 'ctx.session()'.

            // 3. Reanudar ejecución.
            // Ahora, el estado confirmado por el Runner (a través de sessionService.appendEvent)
            // está reflejado de manera confiable en ctx.session().state().
            Object val = ctx.session().state().get("field_1");
            // aquí `val` está garantizado a ser "value_2" porque el `sessionService.appendEvent`
            // llamado por el Runner habría actualizado el estado de la sesión dentro del objeto `ctx`.

            System.out.println("Ejecución reanudada. El valor de field_1 es ahora: " + val);

            // ... código subsecuente continúa ...
            // Si este código subsecuente necesita ceder otro evento, lo haría aquí.
    ```

Este ciclo cooperativo de ceder/pausar/reanudar entre el `Runner` y tu Lógica de Ejecución, mediado por objetos `Event`, forma el núcleo del ADK Runtime.

## Componentes clave del Runtime

Varios componentes trabajan juntos dentro del ADK Runtime para ejecutar una invocación de agente. Entender sus roles aclara cómo funciona el bucle de eventos:

1. ### `Runner`

      * **Rol:** El punto de entrada principal y orquestador para una única consulta de usuario (`run_async`).
      * **Función:** Gestiona el Bucle de Eventos general, recibe eventos cedidos por la Lógica de Ejecución, coordina con Services para procesar y confirmar acciones de eventos (cambios de estado/artefactos), y reenvía eventos procesados upstream (por ejemplo, a la UI). Esencialmente impulsa la conversación turno por turno basado en eventos cedidos. (Definido en `google.adk.runners.runner`).

2. ### Componentes de Lógica de Ejecución

      * **Rol:** Las partes que contienen tu código personalizado y las capacidades centrales del agente.
      * **Componentes:**
      * `Agent` (`BaseAgent`, `LlmAgent`, etc.): Tus unidades de lógica primarias que procesan información y deciden acciones. Implementan el método `_run_async_impl` que cede eventos.
      * `Tools` (`BaseTool`, `FunctionTool`, `AgentTool`, etc.): Funciones o capacidades externas usadas por agentes (a menudo `LlmAgent`) para interactuar con el mundo exterior o realizar tareas específicas. Se ejecutan y retornan resultados, que luego se envuelven en eventos.
      * `Callbacks` (Funciones): Funciones definidas por el usuario adjuntas a agentes (por ejemplo, `before_agent_callback`, `after_model_callback`) que se conectan a puntos específicos en el flujo de ejecución, potencialmente modificando comportamiento o estado, cuyos efectos son capturados en eventos.
      * **Función:** Realizan el pensamiento, cálculo o interacción externa real. Comunican sus resultados o necesidades **cediendo objetos `Event`** y pausando hasta que el Runner los procese.

3. ### `Event`

      * **Rol:** El mensaje pasado de ida y vuelta entre el `Runner` y la Lógica de Ejecución.
      * **Función:** Representa una ocurrencia atómica (input del usuario, texto del agente, llamada/resultado de herramienta, solicitud de cambio de estado, señal de control). Lleva tanto el contenido de la ocurrencia como los efectos secundarios pretendidos (`actions` como `state_delta`).

4. ### `Services`

      * **Rol:** Componentes backend responsables de gestionar recursos persistentes o compartidos. Usados principalmente por el `Runner` durante el procesamiento de eventos.
      * **Componentes:**
      * `SessionService` (`BaseSessionService`, `InMemorySessionService`, etc.): Gestiona objetos `Session`, incluyendo guardar/cargarlos, aplicar `state_delta` al estado de la sesión, y añadir eventos al `historial de eventos`.
      * `ArtifactService` (`BaseArtifactService`, `InMemoryArtifactService`, `GcsArtifactService`, etc.): Gestiona el almacenamiento y recuperación de datos de artefactos binarios. Aunque `save_artifact` se llama a través del contexto durante la lógica de ejecución, el `artifact_delta` en el evento confirma la acción para el Runner/SessionService.
      * `MemoryService` (`BaseMemoryService`, etc.): (Opcional) Gestiona memoria semántica a largo plazo a través de sesiones para un usuario.
      * **Función:** Proveen la capa de persistencia. El `Runner` interactúa con ellos para asegurar que los cambios señalados por `event.actions` estén almacenados de manera confiable *antes* de que la Lógica de Ejecución se reanude.

5. ### `Session`

      * **Rol:** Un contenedor de datos que contiene el estado e historial para *una conversación específica* entre un usuario y la aplicación.
      * **Función:** Almacena el diccionario `state` actual, la lista de todos los `events` pasados (`historial de eventos`), y referencias a artefactos asociados. Es el registro primario de la interacción, gestionado por el `SessionService`.

6. ### `Invocation`

      * **Rol:** Un término conceptual que representa todo lo que sucede en respuesta a una *única* consulta de usuario, desde el momento en que el `Runner` la recibe hasta que la lógica del agente termina de ceder eventos para esa consulta.
      * **Función:** Una invocación podría involucrar múltiples ejecuciones de agentes (si se usa transferencia de agente o `AgentTool`), múltiples llamadas LLM, ejecuciones de herramientas y ejecuciones de callbacks, todos unidos por un único `invocation_id` dentro del `InvocationContext`. Las variables de estado con prefijo `temp:` están estrictamente limitadas a una única invocación y se descartan después.

Estos actores interactúan continuamente a través del Bucle de Eventos para procesar la solicitud de un usuario.

## Cómo Funciona: Una Invocación Simplificada

Tracemos un flujo simplificado para una consulta típica de usuario que involucra un agente LLM llamando a una herramienta:

![intro_components.png](../assets/invocation-flow.png)

### Desglose Paso a Paso

1. **Input del Usuario:** El Usuario envía una consulta (por ejemplo, "¿Cuál es la capital de Francia?").
2. **Runner Inicia:** `Runner.run_async` comienza. Interactúa con el `SessionService` para cargar la `Session` relevante y añade la consulta del usuario como el primer `Event` al historial de la sesión. Se prepara un `InvocationContext` (`ctx`).
3. **Ejecución del Agente:** El `Runner` llama a `agent.run_async(ctx)` en el agente raíz designado (por ejemplo, un `LlmAgent`).
4. **Llamada LLM (Ejemplo):** El `Agent_Llm` determina que necesita información, tal vez llamando a una herramienta. Prepara una solicitud para el `LLM`. Asumamos que el LLM decide llamar a `MyTool`.
5. **Ceder Evento FunctionCall:** El `Agent_Llm` recibe la respuesta `FunctionCall` del LLM, la envuelve en un `Event(author='Agent_Llm', content=Content(parts=[Part(function_call=...)]))`, y `cede` o `emite` este evento.
6. **Agente se Pausa:** La ejecución del `Agent_Llm` se pausa inmediatamente después del `yield`.
7. **Runner Procesa:** El `Runner` recibe el evento FunctionCall. Lo pasa al `SessionService` para registrarlo en el historial. El `Runner` luego cede el evento upstream al `User` (o aplicación).
8. **Agente se Reanuda:** El `Runner` señala que el evento está procesado, y `Agent_Llm` reanuda la ejecución.
9. **Ejecución de Herramienta:** El flujo interno del `Agent_Llm` ahora procede a ejecutar la `MyTool` solicitada. Llama a `tool.run_async(...)`.
10. **Herramienta Retorna Resultado:** `MyTool` se ejecuta y retorna su resultado (por ejemplo, `{'result': 'Paris'}`).
11. **Ceder Evento FunctionResponse:** El agente (`Agent_Llm`) envuelve el resultado de la herramienta en un `Event` conteniendo una parte `FunctionResponse` (por ejemplo, `Event(author='Agent_Llm', content=Content(role='user', parts=[Part(function_response=...)]))`). Este evento también podría contener `actions` si la herramienta modificó el estado (`state_delta`) o guardó artefactos (`artifact_delta`). El agente `cede` este evento.
12. **Agente se Pausa:** `Agent_Llm` se pausa nuevamente.
13. **Runner Procesa:** `Runner` recibe el evento FunctionResponse. Lo pasa a `SessionService` que aplica cualquier `state_delta`/`artifact_delta` y añade el evento al historial. `Runner` cede el evento upstream.
14. **Agente se Reanuda:** `Agent_Llm` se reanuda, ahora sabiendo que el resultado de la herramienta y cualquier cambio de estado están confirmados.
15. **Llamada LLM Final (Ejemplo):** `Agent_Llm` envía el resultado de la herramienta de vuelta al `LLM` para generar una respuesta en lenguaje natural.
16. **Ceder Evento de Texto Final:** `Agent_Llm` recibe el texto final del `LLM`, lo envuelve en un `Event(author='Agent_Llm', content=Content(parts=[Part(text=...)]))`, y lo `cede`.
17. **Agente se Pausa:** `Agent_Llm` se pausa.
18. **Runner Procesa:** `Runner` recibe el evento de texto final, lo pasa a `SessionService` para el historial, y lo cede upstream al `User`. Esto probablemente está marcado como `is_final_response()`.
19. **Agente se Reanuda y Termina:** `Agent_Llm` se reanuda. Habiendo completado su tarea para esta invocación, su generador `run_async` termina.
20. **Runner Completa:** El `Runner` ve que el generador del agente está agotado y termina su bucle para esta invocación.

Este ciclo de ceder/pausar/procesar/reanudar asegura que los cambios de estado se apliquen consistentemente y que la lógica de ejecución siempre opere sobre el estado confirmado más recientemente después de ceder un evento.

## Comportamientos Importantes del Runtime

Entender algunos aspectos clave de cómo el ADK Runtime maneja estado, streaming y operaciones asíncronas es crucial para construir agentes predecibles y eficientes.

### Actualizaciones de Estado y Momento de Confirmación

* **La Regla:** Cuando tu código (en un agente, herramienta o callback) modifica el estado de la sesión (por ejemplo, `context.state['my_key'] = 'new_value'`), este cambio se registra inicialmente de manera local dentro del `InvocationContext` actual. El cambio solo está **garantizado a ser persistido** (guardado por el `SessionService`) *después* de que el `Event` que lleva el `state_delta` correspondiente en sus `actions` ha sido `cedido` por tu código y subsecuentemente procesado por el `Runner`.

* **Implicación:** El código que se ejecuta *después* de reanudar de un `yield` puede asumir de manera confiable que los cambios de estado señalados en el *evento cedido* han sido confirmados.

=== "Python"

    ```py
    # Dentro de lógica del agente (conceptual)

    # 1. Modificar estado
    ctx.session.state['status'] = 'processing'
    event1 = Event(..., actions=EventActions(state_delta={'status': 'processing'}))

    # 2. Ceder evento con el delta
    yield event1
    # --- PAUSA --- Runner procesa event1, SessionService confirma 'status' = 'processing' ---

    # 3. Reanudar ejecución
    # Ahora es seguro confiar en el estado confirmado
    current_status = ctx.session.state['status'] # Garantizado a ser 'processing'
    print(f"Estado después de reanudar: {current_status}")
    ```

=== "TypeScript"

    ```typescript
    // Dentro de lógica del agente (conceptual)

    // 1. Modificar estado
    // En TypeScript, modificas estado a través del contexto, que rastrea el cambio.
    ctx.state.set('status', 'processing');
    // El framework automáticamente poblará actions con el delta de estado
    // desde el contexto. Para ilustración, se muestra aquí.
    const event1 = createEvent({
        actions: createEventActions({stateDelta: {'status': 'processing'}}),
        // ... otros campos del evento
    });

    // 2. Ceder evento con el delta
    yield event1;
    // --- PAUSA --- Runner procesa event1, SessionService confirma 'status' = 'processing' ---

    // 3. Reanudar ejecución
    // Ahora es seguro confiar en el estado confirmado en el objeto session.
    const currentStatus = ctx.session.state['status']; // Garantizado a ser 'processing'
    console.log(`Estado después de reanudar: ${currentStatus}`);
    ```

=== "Go"

    ```go
      // Dentro de lógica del agente (conceptual)

    func (a *Agent) RunConceptual(ctx agent.InvocationContext) iter.Seq2[*session.Event, error] {
      // Toda la lógica está envuelta en una función que será retornada como un iterador.
      return func(yield func(*session.Event, error) bool) {
          // ... código anterior se ejecuta basado en el estado actual del input `ctx` ...
          // por ejemplo, val := ctx.State().Get("field_1") podría retornar "value_1" aquí.

          // 1. Determinar que se necesita un cambio o salida, construir el evento
          updateData := map[string]interface{}{"field_1": "value_2"}
          eventWithStateChange := session.NewEvent(ctx.InvocationID())
          eventWithStateChange.Author = a.Name()
          eventWithStateChange.Actions = &session.EventActions{StateDelta: updateData}
          // ... otros campos del evento ...


          // 2. Ceder el evento al Runner para procesamiento y confirmación.
          // La ejecución del agente continúa inmediatamente después de esta llamada.
          if !yield(eventWithStateChange, nil) {
              // Si yield retorna false, significa que el consumidor (el Runner)
              // ha dejado de escuchar, así que deberíamos dejar de producir eventos.
              return
          }

          // <<<<<<<<<<<< EL RUNNER PROCESA Y CONFIRMA EL EVENTO >>>>>>>>>>>>
          // Esto sucede fuera del agente, después de que el iterador del agente ha
          // producido el evento.

          // 3. El agente NO PUEDE ver inmediatamente el cambio de estado que acaba de ceder.
          // El estado es inmutable dentro de una única invocación `Run`.
          val := ctx.State().Get("field_1")
          // `val` aquí es TODAVÍA "value_1" (o lo que fuera al inicio).
          // El estado actualizado ("value_2") solo estará disponible en el `ctx`
          // de la *siguiente* invocación `Run` en un turno subsecuente.

          // ... código subsecuente continúa, potencialmente cediendo más eventos ...
          finalEvent := session.NewEvent(ctx.InvocationID())
          finalEvent.Author = a.Name()
          // ...
          yield(finalEvent, nil)
      }
    }
    ```

=== "Java"

    ```java
    // Dentro de lógica del agente (conceptual)
    // ... código anterior se ejecuta basado en el estado actual ...

    // 1. Preparar modificación de estado y construir el evento
    ConcurrentHashMap<String, Object> stateChanges = new ConcurrentHashMap<>();
    stateChanges.put("status", "processing");

    EventActions actions = EventActions.builder().stateDelta(stateChanges).build();
    Content content = Content.builder().parts(Part.fromText("Actualización de estado: processing")).build();

    Event event1 = Event.builder()
        .actions(actions)
        // ...
        .build();

    // 2. Ceder evento con el delta
    return Flowable.just(event1)
        .map(
            emittedEvent -> {
                // --- PAUSA CONCEPTUAL Y PROCESAMIENTO DEL RUNNER ---
                // 3. Reanudar ejecución (conceptualmente)
                // Ahora es seguro confiar en el estado confirmado.
                String currentStatus = (String) ctx.session().state().get("status");
                System.out.println("Estado después de reanudar (dentro de lógica del agente): " + currentStatus); // Garantizado a ser 'processing'

                // El evento mismo (event1) se pasa adelante.
                // Si la lógica subsecuente dentro de este paso del agente produjo *otro* evento,
                // usarías concatMap para emitir ese nuevo evento.
                return emittedEvent;
            });

    // ... lógica subsecuente del agente podría involucrar más operadores reactivos
    // o emitir más eventos basados en el ahora-actualizado `ctx.session().state()`.
    ```

### "Lecturas Sucias" del Estado de la Sesión

* **Definición:** Aunque la confirmación sucede *después* del yield, el código ejecutándose *más tarde dentro de la misma invocación*, pero *antes* de que el evento que cambia el estado sea realmente cedido y procesado, **a menudo puede ver los cambios locales no confirmados**. Esto a veces se llama una "lectura sucia".
* **Ejemplo:**

=== "Python"

    ```py
    # Código en before_agent_callback
    callback_context.state['field_1'] = 'value_1'
    # El estado está configurado localmente a 'value_1', pero aún no confirmado por el Runner

    # ... el agente se ejecuta ...

    # Código en una herramienta llamada más tarde *dentro de la misma invocación*
    # Legible (lectura sucia), pero 'value_1' aún no está garantizado persistente.
    val = tool_context.state['field_1'] # 'val' probablemente será 'value_1' aquí
    print(f"Valor de lectura sucia en herramienta: {val}")

    # Asume que el evento llevando state_delta={'field_1': 'value_1'}
    # es cedido *después* de que esta herramienta se ejecuta y es procesado por el Runner.
    ```

=== "TypeScript"

    ```typescript
    // Código en beforeAgentCallback
    callbackContext.state.set('field_1', 'value_1');
    // El estado está configurado localmente a 'value_1', pero aún no confirmado por el Runner

    // --- el agente se ejecuta ... ---

    // --- Código en una herramienta llamada más tarde *dentro de la misma invocación* ---
    // Legible (lectura sucia), pero 'value_1' aún no está garantizado persistente.
    const val = toolContext.state.get('field_1'); // 'val' probablemente será 'value_1' aquí
    console.log(`Valor de lectura sucia en herramienta: ${val}`);

    // Asume que el evento llevando state_delta={'field_1': 'value_1'}
    // es cedido *después* de que esta herramienta se ejecuta y es procesado por el Runner.
    ```

=== "Go"

    ```go
    // Código en before_agent_callback
    // El callback modificaría el estado de la sesión del contexto directamente.
    // Este cambio es local al contexto de invocación actual.
    ctx.State.Set("field_1", "value_1")
    // El estado está configurado localmente a 'value_1', pero aún no confirmado por el Runner

    // ... el agente se ejecuta ...

    // Código en una herramienta llamada más tarde *dentro de la misma invocación*
    // Legible (lectura sucia), pero 'value_1' aún no está garantizado persistente.
    val := ctx.State.Get("field_1") // 'val' probablemente será 'value_1' aquí
    fmt.Printf("Valor de lectura sucia en herramienta: %v\n", val)

    // Asume que el evento llevando state_delta={'field_1': 'value_1'}
    // es cedido *después* de que esta herramienta se ejecuta y es procesado por el Runner.
    ```

=== "Java"

    ```java
    // Modificar estado - Código en BeforeAgentCallback
    // Y prepara este cambio en callbackContext.eventActions().stateDelta().
    callbackContext.state().put("field_1", "value_1");

    // --- el agente se ejecuta ... ---

    // --- Código en una herramienta llamada más tarde *dentro de la misma invocación* ---
    // Legible (lectura sucia), pero 'value_1' aún no está garantizado persistente.
    Object val = toolContext.state().get("field_1"); // 'val' probablemente será 'value_1' aquí
    System.out.println("Valor de lectura sucia en herramienta: " + val);
    // Asume que el evento llevando state_delta={'field_1': 'value_1'}
    // es cedido *después* de que esta herramienta se ejecuta y es procesado por el Runner.
    ```

* **Implicaciones:**
  * **Beneficio:** Permite que diferentes partes de tu lógica dentro de un único paso complejo (por ejemplo, múltiples callbacks o llamadas de herramientas antes del siguiente turno LLM) coordinen usando estado sin esperar un ciclo completo de yield/confirmación.
  * **Advertencia:** Depender fuertemente de lecturas sucias para lógica crítica puede ser arriesgado. Si la invocación falla *antes* de que el evento llevando el `state_delta` sea cedido y procesado por el `Runner`, el cambio de estado no confirmado se perderá. Para transiciones de estado críticas, asegura que estén asociadas con un evento que sea procesado exitosamente.

### Salida Streaming vs. No-Streaming (`partial=True`)

Esto se relaciona principalmente con cómo se manejan las respuestas del LLM, especialmente cuando se usan APIs de generación streaming.

* **Streaming:** El LLM genera su respuesta token por token o en pequeños fragmentos.
  * El framework (a menudo dentro de `BaseLlmFlow`) cede múltiples objetos `Event` para una única respuesta conceptual. La mayoría de estos eventos tendrán `partial=True`.
  * El `Runner`, al recibir un evento con `partial=True`, típicamente **lo reenvía inmediatamente** upstream (para visualización en UI) pero **omite procesar sus `actions`** (como `state_delta`).
  * Eventualmente, el framework cede un evento final para esa respuesta, marcado como no parcial (`partial=False` o implícitamente a través de `turn_complete=True`).
  * El `Runner` **procesa completamente solo este evento final**, confirmando cualquier `state_delta` o `artifact_delta` asociado.
* **No-Streaming:** El LLM genera la respuesta completa de una vez. El framework cede un único evento marcado como no parcial, que el `Runner` procesa completamente.
* **Por qué Importa:** Asegura que los cambios de estado se apliquen atómicamente y solo una vez basado en la respuesta *completa* del LLM, mientras aún permite que la UI muestre texto progresivamente a medida que se genera.

## Async es Primario (`run_async`)

* **Diseño Central:** El ADK Runtime está fundamentalmente construido sobre patrones y bibliotecas asíncronos (como `asyncio` de Python, `RxJava` de Java, y `Promise`s y `AsyncGenerator`s nativos en TypeScript) para manejar operaciones concurrentes (como esperar respuestas LLM o ejecuciones de herramientas) eficientemente sin bloquear.
* **Punto de Entrada Principal:** `Runner.run_async` es el método primario para ejecutar invocaciones de agente. Todos los componentes ejecutables centrales (Agentes, flujos específicos) usan métodos `asíncronos` internamente.
* **Conveniencia Síncrona (`run`):** Existe un método síncrono `Runner.run` principalmente por conveniencia (por ejemplo, en scripts simples o entornos de prueba). Sin embargo, internamente, `Runner.run` típicamente solo llama a `Runner.run_async` y gestiona la ejecución del bucle de eventos async por ti.
* **Experiencia del Desarrollador:** Recomendamos diseñar tus aplicaciones (por ejemplo, servidores web usando ADK) para ser asíncronas para mejor rendimiento. En Python, esto significa usar `asyncio`; en Java, aprovechar el modelo de programación reactiva de `RxJava`; y en TypeScript, esto significa construir usando `Promise`s y `AsyncGenerator`s nativos.
* **Callbacks/Tools Síncronos:** El framework ADK soporta tanto funciones asíncronas como síncronas para herramientas y callbacks.
    * **I/O Bloqueante:** Para operaciones síncronas de I/O de larga duración, el framework intenta prevenir estancamientos. Python ADK puede usar asyncio.to_thread, mientras que Java ADK a menudo depende de planificadores RxJava apropiados o wrappers para llamadas bloqueantes. En TypeScript, el framework simplemente espera la función; si una función síncrona realiza I/O bloqueante, estancará el bucle de eventos. Los desarrolladores deberían usar APIs de I/O asíncronas (que retornan una Promise) siempre que sea posible.
    * **Trabajo CPU-intensivo:** Las tareas síncronas puramente intensivas en CPU aún bloquearán su hilo de ejecución en ambos entornos.

Entender estos comportamientos te ayuda a escribir aplicaciones ADK más robustas y depurar problemas relacionados con consistencia de estado, actualizaciones streaming y ejecución asíncrona.