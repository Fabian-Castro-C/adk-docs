# Comprimir el contexto del agente para mejorar el rendimiento

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v1.16.0</span>
</div>

A medida que un agente ADK se ejecuta, recopila información de *contexto*, incluyendo instrucciones del usuario, datos recuperados, respuestas de herramientas y contenido generado. A medida que el tamaño de estos datos de contexto crece, los tiempos de procesamiento del agente también suelen aumentar. Cada vez más datos se envían al modelo de IA generativa utilizado por el agente, aumentando el tiempo de procesamiento y ralentizando las respuestas. La característica de Compactación de Contexto del ADK está diseñada para reducir el tamaño del contexto mientras un agente se ejecuta, resumiendo las partes más antiguas del historial de eventos del flujo de trabajo del agente.

La característica de Compactación de Contexto utiliza un enfoque de *ventana deslizante* para recopilar y resumir datos de eventos del flujo de trabajo del agente dentro de una [Session](/sessions/session/). Cuando configuras esta característica en tu agente, resume los datos de eventos más antiguos una vez que alcanza un umbral de un número específico de eventos de flujo de trabajo, o invocaciones, dentro de la sesión actual.

## Configurar la compactación de contexto

Agrega la compactación de contexto al flujo de trabajo de tu agente añadiendo una configuración de Compactación de Eventos al objeto App de tu flujo de trabajo. Como parte de la configuración, debes especificar un intervalo de compactación y un tamaño de superposición, como se muestra en el siguiente código de ejemplo:

```python
from google.adk.apps.app import App
from google.adk.apps.app import EventsCompactionConfig

app = App(
    name='my-agent',
    root_agent=root_agent,
    events_compaction_config=EventsCompactionConfig(
        compaction_interval=3,  # Activar la compactación cada 3 invocaciones nuevas.
        overlap_size=1          # Incluir la última invocación de la ventana anterior.
    ),
)
```

Una vez configurado, el `Runner` del ADK maneja el proceso de compactación en segundo plano cada vez que la sesión alcanza el intervalo.

## Ejemplo de compactación de contexto

Si estableces `compaction_interval` en 3 y `overlap_size` en 1, los datos de eventos se comprimen al completarse los eventos 3, 6, 9, y así sucesivamente. La configuración de superposición aumenta el tamaño de la segunda compresión resumida, y de cada resumen posterior, como se muestra en la Figura 1.

![Context compaction example illustration](/assets/context-compaction.svg)
**Figura 1.** Ilustración de la configuración de compactación de eventos con un intervalo de 3 y una superposición de 1.

Con esta configuración de ejemplo, las tareas de compresión de contexto ocurren de la siguiente manera:

1.  **El evento 3 se completa**: Los 3 eventos se comprimen en un resumen
1.  **El evento 6 se completa**: Los eventos 3 a 6 se comprimen, incluyendo la superposición de 1 evento anterior
1.  **El evento 9 se completa**: Los eventos 6 a 9 se comprimen, incluyendo la superposición de 1 evento anterior

## Configuraciones

Las configuraciones para esta característica controlan con qué frecuencia se comprimen los datos de eventos y cuántos datos se retienen mientras se ejecuta el flujo de trabajo del agente. Opcionalmente, puedes configurar un objeto compactador

*   **`compaction_interval`**: Establece el número de eventos completados que activa la compactación de los datos de eventos anteriores.
*   **`overlap_size`**: Establece cuántos de los eventos previamente compactados se incluyen en un conjunto de contexto recién compactado.
*   **`summarizer`**: (Opcional) Define un objeto resumidor incluyendo un modelo de IA específico para usar en la resumición. Para más información, consulta [Definir un Resumidor](#define-summarizer).

### Definir un Resumidor {#define-summarizer}
Puedes personalizar el proceso de compresión de contexto definiendo un resumidor. La clase LlmEventSummarizer te permite especificar un modelo particular para la resumición. El siguiente ejemplo de código demuestra cómo definir y configurar un resumidor personalizado:

```python
from google.adk.apps.app import App, EventsCompactionConfig
from google.adk.apps.llm_event_summarizer import LlmEventSummarizer
from google.adk.models import Gemini

# Definir el modelo de IA a utilizar para la resumición:
summarization_llm = Gemini(model="gemini-2.5-flash")

# Crear el resumidor con el modelo personalizado:
my_summarizer = LlmEventSummarizer(llm=summarization_llm)

# Configurar la App con el resumidor personalizado y las configuraciones de compactación:
app = App(
    name='my-agent',
    root_agent=root_agent,
    events_compaction_config=EventsCompactionConfig(
        compaction_interval=3,
        overlap_size=1,
        summarizer=my_summarizer,
    ),
)
```

Puedes refinar aún más la operación del `SlidingWindowCompactor` modificando su clase resumidora `LlmEventSummarizer`, incluyendo cambiar la configuración `prompt_template` de esa clase. Para más detalles, consulta el [código de `LlmEventSummarizer`](https://github.com/google/adk-python/blob/main/src/google/adk/apps/llm_event_summarizer.py#L60).