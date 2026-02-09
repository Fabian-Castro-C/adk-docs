# Eventos

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Los eventos son las unidades fundamentales del flujo de información dentro del Kit de Desarrollo de Agentes (ADK). Representan cada ocurrencia significativa durante el ciclo de vida de interacción de un agente, desde la entrada inicial del usuario hasta la respuesta final y todos los pasos intermedios. Comprender los eventos es crucial porque son la forma principal en que los componentes se comunican, se gestiona el estado y se dirige el flujo de control.

## Qué Son los Eventos y Por Qué Importan

Un `Event` en ADK es un registro inmutable que representa un punto específico en la ejecución del agente. Captura mensajes de usuario, respuestas del agente, solicitudes para usar herramientas (llamadas a funciones), resultados de herramientas, cambios de estado, señales de control y errores.

=== "Python"
    Técnicamente, es una instancia de la clase `google.adk.events.Event`, que se basa en la estructura básica `LlmResponse` al agregar metadatos esenciales específicos de ADK y una carga útil de `actions`.

    ```python
    # Estructura Conceptual de un Evento (Python)
    # from google.adk.events import Event, EventActions
    # from google.genai import types

    # class Event(LlmResponse): # Vista simplificada
    #     # --- campos de LlmResponse ---
    #     content: Optional[types.Content]
    #     partial: Optional[bool]
    #     # ... otros campos de respuesta ...

    #     # --- adiciones específicas de ADK ---
    #     author: str          # 'user' o nombre del agente
    #     invocation_id: str   # ID para toda la ejecución de la interacción
    #     id: str              # ID único para este evento específico
    #     timestamp: float     # Hora de creación
    #     actions: EventActions # Importante para efectos secundarios y control
    #     branch: Optional[str] # Ruta de jerarquía
    #     # ...
    ```

=== "Go"
    En Go, esto es una estructura de tipo `google.golang.org/adk/session.Event`.

    ```go
    // Estructura Conceptual de un Evento (Go - Ver session/session.go)
    // Vista simplificada basada en la estructura session.Event
    type Event struct {
        // --- Campos del model.LLMResponse embebido ---
        model.LLMResponse

        // --- adiciones específicas de ADK ---
        Author       string         // 'user' o nombre del agente
        InvocationID string         // ID para toda la ejecución de la interacción
        ID           string         // ID único para este evento específico
        Timestamp    time.Time      // Hora de creación
        Actions      EventActions   // Importante para efectos secundarios y control
        Branch       string         // Ruta de jerarquía
        // ... otros campos
    }

    // model.LLMResponse contiene el campo Content
    type LLMResponse struct {
        Content *genai.Content
        // ... otros campos
    }
    ```

=== "Java"
    En Java, esto es una instancia de la clase `com.google.adk.events.Event`. También se basa en una estructura de respuesta básica al agregar metadatos esenciales específicos de ADK y una carga útil de `actions`.

    ```java
    // Estructura Conceptual de un Evento (Java - Ver com.google.adk.events.Event.java)
    // Vista simplificada basada en el com.google.adk.events.Event.java proporcionado
    // public class Event extends JsonBaseModel {
    //     // --- Campos análogos a LlmResponse ---
    //     private Optional<Content> content;
    //     private Optional<Boolean> partial;
    //     // ... otros campos de respuesta como errorCode, errorMessage ...

    //     // --- adiciones específicas de ADK ---
    //     private String author;         // 'user' o nombre del agente
    //     private String invocationId;   // ID para toda la ejecución de la interacción
    //     private String id;             // ID único para este evento específico
    //     private long timestamp;        // Hora de creación (milisegundos epoch)
    //     private EventActions actions;  // Importante para efectos secundarios y control
    //     private Optional<String> branch; // Ruta de jerarquía
    //     // ... otros campos como turnComplete, longRunningToolIds etc.
    // }
    ```

Los eventos son centrales para la operación de ADK por varias razones clave:

1.  **Comunicación:** Sirven como el formato de mensaje estándar entre la interfaz de usuario, el `Runner`, los agentes, el LLM y las herramientas. Todo fluye como un `Event`.

2.  **Señalización de Cambios de Estado y Artefactos:** Los eventos llevan instrucciones para modificaciones de estado y rastrean actualizaciones de artefactos. El `SessionService` usa estas señales para garantizar la persistencia. En Python, los cambios se señalan mediante `event.actions.state_delta` y `event.actions.artifact_delta`.

3.  **Flujo de Control:** Campos específicos como `event.actions.transfer_to_agent` o `event.actions.escalate` actúan como señales que dirigen el framework, determinando qué agente se ejecuta a continuación o si un bucle debe terminar.

4.  **Historial y Observabilidad:** La secuencia de eventos registrados en `session.events` proporciona un historial completo y cronológico de una interacción, invaluable para depuración, auditoría y comprensión del comportamiento del agente paso a paso.

En esencia, todo el proceso, desde la consulta de un usuario hasta la respuesta final del agente, se orquesta a través de la generación, interpretación y procesamiento de objetos `Event`.


## Comprender y Usar Eventos

Como desarrollador, interactuarás principalmente con el flujo de eventos generado por el `Runner`. Así es como comprender y extraer información de ellos:

