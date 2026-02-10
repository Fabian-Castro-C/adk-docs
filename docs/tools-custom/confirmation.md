# Obtener confirmación de acción para Herramientas ADK

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v1.14.0</span><span class="lst-preview">Experimental</span>
</div>

Algunos flujos de trabajo de agentes requieren confirmación para la toma de decisiones, verificación,
seguridad o supervisión general. En estos casos, deseas obtener una respuesta de
un humano o sistema supervisor antes de proceder con un flujo de trabajo. La función de *Confirmación
de Herramienta* en el Agent Development Kit (ADK) permite que una Herramienta ADK
pause su ejecución e interactúe con un usuario u otro sistema para confirmación o
para recopilar datos estructurados antes de proceder. Puedes usar la Confirmación de Herramienta con
una Herramienta ADK de las siguientes maneras:

-   **[Confirmación Booleana](#boolean-confirmation):** Puedes
    configurar un FunctionTool con un parámetro `require_confirmation`. Esta
    opción pausa la herramienta para una respuesta de confirmación sí o no.
-   **[Confirmación Avanzada](#advanced-confirmation):** Para escenarios que requieren
    respuestas de datos estructurados, puedes configurar un `FunctionTool` con un texto
    de solicitud para explicar la confirmación y una respuesta esperada.

!!! example "Experimental"
    La función de Confirmación de Herramienta es experimental y tiene algunas
    [limitaciones conocidas](#known-limitations).
    ¡Agradecemos tus
    [comentarios](https://github.com/google/adk-python/issues/new?template=feature_request.md&labels=tool%20confirmation)!

Puedes configurar cómo se comunica una solicitud a un usuario, y el sistema también puede
usar [respuestas remotas](#remote-response) enviadas a través de la API
REST del servidor ADK. Cuando uses la función de confirmación con la interfaz de usuario web del ADK,
el flujo de trabajo del agente muestra un cuadro de diálogo al usuario para solicitar
entrada, como se muestra en la Figura 1:

![Screenshot of default user interface for tool confirmation](/assets/confirmation-ui.png)

**Figura 1.** Ejemplo de cuadro de diálogo de solicitud de respuesta de confirmación usando una
implementación de respuesta de herramienta avanzada.

Las siguientes secciones describen cómo usar esta función para los escenarios de confirmación.
Para un ejemplo de código completo, consulta el ejemplo
[human_tool_confirmation](https://github.com/google/adk-python/blob/fc90ce968f114f84b14829f8117797a4c256d710/contributing/samples/human_tool_confirmation/agent.py).
Hay formas adicionales de incorporar entrada humana en tu flujo de trabajo de agente,
para más detalles, consulta el patrón de agente
[Human-in-the-loop](/agents/multi-agents/#human-in-the-loop-pattern).

## Confirmación booleana {#boolean-confirmation}

Cuando tu herramienta solo requiere un simple `yes` o `no` del usuario, puedes
agregar un paso de confirmación usando la clase `FunctionTool` como envoltura. Por
ejemplo, si tienes una herramienta llamada `reimburse`, puedes habilitar un paso de
confirmación envolviéndola con la clase `FunctionTool` y estableciendo el
parámetro `require_confirmation` en `True`, como se muestra en el siguiente ejemplo:

```
# From agent.py
root_agent = Agent(
   ...
   tools=[
        # Establece require_confirmation en True para requerir confirmación del usuario
        # para la llamada de la herramienta.
        FunctionTool(reimburse, require_confirmation=True),
    ],
...
```

Este método de implementación requiere código mínimo, pero está limitado a aprobaciones
simples del usuario o sistema de confirmación. Para un ejemplo completo de este
enfoque, consulta el ejemplo de código
[human_tool_confirmation](https://github.com/google/adk-python/blob/fc90ce968f114f84b14829f8117797a4c256d710/contributing/samples/human_tool_confirmation/agent.py).

### Función de requerir confirmación

Puedes modificar el comportamiento de la respuesta `require_confirmation` reemplazando su
valor de entrada con una función que devuelve una respuesta booleana. El siguiente
ejemplo muestra una función para determinar si se requiere una confirmación:

```
async def confirmation_threshold(
    amount: int, tool_context: ToolContext
) -> bool:
  """Devuelve true si el monto es mayor que 1000."""
  return amount > 1000
```

Esta función puede entonces establecerse como el valor del parámetro para el
parámetro `require_confirmation`:

```
root_agent = Agent(
   ...
   tools=[
        # Establece require_confirmation en True para requerir confirmación del usuario
        FunctionTool(reimburse, require_confirmation=confirmation_threshold),
    ],
...
```

Para un ejemplo completo de esta implementación, consulta el ejemplo de código
[human_tool_confirmation](https://github.com/google/adk-python/blob/fc90ce968f114f84b14829f8117797a4c256d710/contributing/samples/human_tool_confirmation/agent.py).

## Confirmación avanzada {#advanced-confirmation}

Cuando una confirmación de herramienta requiere más detalles para el usuario o una respuesta
más compleja, usa una implementación de tool_confirmation. Este enfoque extiende el
objeto `ToolContext` para agregar una descripción de texto de la solicitud para el usuario y
permite datos de respuesta más complejos. Al implementar la confirmación de herramienta de esta
manera, puedes pausar la ejecución de una herramienta, solicitar información específica y luego
reanudar la herramienta con los datos proporcionados.

Este flujo de confirmación tiene una etapa de solicitud donde el sistema ensambla y envía
una solicitud de entrada de respuesta humana, y una etapa de respuesta donde el sistema recibe
y procesa los datos devueltos.

### Definición de confirmación

Al crear una Herramienta con una confirmación avanzada, crea una función que
incluya un objeto ToolContext. Luego define la confirmación usando un
objeto tool_confirmation, el método `tool_context.request_confirmation()` con
parámetros `hint` y `payload`. Estas propiedades se usan de la siguiente manera:

-   `hint`: Mensaje descriptivo que explica qué se necesita del usuario.
-   `payload`: La estructura de los datos que esperas como respuesta. Este tipo de
    datos es Any y debe ser serializable en una cadena con formato JSON, como un
    diccionario o modelo pydantic.

El siguiente código muestra una implementación de ejemplo para una herramienta que procesa
solicitudes de tiempo libre para un empleado:

```
def request_time_off(days: int, tool_context: ToolContext):
  """Solicitar día libre para el empleado."""
  ...
  tool_confirmation = tool_context.tool_confirmation
  if not tool_confirmation:
    tool_context.request_confirmation(
        hint=(
            'Por favor aprueba o rechaza la solicitud de llamada de herramienta request_time_off() '
            'respondiendo con un FunctionResponse con un payload de '
            'ToolConfirmation esperado.'
        ),
        payload={
            'approved_days': 0,
        },
    )
    # Devuelve el estado intermedio indicando que la herramienta está esperando una
    # respuesta de confirmación:
    return {'status': 'Se requiere la aprobación del gerente.'}

  approved_days = tool_confirmation.payload['approved_days']
  approved_days = min(approved_days, days)
  if approved_days == 0:
    return {'status': 'La solicitud de tiempo libre es rechazada.', 'approved_days': 0}
  return {
      'status': 'ok',
      'approved_days': approved_days,
  }
```

Para un ejemplo completo de este enfoque, consulta el ejemplo de código
[human_tool_confirmation](https://github.com/google/adk-python/blob/fc90ce968f114f84b14829f8117797a4c256d710/contributing/samples/human_tool_confirmation/agent.py).
Ten en cuenta que la ejecución de la herramienta del flujo de trabajo del agente se pausa mientras se
obtiene una confirmación. Después de recibir la confirmación, puedes acceder a la
respuesta de confirmación en el objeto `tool_confirmation.payload` y luego proceder
con la ejecución del flujo de trabajo.

## Confirmación remota con API REST {#remote-response}

Si no hay una interfaz de usuario activa para una confirmación humana de un flujo de
trabajo del agente, puedes manejar la confirmación a través de una interfaz de línea de comandos o
enrutándola a través de otro canal como correo electrónico o una aplicación de chat. Para confirmar
la llamada de la herramienta, el usuario o aplicación que llama necesita enviar un
evento `FunctionResponse` con los datos de confirmación de la herramienta.

Puedes enviar la solicitud al endpoint `/run` o `/run_sse` del servidor API de ADK,
o directamente al ejecutor ADK. El siguiente ejemplo usa un comando `curl` para
enviar la confirmación al endpoint `/run_sse`:

```
 curl -X POST http://localhost:8000/run_sse \
 -H "Content-Type: application/json" \
 -d '{
    "app_name": "human_tool_confirmation",
    "user_id": "user",
    "session_id": "7828f575-2402-489f-8079-74ea95b6a300",
    "new_message": {
        "parts": [
            {
                "function_response": {
                    "id": "adk-13b84a8c-c95c-4d66-b006-d72b30447e35",
                    "name": "adk_request_confirmation",
                    "response": {
                        "confirmed": true
                    }
                }
            }
        ],
        "role": "user"
    }
}'
```

Una respuesta basada en REST para una confirmación debe cumplir con los siguientes
requisitos:

-   El `id` en el `function_response` debe coincidir con el `function_call_id`
    del evento `FunctionCall` de `RequestConfirmation`.
-   El `name` debe ser `adk_request_confirmation`.
-   El objeto `response` contiene el estado de confirmación y cualquier
    dato de payload adicional requerido por la herramienta.

!!! note "Nota: Confirmación con función Resume"

    Si tu flujo de trabajo del agente ADK está configurado con la
    función [Resume](/runtime/resume/), también debes incluir
    el parámetro de ID de Invocación (`invocation_id`) con la respuesta de
    confirmación. El ID de Invocación que proporciones debe ser la misma invocación
    que generó la solicitud de confirmación, de lo contrario el sistema
    inicia una nueva invocación con la respuesta de confirmación. Si tu
    agente usa la función Resume, considera incluir el ID de Invocación
    como parámetro con tu solicitud de confirmación, para que pueda ser
    incluido con la respuesta. Para más detalles sobre el uso de la función
    Resume, consulta
    [Resume stopped agents](/runtime/resume/).

## Limitaciones conocidas {#known-limitations}

La función de confirmación de herramienta tiene las siguientes limitaciones:

-   [DatabaseSessionService](/api-reference/python/google-adk.html#google.adk.sessions.DatabaseSessionService)
    no es compatible con esta función.
-   [VertexAiSessionService](/api-reference/python/google-adk.html#google.adk.sessions.VertexAiSessionService)
    no es compatible con esta función.

## Próximos pasos

Para más información sobre la construcción de herramientas ADK para flujos de trabajo de agentes, consulta [Function
tools](/tools-custom/function-tools/).