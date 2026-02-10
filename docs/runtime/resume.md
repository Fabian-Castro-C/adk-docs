# Reanudar agentes detenidos

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v1.14.0</span>
</div>

La ejecución de un agente ADK puede ser interrumpida por varios factores incluyendo
conexiones de red caídas, falla de energía, o un sistema externo requerido quedando
fuera de línea. La característica de Reanudación de ADK permite que un flujo de trabajo de agente retome desde donde se
quedó, evitando la necesidad de reiniciar todo el flujo de trabajo. En ADK Python 1.16
y superior, puedes configurar un flujo de trabajo ADK para que sea reanudable, de modo que rastree
la ejecución del flujo de trabajo y luego te permita reanudarlo después de una
interrupción inesperada.

Esta guía explica cómo configurar tu flujo de trabajo de agente ADK para que sea reanudable.
Si usas Agentes Personalizados, puedes actualizarlos para que sean reanudables. Para más
información, consulta 
[Agregar reanudación a Agentes personalizados](#custom-agents).

## Agregar configuración reanudable

Habilita la función de Reanudación para un flujo de trabajo de agente aplicando una configuración de
Reanudabilidad al objeto App de tu flujo de trabajo ADK, como se muestra en el siguiente
ejemplo de código:

```python
app = App(
    name='my_resumable_agent',
    root_agent=root_agent,
    # Establece la configuración de reanudabilidad para habilitar la reanudabilidad.
    resumability_config=ResumabilityConfig(
        is_resumable=True,
    ),
)
```

!!! warning "Precaución: Funciones de Larga Ejecución, Confirmaciones, Autenticación"
    Para agentes que usan
    [Funciones de Larga Ejecución](/tools-custom/function-tools/#long-run-tool),
    [Confirmaciones](/tools-custom/confirmation/), o
    [Autenticación](/tools-custom/authentication/)
    que requieren entrada del usuario, agregar una confirmación reanudable cambia cómo operan estas características.
    Para más información, consulta la documentación para esas características.

!!! info "Nota: Agentes Personalizados"
    La Reanudación no está soportada por defecto para Agentes Personalizados. Debes
    actualizar el código del agente para un Agente Personalizado para soportar la característica de Reanudación. Para
    información sobre modificar Agentes Personalizados para soportar funcionalidad de reanudación incremental,
    consulta 
    [Agregar reanudación a Agentes personalizados](#custom-agents).

## Reanudar un flujo de trabajo detenido

Cuando un flujo de trabajo ADK detiene su ejecución puedes reanudar el flujo de trabajo usando un
comando que contenga el ID de Invocación para la instancia del flujo de trabajo, que puede ser
encontrado en el historial de
[Eventos](/events/#understanding-and-using-events)
del flujo de trabajo. Asegúrate de que el servidor API de ADK esté ejecutándose, en caso de que fuera
interrumpido o apagado, y luego ejecuta el siguiente comando para reanudar el
flujo de trabajo, como se muestra en el siguiente ejemplo de solicitud API.

```console
# reinicia el servidor API si es necesario:
adk api_server my_resumable_agent/

# reanuda el agente:
curl -X POST http://localhost:8000/run_sse \
 -H "Content-Type: application/json" \
 -d '{
   "app_name": "my_resumable_agent",
   "user_id": "u_123",
   "session_id": "s_abc",
   "invocation_id": "invocation-123",
 }'
```

También puedes reanudar un flujo de trabajo usando el método Run Async del objeto Runner, como se
muestra a continuación:

```python
runner.run_async(user_id='u_123', session_id='s_abc', 
    invocation_id='invocation-123')

# Cuando new_message se establece a una respuesta de función,
# estamos intentando reanudar una función de larga ejecución.
```

!!! info "Nota"
    Reanudar un flujo de trabajo desde la interfaz de usuario web de ADK o usando la herramienta de
    línea de comandos (CLI) de ADK actualmente no está soportado.

## Cómo funciona

La característica de Reanudación funciona registrando las tareas completadas del flujo de trabajo del Agente,
incluyendo pasos incrementales usando
[Eventos](/events/) y
[Acciones de Eventos](/events/#detecting-actions-and-side-effects).
rastreando la finalización de tareas del agente dentro de un flujo de trabajo reanudable. Si un flujo de trabajo es
interrumpido y luego reiniciado más tarde, el sistema reanuda el flujo de trabajo estableciendo
el estado de finalización de cada agente. Si un agente no se completó, el sistema del flujo de trabajo
restablece cualquier Evento completado para ese agente, y reinicia el flujo de trabajo
desde el estado parcialmente completado. Para flujos de trabajo multi-agente, el comportamiento específico de
reanudación varía, basándose en las clases multi-agente en tu flujo de trabajo, como se
describe a continuación:

-   **Agente Secuencial**: Lee el current_sub_agent de su estado guardado
    para encontrar el siguiente sub-agente a ejecutar en la secuencia.
-   **Agente de Bucle**: Usa los valores current_sub_agent y times_looped para
    continuar el bucle desde la última iteración y sub-agente completados.
-   **Agente Paralelo**: Determina qué sub-agentes ya han completado
    y solo ejecuta aquellos que no han terminado.

El registro de eventos incluye resultados de Herramientas que devolvieron exitosamente un resultado.
Así que si un agente ejecutó exitosamente las Herramientas de Función A y B, y luego falló
durante la ejecución de la herramienta C, el sistema restablece los resultados de las
herramientas A y B, y reanuda el flujo de trabajo re-ejecutando la solicitud de la herramienta C.

!!! warning "Precaución: Comportamiento de ejecución de Herramientas"
    Al reanudar un flujo de trabajo con Herramientas, la característica de Reanudación asegura
    que las Herramientas en un agente se ejecuten ***al menos una vez***, y pueden ejecutarse más de
    una vez al reanudar un flujo de trabajo. Si tu agente usa Herramientas donde ejecuciones duplicadas
    tendrían un impacto negativo, como compras, debes modificar la Herramienta para
    verificar y prevenir ejecuciones duplicadas.

!!! note "Nota: Modificación de flujo de trabajo con Reanudación no soportada"
    No modifiques un flujo de trabajo de agente detenido antes de reanudarlo. 
    Por ejemplo, agregar o eliminar agentes de un flujo de trabajo que se ha detenido
    y luego reanudar ese flujo de trabajo no está soportado.

## Agregar reanudación a Agentes personalizados {#custom-agents}

Los agentes personalizados tienen requisitos de implementación específicos para soportar
reanudabilidad. Debes decidir y definir pasos del flujo de trabajo dentro de tu agente
personalizado que produzcan un resultado que pueda ser preservado antes de pasar al
siguiente paso de procesamiento. Los siguientes pasos describen cómo modificar un Agente
Personalizado para soportar una Reanudación de flujo de trabajo.

-   **Crear clase CustomAgentState**: Extiende el BaseAgentState para crear
    un objeto que preserve el estado de tu agente.
    -   **Opcionalmente, crear clase WorkFlowStep**: Si tu agente personalizado
        tiene pasos secuenciales, considera crear un objeto de lista WorkFlowStep que
        defina los pasos discretos y guardables del agente.
-   **Agregar estado inicial del agente:** Modifica la función async run de tu agente para
    establecer el estado inicial de tu agente.
-   **Agregar puntos de control del estado del agente**: Modifica la función async run de tu agente
    para generar y guardar el estado del agente para cada paso completado de la
    tarea general del agente.
-   **Agregar fin del estado del agente para rastrear el estado del agente:** Modifica la función async run de tu agente
    para incluir un estado `end_of_agent=True` al completar exitosamente
    la tarea completa del agente.

El siguiente ejemplo muestra las modificaciones de código requeridas al ejemplo de la
clase StoryFlowAgent mostrada en la guía de
[Agentes Personalizados](/agents/custom-agents/#full-code-example):

```python
class WorkflowStep(int, Enum):
 INITIAL_STORY_GENERATION = 1
 CRITIC_REVISER_LOOP = 2
 POST_PROCESSING = 3
 CONDITIONAL_REGENERATION = 4

# Extender BaseAgentState

### class StoryFlowAgentState(BaseAgentState):

###   step = WorkflowStep

@override
async def _run_async_impl(
    self, ctx: InvocationContext
) -> AsyncGenerator[Event, None]:
    """
    Implementa la lógica de orquestación personalizada para el flujo de trabajo de la historia.
    Usa los atributos de instancia asignados por Pydantic (ej., self.story_generator).
    """
    agent_state = self._load_agent_state(ctx, WorkflowStep)

    if agent_state is None:
      # Registra el inicio del agente
      agent_state = StoryFlowAgentState(step=WorkflowStep.INITIAL_STORY_GENERATION)
      yield self._create_agent_state_event(ctx, agent_state)

    next_step = agent_state.step
    logger.info(f"[{self.name}] Starting story generation workflow.")

    # Paso 1. Generación Inicial de Historia
    if next_step <= WorkflowStep.INITIAL_STORY_GENERATION:
      logger.info(f"[{self.name}] Running StoryGenerator...")
      async for event in self.story_generator.run_async(ctx):
          yield event

      # Verifica si la historia fue generada antes de proceder
      if "current_story" not in ctx.session.state or not ctx.session.state[
          "current_story"
      ]:
          return  # Detiene el procesamiento si la historia inicial falló

    agent_state = StoryFlowAgentState(step=WorkflowStep.CRITIC_REVISER_LOOP)
    yield self._create_agent_state_event(ctx, agent_state)

    # Paso 2. Bucle Crítico-Revisor
    if next_step <= WorkflowStep.CRITIC_REVISER_LOOP:
      logger.info(f"[{self.name}] Running CriticReviserLoop...")
      async for event in self.loop_agent.run_async(ctx):
          logger.info(
              f"[{self.name}] Event from CriticReviserLoop: "
              f"{event.model_dump_json(indent=2, exclude_none=True)}"
          )
          yield event

    agent_state = StoryFlowAgentState(step=WorkflowStep.POST_PROCESSING)
    yield self._create_agent_state_event(ctx, agent_state)

    # Paso 3. Post-Procesamiento Secuencial (Gramática y Verificación de Tono)
    if next_step <= WorkflowStep.POST_PROCESSING:
      logger.info(f"[{self.name}] Running PostProcessing...")
      async for event in self.sequential_agent.run_async(ctx):
          logger.info(
              f"[{self.name}] Event from PostProcessing: "
              f"{event.model_dump_json(indent=2, exclude_none=True)}"
          )
          yield event

    agent_state = StoryFlowAgentState(step=WorkflowStep.CONDITIONAL_REGENERATION)
    yield self._create_agent_state_event(ctx, agent_state)

    # Paso 4. Lógica Condicional Basada en Tono
    if next_step <= WorkflowStep.CONDITIONAL_REGENERATION:
      tone_check_result = ctx.session.state.get("tone_check_result")
      if tone_check_result == "negative":
          logger.info(f"[{self.name}] Tone is negative. Regenerating story...")
          async for event in self.story_generator.run_async(ctx):
              logger.info(
                  f"[{self.name}] Event from StoryGenerator (Regen): "
                  f"{event.model_dump_json(indent=2, exclude_none=True)}"
              )
              yield event
      else:
          logger.info(f"[{self.name}] Tone is not negative. Keeping current story.")

    logger.info(f"[{self.name}] Workflow finished.")
    yield self._create_agent_state_event(ctx, end_of_agent=True)
```