!!! Note
    Los parámetros específicos o nombres de métodos para las primitivas pueden variar ligeramente según el lenguaje del SDK (por ejemplo, `event.content()` en Python, `event.content().get().parts()` en Java). Consulta la documentación de la API específica del lenguaje para más detalles.

### Identificar el Origen y Tipo de Evento

Determina rápidamente lo que representa un evento verificando:

*   **¿Quién lo envió? (`event.author`)**
    *   `'user'`: Indica entrada directamente del usuario final.
    *   `'NombreAgente'`: Indica salida o acción de un agente específico (por ejemplo, `'WeatherAgent'`, `'SummarizerAgent'`).
*   **¿Cuál es la carga útil principal? (`event.content` y `event.content.parts`)**
    *   **Texto:** Indica un mensaje conversacional. Para Python, verifica si existe `event.content.parts[0].text`. Para Java, verifica si `event.content()` está presente, sus `parts()` están presentes y no están vacías, y el `text()` de la primera parte está presente.
    *   **Solicitud de Llamada a Herramienta:** Verifica `event.get_function_calls()`. Si no está vacío, el LLM está pidiendo ejecutar una o más herramientas. Cada elemento en la lista tiene `.name` y `.args`.
    *   **Resultado de Herramienta:** Verifica `event.get_function_responses()`. Si no está vacío, este evento lleva el resultado(s) de la ejecución de herramienta(s). Cada elemento tiene `.name` y `.response` (el diccionario devuelto por la herramienta). *Nota:* Para la estructuración del historial, el `role` dentro del `content` es a menudo `'user'`, pero el `author` del evento es típicamente el agente que solicitó la llamada a la herramienta.

*   **¿Es salida en streaming? (`event.partial`)**
    Indica si este es un fragmento incompleto de texto del LLM.
    *   `True`: Seguirá más texto.
    *   `False` o `None`/`Optional.empty()`: Esta parte del contenido está completa (aunque el turno completo podría no haber terminado si `turn_complete` también es falso).

=== "Python"

    ```python
    # Pseudocódigo: Identificación básica de eventos (Python)
    # async for event in runner.run_async(...):
    #     print(f"Evento de: {event.author}")
    #
    #     if event.content and event.content.parts:
    #         if event.get_function_calls():
    #             print("  Tipo: Solicitud de Llamada a Herramienta")
    #         elif event.get_function_responses():
    #             print("  Tipo: Resultado de Herramienta")
    #         elif event.content.parts[0].text:
    #             if event.partial:
    #                 print("  Tipo: Fragmento de Texto en Streaming")
    #             else:
    #                 print("  Tipo: Mensaje de Texto Completo")
    #         else:
    #             print("  Tipo: Otro Contenido (ej. resultado de código)")
    #     elif event.actions and (event.actions.state_delta or event.actions.artifact_delta):
    #         print("  Tipo: Actualización de Estado/Artefacto")
    #     else:
    #         print("  Tipo: Señal de Control u Otro")
    ```

=== "Go"

    ```go
      // Pseudocódigo: Identificación básica de eventos (Go)
    import (
      "fmt"
      "google.golang.org/adk/session"
      "google.golang.org/genai"
    )

    func hasFunctionCalls(content *genai.Content) bool {
      if content == nil {
        return false
      }
      for _, part := range content.Parts {
        if part.FunctionCall != nil {
          return true
        }
      }
      return false
    }

    func hasFunctionResponses(content *genai.Content) bool {
      if content == nil {
        return false
      }
      for _, part := range content.Parts {
        if part.FunctionResponse != nil {
          return true
        }
      }
      return false
    }

    func processEvents(events <-chan *session.Event) {
      for event := range events {
        fmt.Printf("Evento de: %s\n", event.Author)

        if event.LLMResponse != nil && event.LLMResponse.Content != nil {
          if hasFunctionCalls(event.LLMResponse.Content) {
            fmt.Println("  Tipo: Solicitud de Llamada a Herramienta")
          } else if hasFunctionResponses(event.LLMResponse.Content) {
            fmt.Println("  Tipo: Resultado de Herramienta")
          } else if len(event.LLMResponse.Content.Parts) > 0 {
            if event.LLMResponse.Content.Parts[0].Text != "" {
              if event.LLMResponse.Partial {
                fmt.Println("  Tipo: Fragmento de Texto en Streaming")
              } else {
                fmt.Println("  Tipo: Mensaje de Texto Completo")
              }
            } else {
              fmt.Println("  Tipo: Otro Contenido (ej. resultado de código)")
            }
          }
        } else if len(event.Actions.StateDelta) > 0 {
          fmt.Println("  Tipo: Actualización de Estado")
        } else {
          fmt.Println("  Tipo: Señal de Control u Otro")
        }
      }
    }

    ```

