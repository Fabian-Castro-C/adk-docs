# Sesiones Rewind para agentes

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v1.17.0</span>
</div>

La función Rewind de sesiones de ADK te permite revertir una sesión a un estado de solicitud anterior, permitiéndote deshacer errores, explorar rutas alternativas o reiniciar un proceso desde un punto conocido bueno. Este documento proporciona una descripción general de la función, cómo usarla y sus limitaciones.

## Revertir una sesión

Cuando reviertes una sesión, especificas una solicitud de usuario, o ***invocación***, que deseas deshacer, y el sistema deshace esa solicitud y las solicitudes posteriores. Entonces, si tienes tres solicitudes (A, B, C) y quieres volver al estado en la solicitud A, especificas B, lo que deshace los cambios de las solicitudes B y C. Reviertes una sesión usando el método rewind en una instancia de ***Runner***, especificando el usuario, la sesión y el id de invocación, como se muestra en el siguiente fragmento de código:

```python
# Crear runner
runner = InMemoryRunner(
    agent=agent.root_agent,
    app_name=APP_NAME,
)

# Crear una sesión
session = await runner.session_service.create_session(
    app_name=APP_NAME, user_id=USER_ID
)
# llamar al agente con la función wrapper "call_agent_async()"
await call_agent_async(
    runner, USER_ID, session.id, "set state color to red"
)
# ... más llamadas al agente ...
events_list = await call_agent_async(
    runner, USER_ID, session.id, "update state color to blue"
)

# obtener id de invocación
rewind_invocation_id=events_list[1].invocation_id

# revertir invocaciones (state color: red)
await runner.rewind_async(
    user_id=USER_ID,
    session_id=session.id,
    rewind_before_invocation_id=rewind_invocation_id,
)
```

Cuando llamas al método ***rewind***, todos los recursos de nivel de sesión administrados por ADK se restauran al estado en el que estaban *antes* de la solicitud que especificaste con el ***id de invocación***. Sin embargo, los recursos globales, como el estado y los artefactos a nivel de aplicación o de usuario, no se restauran. Para un ejemplo completo de una reversión de sesión de agente, consulta el código de muestra [rewind_session](https://github.com/google/adk-python/tree/main/contributing/samples/rewind_session). Para más información sobre las limitaciones de la función Rewind, consulta [Limitaciones](#limitations).

## Cómo funciona

La función Rewind crea una solicitud especial de ***rewind*** que restaura el estado y los artefactos de la sesión a su condición *antes* del punto de reversión especificado por un id de invocación. Este enfoque significa que todas las solicitudes, incluidas las solicitudes revertidas, se conservan en el registro para depuración, análisis o auditoría posteriores. Después de la reversión, el sistema ignora las solicitudes revertidas cuando prepara las siguientes solicitudes para el modelo de IA. Este comportamiento significa que el modelo de IA utilizado por el agente efectivamente olvida cualquier interacción desde el punto de reversión hasta la siguiente solicitud.

## Limitaciones {#limitations}

La función Rewind tiene algunas limitaciones que debes tener en cuenta al usarla con tu flujo de trabajo del agente:

*   **Recursos globales del agente:** El estado y los artefactos a nivel de aplicación y de usuario *no* se restauran con la función rewind. Solo se restauran el estado y los artefactos a nivel de sesión.
*   **Dependencias externas:** La función rewind no administra dependencias externas. Si una herramienta en tu agente interactúa con sistemas externos, es tu responsabilidad manejar la restauración de esos sistemas a su estado anterior.
*   **Atomicidad:** Las actualizaciones de estado, las actualizaciones de artefactos y la persistencia de eventos no se realizan en una única transacción atómica. Por lo tanto, debes evitar revertir sesiones activas o manipular simultáneamente artefactos de sesión durante una reversión para prevenir inconsistencias.