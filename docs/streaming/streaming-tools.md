# Herramientas de Streaming

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.5.0</span><span class="lst-preview">Experimental</span>
</div>

Las herramientas de streaming permiten a las herramientas (funciones) transmitir resultados intermedios de vuelta a los agentes y los agentes pueden responder a esos resultados intermedios.
Por ejemplo, podemos usar herramientas de streaming para monitorear los cambios en el precio de las acciones y hacer que el agente reaccione a ellos. Otro ejemplo es que podemos hacer que el agente monitoree la transmisión de video, y cuando hay cambios en la transmisión de video, el agente puede reportar los cambios.

!!! info

    Esto solo está soportado en agentes/API de streaming (en vivo).

Para definir una herramienta de streaming, debes adherirte a lo siguiente:

1.  **Función Asíncrona:** La herramienta debe ser una función `async` de Python.
2.  **Tipo de Retorno AsyncGenerator:** La función debe estar tipada para retornar un `AsyncGenerator`. El primer parámetro de tipo de `AsyncGenerator` es el tipo de los datos que haces `yield` (por ejemplo, `str` para mensajes de texto, o un objeto personalizado para datos estructurados). El segundo parámetro de tipo es típicamente `None` si el generador no recibe valores mediante `send()`.


Soportamos dos tipos de herramientas de streaming:
- Tipo simple. Este es un tipo de herramientas de streaming que solo toma streams que no son de video/audio (los streams que alimentas a adk web o adk runner) como entrada.
- Herramientas de streaming de video. Esto solo funciona en streaming de video y el stream de video (los streams que alimentas a adk web o adk runner) se pasará a esta función.

Ahora definamos un agente que pueda monitorear cambios en el precio de las acciones y monitorear los cambios en la transmisión de video.

```python
import asyncio
from typing import AsyncGenerator

from google.adk.agents import LiveRequestQueue
from google.adk.agents.llm_agent import Agent
from google.adk.tools.function_tool import FunctionTool
from google.genai import Client
from google.genai import types as genai_types


async def monitor_stock_price(stock_symbol: str) -> AsyncGenerator[str, None]:
  """Esta función monitoreará el precio del stock_symbol dado de manera continua, streaming y asíncrona."""
  print(f"Start monitor stock price for {stock_symbol}!")

  # Simulemos el cambio de precio de las acciones.
  await asyncio.sleep(4)
  price_alert1 = f"the price for {stock_symbol} is 300"
  yield price_alert1
  print(price_alert1)

  await asyncio.sleep(4)
  price_alert1 = f"the price for {stock_symbol} is 400"
  yield price_alert1
  print(price_alert1)

  await asyncio.sleep(20)
  price_alert1 = f"the price for {stock_symbol} is 900"
  yield price_alert1
  print(price_alert1)

  await asyncio.sleep(20)
  price_alert1 = f"the price for {stock_symbol} is 500"
  yield price_alert1
  print(price_alert1)


# para streaming de video, `input_stream: LiveRequestQueue` es un parámetro clave requerido y reservado para que ADK pase los streams de video.
async def monitor_video_stream(
    input_stream: LiveRequestQueue,
) -> AsyncGenerator[str, None]:
  """Monitorea cuántas personas hay en los streams de video."""
  print("start monitor_video_stream!")
  client = Client(vertexai=False)
  prompt_text = (
      "Count the number of people in this image. Just respond with a numeric"
      " number."
  )
  last_count = None
  while True:
    last_valid_req = None
    print("Start monitoring loop")

    # usa este bucle para obtener las imágenes más recientes y descartar las antiguas
    while input_stream._queue.qsize() != 0:
      live_req = await input_stream.get()

      if live_req.blob is not None and live_req.blob.mime_type == "image/jpeg":
        last_valid_req = live_req

    # Si encontramos una imagen válida, procesarla
    if last_valid_req is not None:
      print("Processing the most recent frame from the queue")

      # Crear una parte de imagen usando los datos y el tipo mime del blob
      image_part = genai_types.Part.from_bytes(
          data=last_valid_req.blob.data, mime_type=last_valid_req.blob.mime_type
      )

      contents = genai_types.Content(
          role="user",
          parts=[image_part, genai_types.Part.from_text(prompt_text)],
      )

      # Llamar al modelo para generar contenido basado en la imagen y el prompt proporcionados
      response = client.models.generate_content(
          model="gemini-2.0-flash-exp",
          contents=contents,
          config=genai_types.GenerateContentConfig(
              system_instruction=(
                  "You are a helpful video analysis assistant. You can count"
                  " the number of people in this image or video. Just respond"
                  " with a numeric number."
              )
          ),
      )
      if not last_count:
        last_count = response.candidates[0].content.parts[0].text
      elif last_count != response.candidates[0].content.parts[0].text:
        last_count = response.candidates[0].content.parts[0].text
        yield response
        print("response:", response)

    # Esperar antes de verificar nuevas imágenes
    await asyncio.sleep(0.5)


# Usa exactamente esta función para ayudar a ADK a detener tus herramientas de streaming cuando se solicite.
# por ejemplo, si queremos detener `monitor_stock_price`, entonces el agente
# invocará esta función con stop_streaming(function_name=monitor_stock_price).
def stop_streaming(function_name: str):
  """Detiene el streaming

  Args:
    function_name: El nombre de la función de streaming a detener.
  """
  pass


root_agent = Agent(
    model="gemini-2.0-flash-exp",
    name="video_streaming_agent",
    instruction="""
      You are a monitoring agent. You can do video monitoring and stock price monitoring
      using the provided tools/functions.
      When users want to monitor a video stream,
      You can use monitor_video_stream function to do that. When monitor_video_stream
      returns the alert, you should tell the users.
      When users want to monitor a stock price, you can use monitor_stock_price.
      Don't ask too many questions. Don't be too talkative.
    """,
    tools=[
        monitor_video_stream,
        monitor_stock_price,
        FunctionTool(stop_streaming),
    ]
)
```

Aquí hay algunas consultas de ejemplo para probar:
- Ayúdame a monitorear el precio de las acciones de $XYZ.
- Ayúdame a monitorear cuántas personas hay en la transmisión de video.