=== "Java"

    ```java
    // Pseudocódigo: Identificación básica de eventos (Java)
    // import com.google.genai.types.Content;
    // import com.google.adk.events.Event;
    // import com.google.adk.events.EventActions;

    // runner.runAsync(...).forEach(event -> { // Asumiendo un flujo síncrono o flujo reactivo
    //     System.out.println("Evento de: " + event.author());
    //
    //     if (event.content().isPresent()) {
    //         Content content = event.content().get();
    //         if (!event.functionCalls().isEmpty()) {
    //             System.out.println("  Tipo: Solicitud de Llamada a Herramienta");
    //         } else if (!event.functionResponses().isEmpty()) {
    //             System.out.println("  Tipo: Resultado de Herramienta");
    //         } else if (content.parts().isPresent() && !content.parts().get().isEmpty() &&
    //                    content.parts().get().get(0).text().isPresent()) {
    //             if (event.partial().orElse(false)) {
    //                 System.out.println("  Tipo: Fragmento de Texto en Streaming");
    //             } else {
    //                 System.out.println("  Tipo: Mensaje de Texto Completo");
    //             }
    //         } else {
    //             System.out.println("  Tipo: Otro Contenido (ej. resultado de código)");
    //         }
    //     } else if (event.actions() != null &&
    //                ((event.actions().stateDelta() != null && !event.actions().stateDelta().isEmpty()) ||
    //                 (event.actions().artifactDelta() != null && !event.actions().artifactDelta().isEmpty()))) {
    //         System.out.println("  Tipo: Actualización de Estado/Artefacto");
    //     } else {
    //         System.out.println("  Tipo: Señal de Control u Otro");
    //     }
    // });
    ```

### Extraer Información Clave

Una vez que conozcas el tipo de evento, accede a los datos relevantes:

*   **Contenido de Texto:**
    Siempre verifica la presencia de contenido y partes antes de acceder al texto. En Python es `text = event.content.parts[0].text`.

*   **Detalles de Llamada a Función:**

    === "Python"

        ```python
        calls = event.get_function_calls()
        if calls:
            for call in calls:
                tool_name = call.name
                arguments = call.args # Esto suele ser un diccionario
                print(f"  Herramienta: {tool_name}, Args: {arguments}")
                # La aplicación podría despachar la ejecución basándose en esto
        ```

    === "Go"

        ```go
        import (
            "fmt"
            "google.golang.org/adk/session"
            "google.golang.org/genai"
        )

        func handleFunctionCalls(event *session.Event) {
            if event.LLMResponse == nil || event.LLMResponse.Content == nil {
                return
            }
            calls := event.Content.FunctionCalls()
            if len(calls) > 0 {
                for _, call := range calls {
                    toolName := call.Name
                    arguments := call.Args
                    fmt.Printf("  Herramienta: %s, Args: %v\n", toolName, arguments)
                    // La aplicación podría despachar la ejecución basándose en esto
                }
            }
        }
        ```

    === "Java"

        ```java
        import com.google.genai.types.FunctionCall;
        import com.google.common.collect.ImmutableList;
        import java.util.Map;

        ImmutableList<FunctionCall> calls = event.functionCalls(); // desde Event.java
        if (!calls.isEmpty()) {
          for (FunctionCall call : calls) {
            String toolName = call.name().get();
            // args es Optional<Map<String, Object>>
            Map<String, Object> arguments = call.args().get();
                   System.out.println("  Herramienta: " + toolName + ", Args: " + arguments);
            // La aplicación podría despachar la ejecución basándose en esto
          }
        }
        ```

*   **Detalles de Respuesta de Función:**

    === "Python"

        ```python
        responses = event.get_function_responses()
        if responses:
            for response in responses:
                tool_name = response.name
                result_dict = response.response # El diccionario devuelto por la herramienta
                print(f"  Resultado de Herramienta: {tool_name} -> {result_dict}")
        ```

    === "Go"

        ```go
        import (
            "fmt"
            "google.golang.org/adk/session"
            "google.golang.org/genai"
        )

        func handleFunctionResponses(event *session.Event) {
            if event.LLMResponse == nil || event.LLMResponse.Content == nil {
                return
            }
            responses := event.Content.FunctionResponses()
            if len(responses) > 0 {
                for _, response := range responses {
                    toolName := response.Name
                    result := response.Response
                    fmt.Printf("  Resultado de Herramienta: %s -> %v\n", toolName, result)
                }
            }
        }
        ```

    === "Java"

        ```java
        import com.google.genai.types.FunctionResponse;
        import com.google.common.collect.ImmutableList;
        import java.util.Map;

        ImmutableList<FunctionResponse> responses = event.functionResponses(); // desde Event.java
        if (!responses.isEmpty()) {
            for (FunctionResponse response : responses) {
                String toolName = response.name().get();
                Map<String, String> result= response.response().get(); // Verificar antes de obtener la respuesta
                System.out.println("  Resultado de Herramienta: " + toolName + " -> " + result);
            }
        }
        ```

*   **Identificadores:**
    *   `event.id`: ID único para esta instancia específica de evento.
    *   `event.invocation_id`: ID para todo el ciclo de solicitud-de-usuario-a-respuesta-final al que pertenece este evento. Útil para registro y rastreo.

### Detectar Acciones y Efectos Secundarios

El objeto `event.actions` señala cambios que ocurrieron o deberían ocurrir. Siempre verifica si `event.actions` y sus campos/métodos existen antes de acceder a ellos.

*   **Cambios de Estado:** Te da una colección de pares clave-valor que se modificaron en el estado de la sesión durante el paso que produjo este evento.

    === "Python"
        `delta = event.actions.state_delta` (un diccionario de pares `{clave: valor}`).
        ```python
        if event.actions and event.actions.state_delta:
            print(f"  Cambios de estado: {event.actions.state_delta}")
            # Actualiza la UI local o el estado de la aplicación si es necesario
        ```
    === "Go"
        `delta := event.Actions.StateDelta` (un `map[string]any`)
        ```go
        import (
            "fmt"
            "google.golang.org/adk/session"
        )

        func handleStateChanges(event *session.Event) {
            if len(event.Actions.StateDelta) > 0 {
                fmt.Printf("  Cambios de estado: %v\n", event.Actions.StateDelta)
                // Actualiza la UI local o el estado de la aplicación si es necesario
            }
        }
        ```

    === "Java"
        `ConcurrentMap<String, Object> delta = event.actions().stateDelta();`

        ```java
        import java.util.concurrent.ConcurrentMap;
        import com.google.adk.events.EventActions;

        EventActions actions = event.actions(); // Asumiendo que event.actions() no es null
        if (actions != null && actions.stateDelta() != null && !actions.stateDelta().isEmpty()) {
            ConcurrentMap<String, Object> stateChanges = actions.stateDelta();
            System.out.println("  Cambios de estado: " + stateChanges);
            // Actualiza la UI local o el estado de la aplicación si es necesario
        }
        ```

*   **Guardados de Artefactos:** Te da una colección que indica qué artefactos se guardaron y su nuevo número de versión (o información relevante de `Part`).

    === "Python"
        `artifact_changes = event.actions.artifact_delta` (un diccionario de `{nombre_archivo: versión}`).
        ```python
        if event.actions and event.actions.artifact_delta:
            print(f"  Artefactos guardados: {event.actions.artifact_delta}")
            # La UI podría refrescar una lista de artefactos
        ```

    === "Go"
        `artifactChanges := event.Actions.ArtifactDelta` (un `map[string]artifact.Artifact`)
        ```go
        import (
            "fmt"
            "google.golang.org/adk/artifact"
            "google.golang.org/adk/session"
        )

        func handleArtifactChanges(event *session.Event) {
            if len(event.Actions.ArtifactDelta) > 0 {
                fmt.Printf("  Artefactos guardados: %v\n", event.Actions.ArtifactDelta)
                // La UI podría refrescar una lista de artefactos
                // Itera a través de event.Actions.ArtifactDelta para obtener nombre de archivo y detalles de artifact.Artifact
                for filename, art := range event.Actions.ArtifactDelta {
                    fmt.Printf("    Nombre de archivo: %s, Versión: %d, TipoMIME: %s\n", filename, art.Version, art.MIMEType)
                }
            }
        }
        ```

    === "Java"
        `ConcurrentMap<String, Part> artifactChanges = event.actions().artifactDelta();`

        ```java
        import java.util.concurrent.ConcurrentMap;
        import com.google.genai.types.Part;
        import com.google.adk.events.EventActions;

        EventActions actions = event.actions(); // Asumiendo que event.actions() no es null
        if (actions != null && actions.artifactDelta() != null && !actions.artifactDelta().isEmpty()) {
            ConcurrentMap<String, Part> artifactChanges = actions.artifactDelta();
            System.out.println("  Artefactos guardados: " + artifactChanges);
            // La UI podría refrescar una lista de artefactos
            // Itera a través de artifactChanges.entrySet() para obtener nombre de archivo y detalles de Part
        }
        ```

*   **Señales de Flujo de Control:** Verifica banderas booleanas o valores de cadena:

    === "Python"
        *   `event.actions.transfer_to_agent` (cadena): El control debe pasar al agente nombrado.
        *   `event.actions.escalate` (bool): Un bucle debe terminar.
        *   `event.actions.skip_summarization` (bool): Un resultado de herramienta no debe ser resumido por el LLM.
        ```python
        if event.actions:
            if event.actions.transfer_to_agent:
                print(f"  Señal: Transferir a {event.actions.transfer_to_agent}")
            if event.actions.escalate:
                print("  Señal: Escalar (terminar bucle)")
            if event.actions.skip_summarization:
                print("  Señal: Omitir resumen para resultado de herramienta")
        ```

    === "Go"
        *   `event.Actions.TransferToAgent` (cadena): El control debe pasar al agente nombrado.
        *   `event.Actions.Escalate` (bool): Un bucle debe terminar.
        *   `event.Actions.SkipSummarization` (bool): Un resultado de herramienta no debe ser resumido por el LLM.
        ```go
        import (
            "fmt"
            "google.golang.org/adk/session"
        )

        func handleControlFlow(event *session.Event) {
            if event.Actions.TransferToAgent != "" {
                fmt.Printf("  Señal: Transferir a %s\n", event.Actions.TransferToAgent)
            }
            if event.Actions.Escalate {
                fmt.Println("  Señal: Escalar (terminar bucle)")
            }
            if event.Actions.SkipSummarization {
                fmt.Println("  Señal: Omitir resumen para resultado de herramienta")
            }
        }
        ```

    === "Java"
        *   `event.actions().transferToAgent()` (devuelve `Optional<String>`): El control debe pasar al agente nombrado.
        *   `event.actions().escalate()` (devuelve `Optional<Boolean>`): Un bucle debe terminar.
        *   `event.actions().skipSummarization()` (devuelve `Optional<Boolean>`): Un resultado de herramienta no debe ser resumido por el LLM.

        ```java
        import com.google.adk.events.EventActions;
        import java.util.Optional;

        EventActions actions = event.actions(); // Asumiendo que event.actions() no es null
        if (actions != null) {
            Optional<String> transferAgent = actions.transferToAgent();
            if (transferAgent.isPresent()) {
                System.out.println("  Señal: Transferir a " + transferAgent.get());
            }

            Optional<Boolean> escalate = actions.escalate();
            if (escalate.orElse(false)) { // o escalate.isPresent() && escalate.get()
                System.out.println("  Señal: Escalar (terminar bucle)");
            }

            Optional<Boolean> skipSummarization = actions.skipSummarization();
            if (skipSummarization.orElse(false)) { // o skipSummarization.isPresent() && skipSummarization.get()
                System.out.println("  Señal: Omitir resumen para resultado de herramienta");
            }
        }
        ```

### Determinar si un Evento es una Respuesta "Final"

Usa el método auxiliar integrado `event.is_final_response()` para identificar eventos adecuados para mostrar como la salida completa del agente para un turno.

*   **Propósito:** Filtra pasos intermedios (como llamadas a herramientas, texto parcial en streaming, actualizaciones de estado internas) del mensaje final orientado al usuario.
*   **¿Cuándo es `True`?**
    1.  El evento contiene un resultado de herramienta (`function_response`) y `skip_summarization` es `True`.
    2.  El evento contiene una llamada a herramienta (`function_call`) para una herramienta marcada como `is_long_running=True`. En Java, verifica si la lista `longRunningToolIds` está vacía:
        *   `event.longRunningToolIds().isPresent() && !event.longRunningToolIds().get().isEmpty()` es `true`.
    3.  O, **todos** los siguientes se cumplen:
        *   Sin llamadas a funciones (`get_function_calls()` está vacío).
        *   Sin respuestas de funciones (`get_function_responses()` está vacío).
        *   No es un fragmento de flujo parcial (`partial` no es `True`).
        *   No termina con un resultado de ejecución de código que podría necesitar más procesamiento/visualización.
*   **Uso:** Filtra el flujo de eventos en la lógica de tu aplicación.

    === "Python"
        ```python
        # Pseudocódigo: Manejo de respuestas finales en la aplicación (Python)
        # full_response_text = ""
        # async for event in runner.run_async(...):
        #     # Acumula texto en streaming si es necesario...
        #     if event.partial and event.content and event.content.parts and event.content.parts[0].text:
        #         full_response_text += event.content.parts[0].text
        #
        #     # Verifica si es un evento final, mostrable
        #     if event.is_final_response():
        #         print("\n--- Salida Final Detectada ---")
        #         if event.content and event.content.parts and event.content.parts[0].text:
        #              # Si es la parte final de un flujo, usa el texto acumulado
        #              final_text = full_response_text + (event.content.parts[0].text if not event.partial else "")
        #              print(f"Mostrar al usuario: {final_text.strip()}")
        #              full_response_text = "" # Reinicia el acumulador
        #         elif event.actions and event.actions.skip_summarization and event.get_function_responses():
        #              # Maneja la visualización del resultado de herramienta crudo si es necesario
        #              response_data = event.get_function_responses()[0].response
        #              print(f"Mostrar resultado de herramienta crudo: {response_data}")
        #         elif hasattr(event, 'long_running_tool_ids') and event.long_running_tool_ids:
        #              print("Mostrar mensaje: La herramienta se está ejecutando en segundo plano...")
        #         else:
        #              # Maneja otros tipos de respuestas finales si aplica
        #              print("Mostrar: Respuesta final no textual o señal.")
        ```

    === "Go"

        ```go
        // Pseudocódigo: Manejo de respuestas finales en la aplicación (Go)
        import (
            "fmt"
            "strings"
            "google.golang.org/adk/session"
            "google.golang.org/genai"
        )

        // isFinalResponse verifica si un evento es una respuesta final adecuada para mostrar.
        func isFinalResponse(event *session.Event) bool {
            if event.LLMResponse != nil {
                // Condición 1: Resultado de herramienta con omisión de resumen.
                if event.LLMResponse.Content != nil && len(event.LLMResponse.Content.FunctionResponses()) > 0 && event.Actions.SkipSummarization {
                    return true
                }
                // Condición 2: Llamada a herramienta de larga duración.
                if len(event.LongRunningToolIDs) > 0 {
                    return true
                }
                // Condición 3: Un mensaje completo sin llamadas a herramientas o respuestas.
                if (event.LLMResponse.Content == nil ||
                    (len(event.LLMResponse.Content.FunctionCalls()) == 0 && len(event.LLMResponse.Content.FunctionResponses()) == 0)) &&
                    !event.LLMResponse.Partial {
                    return true
                }
            }
            return false
        }

        func handleFinalResponses() {
            var fullResponseText strings.Builder
            // for event := range runner.Run(...) { // Ejemplo de bucle
            // 	// Acumula texto en streaming si es necesario...
            // 	if event.LLMResponse != nil && event.LLMResponse.Partial && event.LLMResponse.Content != nil {
            // 		if len(event.LLMResponse.Content.Parts) > 0 && event.LLMResponse.Content.Parts[0].Text != "" {
            // 			fullResponseText.WriteString(event.LLMResponse.Content.Parts[0].Text)
            // 		}
            // 	}
            //
            // 	// Verifica si es un evento final, mostrable
            // 	if isFinalResponse(event) {
            // 		fmt.Println("\n--- Salida Final Detectada ---")
            // 		if event.LLMResponse != nil && event.LLMResponse.Content != nil {
            // 			if len(event.LLMResponse.Content.Parts) > 0 && event.LLMResponse.Content.Parts[0].Text != "" {
            // 				// Si es la parte final de un flujo, usa el texto acumulado
            // 				finalText := fullResponseText.String()
            // 				if !event.LLMResponse.Partial {
            // 					finalText += event.LLMResponse.Content.Parts[0].Text
            // 				}
            // 				fmt.Printf("Mostrar al usuario: %s\n", strings.TrimSpace(finalText))
            // 				fullResponseText.Reset() // Reinicia el acumulador
            // 			}
            // 		} else if event.Actions.SkipSummarization && event.LLMResponse.Content != nil && len(event.LLMResponse.Content.FunctionResponses()) > 0 {
            // 			// Maneja la visualización del resultado de herramienta crudo si es necesario
            // 			responseData := event.LLMResponse.Content.FunctionResponses()[0].Response
            // 			fmt.Printf("Mostrar resultado de herramienta crudo: %v\n", responseData)
            // 		} else if len(event.LongRunningToolIDs) > 0 {
            // 			fmt.Println("Mostrar mensaje: La herramienta se está ejecutando en segundo plano...")
            // 		} else {
            // 			// Maneja otros tipos de respuestas finales si aplica
            // 			fmt.Println("Mostrar: Respuesta final no textual o señal.")
            // 		}
            // 	}
            // }
        }
        ```

    === "Java"
        ```java
        // Pseudocódigo: Manejo de respuestas finales en la aplicación (Java)
        import com.google.adk.events.Event;
        import com.google.genai.types.Content;
        import com.google.genai.types.FunctionResponse;
        import java.util.Map;

        StringBuilder fullResponseText = new StringBuilder();
        runner.run(...).forEach(event -> { // Asumiendo un flujo de eventos
             // Acumula texto en streaming si es necesario...
             if (event.partial().orElse(false) && event.content().isPresent()) {
                 event.content().flatMap(Content::parts).ifPresent(parts -> {
                     if (!parts.isEmpty() && parts.get(0).text().isPresent()) {
                         fullResponseText.append(parts.get(0).text().get());
                    }
                 });
             }

             // Verifica si es un evento final, mostrable
             if (event.finalResponse()) { // Usando el método de Event.java
                 System.out.println("\n--- Salida Final Detectada ---");
                 if (event.content().isPresent() &&
                     event.content().flatMap(Content::parts).map(parts -> !parts.isEmpty() && parts.get(0).text().isPresent()).orElse(false)) {
                     // Si es la parte final de un flujo, usa el texto acumulado
                     String eventText = event.content().get().parts().get().get(0).text().get();
                     String finalText = fullResponseText.toString() + (event.partial().orElse(false) ? "" : eventText);
                     System.out.println("Mostrar al usuario: " + finalText.trim());
                     fullResponseText.setLength(0); // Reinicia el acumulador
                 } else if (event.actions() != null && event.actions().skipSummarization().orElse(false)
                            && !event.functionResponses().isEmpty()) {
                     // Maneja la visualización del resultado de herramienta crudo si es necesario,
                     // especialmente si finalResponse() fue verdadero debido a otras condiciones
                     // o si quieres mostrar resultados de resumen omitido independientemente de finalResponse()
                     Map<String, Object> responseData = event.functionResponses().get(0).response().get();
                     System.out.println("Mostrar resultado de herramienta crudo: " + responseData);
                 } else if (event.longRunningToolIds().isPresent() && !event.longRunningToolIds().get().isEmpty()) {
                     // Este caso está cubierto por event.finalResponse()
                     System.out.println("Mostrar mensaje: La herramienta se está ejecutando en segundo plano...");
                 } else {
                     // Maneja otros tipos de respuestas finales si aplica
                     System.out.println("Mostrar: Respuesta final no textual o señal.");
                 }
             }
         });
        ```

Al examinar cuidadosamente estos aspectos de un evento, puedes construir aplicaciones robustas que reaccionen apropiadamente a la rica información que fluye a través del sistema ADK.

## Cómo Fluyen los Eventos: Generación y Procesamiento

Los eventos se crean en diferentes puntos y son procesados sistemáticamente por el framework. Comprender este flujo ayuda a aclarar cómo se gestionan las acciones y el historial.

*   **Fuentes de Generación:**
    *   **Entrada de Usuario:** El `Runner` típicamente envuelve los mensajes iniciales del usuario o las entradas a mitad de conversación en un `Event` con `author='user'`.
    *   **Lógica del Agente:** Los agentes (`BaseAgent`, `LlmAgent`) explícitamente hacen `yield Event(...)` objetos (estableciendo `author=self.name`) para comunicar respuestas o señalar acciones.
    *   **Respuestas del LLM:** La capa de integración del modelo ADK traduce la salida cruda del LLM (texto, llamadas a funciones, errores) en objetos `Event`, autorados por el agente que llama.
    *   **Resultados de Herramientas:** Después de que una herramienta se ejecuta, el framework genera un `Event` que contiene el `function_response`. El `author` es típicamente el agente que solicitó la herramienta, mientras que el `role` dentro del `content` se establece a `'user'` para el historial del LLM.


*   **Flujo de Procesamiento:**
    1.  **Yield/Return:** Se genera un evento y se hace yield (Python) o se devuelve/emite (Java) por su fuente.
    2.  **El Runner Recibe:** El `Runner` principal que ejecuta el agente recibe el evento.
    3.  **Procesamiento del SessionService:** El `Runner` envía el evento al `SessionService` configurado. Este es un paso crítico:
        *   **Aplica Deltas:** El servicio fusiona `event.actions.state_delta` en `session.state` y actualiza registros internos basándose en `event.actions.artifact_delta`. (Nota: El *guardado* real del artefacto usualmente ocurrió antes cuando se llamó `context.save_artifact`).
        *   **Finaliza Metadatos:** Asigna un `event.id` único si no está presente, puede actualizar `event.timestamp`.
        *   **Persiste en el Historial:** Agrega el evento procesado a la lista `session.events`.
    4.  **Yield Externo:** El `Runner` hace yield (Python) o devuelve/emite (Java) el evento procesado hacia afuera a la aplicación que llama (por ejemplo, el código que invocó `runner.run_async`).

Este flujo asegura que los cambios de estado y el historial se registren consistentemente junto con el contenido de comunicación de cada evento.


## Ejemplos Comunes de Eventos (Patrones Ilustrativos)

Aquí hay ejemplos concisos de eventos típicos que podrías ver en el flujo:

*   **Entrada de Usuario:**
    ```json
    {
      "author": "user",
      "invocation_id": "e-xyz...",
      "content": {"parts": [{"text": "Reserva un vuelo a Londres para el próximo martes"}]}
      // actions usualmente vacío
    }
    ```
*   **Respuesta de Texto Final del Agente:** (`is_final_response() == True`)
    ```json
    {
      "author": "TravelAgent",
      "invocation_id": "e-xyz...",
      "content": {"parts": [{"text": "Está bien, puedo ayudarte con eso. ¿Podrías confirmar la ciudad de salida?"}]},
      "partial": false,
      "turn_complete": true
      // actions podría tener delta de estado, etc.
    }
    ```
*   **Respuesta de Texto en Streaming del Agente:** (`is_final_response() == False`)
    ```json
    {
      "author": "SummaryAgent",
      "invocation_id": "e-abc...",
      "content": {"parts": [{"text": "El documento discute tres puntos principales:"}]},
      "partial": true,
      "turn_complete": false
    }
    // ... siguen más eventos con partial=True ...
    ```
*   **Solicitud de Llamada a Herramienta (por LLM):** (`is_final_response() == False`)
    ```json
    {
      "author": "TravelAgent",
      "invocation_id": "e-xyz...",
      "content": {"parts": [{"function_call": {"name": "find_airports", "args": {"city": "London"}}}]}
      // actions usualmente vacío
    }
    ```
*   **Resultado de Herramienta Proporcionado (al LLM):** (`is_final_response()` depende de `skip_summarization`)
    ```json
    {
      "author": "TravelAgent", // El autor es el agente que solicitó la llamada
      "invocation_id": "e-xyz...",
      "content": {
        "role": "user", // Rol para el historial del LLM
        "parts": [{"function_response": {"name": "find_airports", "response": {"result": ["LHR", "LGW", "STN"]}}}]
      }
      // actions podría tener skip_summarization=True
    }
    ```
*   **Actualización Solo de Estado/Artefacto:** (`is_final_response() == False`)
    ```json
    {
      "author": "InternalUpdater",
      "invocation_id": "e-def...",
      "content": null,
      "actions": {
        "state_delta": {"user_status": "verified"},
        "artifact_delta": {"verification_doc.pdf": 2}
      }
    }
    ```
*   **Señal de Transferencia de Agente:** (`is_final_response() == False`)
    ```json
    {
      "author": "OrchestratorAgent",
      "invocation_id": "e-789...",
      "content": {"parts": [{"function_call": {"name": "transfer_to_agent", "args": {"agent_name": "BillingAgent"}}}]},
      "actions": {"transfer_to_agent": "BillingAgent"} // Añadido por el framework
    }
    ```
*   **Señal de Escalación de Bucle:** (`is_final_response() == False`)
    ```json
    {
      "author": "CheckerAgent",
      "invocation_id": "e-loop...",
      "content": {"parts": [{"text": "Se alcanzó el máximo de reintentos."}]}, // Contenido opcional
      "actions": {"escalate": true}
    }
    ```

## Contexto Adicional y Detalles de Eventos

Más allá de los conceptos centrales, aquí hay algunos detalles específicos sobre el contexto y los eventos que son importantes para ciertos casos de uso:

1.  **`ToolContext.function_call_id` (Vinculación de Acciones de Herramientas):**
    *   Cuando un LLM solicita una herramienta (FunctionCall), esa solicitud tiene un ID. El `ToolContext` proporcionado a tu función de herramienta incluye este `function_call_id`.
    *   **Importancia:** Este ID es crucial para vincular acciones como la autenticación de vuelta a la solicitud de herramienta específica que las inició, especialmente si se llaman múltiples herramientas en un turno. El framework usa este ID internamente.

2.  **Cómo se Registran los Cambios de Estado/Artefactos:**
    *   Cuando modificas el estado o guardas un artefacto usando `CallbackContext` o `ToolContext`, estos cambios no se escriben inmediatamente en el almacenamiento persistente.
    *   En su lugar, pueblan los campos `state_delta` y `artifact_delta` dentro del objeto `EventActions`.
    *   Este objeto `EventActions` se adjunta al *siguiente evento* generado después del cambio (por ejemplo, la respuesta del agente o un evento de resultado de herramienta).
    *   El método `SessionService.append_event` lee estos deltas del evento entrante y los aplica al estado persistente de la sesión y los registros de artefactos. Esto asegura que los cambios estén vinculados cronológicamente al flujo de eventos.

3.  **Prefijos de Alcance de Estado (`app:`, `user:`, `temp:`):**
    *   Al gestionar el estado mediante `context.state`, puedes usar opcionalmente prefijos:
        *   `app:mi_configuracion`: Sugiere estado relevante para toda la aplicación (requiere un `SessionService` persistente).
        *   `user:preferencia_usuario`: Sugiere estado relevante para el usuario específico a través de las sesiones (requiere un `SessionService` persistente).
        *   `temp:resultado_intermedio` o sin prefijo: Típicamente estado específico de la sesión o temporal para la invocación actual.
    *   El `SessionService` subyacente determina cómo se manejan estos prefijos para la persistencia.

4.  **Eventos de Error:**
    *   Un `Event` puede representar un error. Verifica los campos `event.error_code` y `event.error_message` (heredados de `LlmResponse`).
    *   Los errores pueden originarse del LLM (por ejemplo, filtros de seguridad, límites de recursos) o potencialmente ser empaquetados por el framework si una herramienta falla críticamente. Verifica el contenido de `FunctionResponse` de la herramienta para errores típicos específicos de herramientas.
    ```json
    // Ejemplo de Evento de Error (conceptual)
    {
      "author": "LLMAgent",
      "invocation_id": "e-err...",
      "content": null,
      "error_code": "SAFETY_FILTER_TRIGGERED",
      "error_message": "Respuesta bloqueada debido a configuraciones de seguridad.",
      "actions": {}
    }
    ```

Estos detalles proporcionan una imagen más completa para casos de uso avanzados que involucran autenticación de herramientas, alcance de persistencia de estado y manejo de errores dentro del flujo de eventos.

## Mejores Prácticas para Trabajar con Eventos

Para usar eventos efectivamente en tus aplicaciones ADK:

*   **Autoría Clara:** Al construir agentes personalizados, asegura la atribución correcta para las acciones del agente en el historial. El framework generalmente maneja la autoría correctamente para eventos de LLM/herramientas.

    === "Python"
        Usa `yield Event(author=self.name, ...)` en subclases de `BaseAgent`.

    === "Go"
        En métodos `Run` de agentes personalizados, el framework típicamente maneja la autoría. Si creas un evento manualmente, establece el autor: `yield(&session.Event{Author: a.name, ...}, nil)`

    === "Java"
        Al construir un `Event` en la lógica de tu agente personalizado, establece el autor, por ejemplo: `Event.builder().author(this.getAgentName()) // ... .build();`

*   **Contenido Semántico y Acciones:** Usa `event.content` para el mensaje/datos centrales (texto, llamada/respuesta de función). Usa `event.actions` específicamente para señalar efectos secundarios (deltas de estado/artefactos) o flujo de control (`transfer`, `escalate`, `skip_summarization`).
*   **Conciencia de Idempotencia:** Comprende que el `SessionService` es responsable de aplicar los cambios de estado/artefactos señalados en `event.actions`. Si bien los servicios ADK apuntan a la consistencia, considera los efectos posteriores potenciales si la lógica de tu aplicación vuelve a procesar eventos.
*   **Usa `is_final_response()`:** Confía en este método auxiliar en tu capa de aplicación/UI para identificar respuestas de texto completas orientadas al usuario. Evita replicar manualmente su lógica.
*   **Aprovecha el Historial:** La lista de eventos de la sesión es tu herramienta de depuración principal. Examina la secuencia de autores, contenido y acciones para rastrear la ejecución y diagnosticar problemas.
*   **Usa Metadatos:** Usa `invocation_id` para correlacionar todos los eventos dentro de una sola interacción de usuario. Usa `event.id` para referenciar ocurrencias específicas y únicas.

Tratar los eventos como mensajes estructurados con propósitos claros para su contenido y acciones es clave para construir, depurar y gestionar comportamientos de agentes complejos en ADK.