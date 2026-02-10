# Construye tu Primer Equipo de Agentes Inteligentes: Un Bot del Clima Progresivo con ADK

<!-- Optional outer container for overall padding/spacing -->
<div style="padding: 10px 0;">

  <!-- Line 1: Open in Colab -->
  <!-- This div ensures the link takes up its own line and adds space below -->
  <div style="margin-bottom: 10px;">
    <a href="https://colab.research.google.com/github/google/blob/main/examples/python/tutorial/agent_team/adk_tutorial.ipynb" target="_blank" style="display: inline-flex; align-items: center; gap: 5px; text-decoration: none; color: #4285F4;">
      <img width="32px" src="https://www.gstatic.com/pantheon/images/bigquery/welcome_page/colab-logo.svg" alt="Google Colaboratory logo">
      <span>Abrir en Colab</span>
    </a>
  </div>

  <!-- Line 2: Share Links -->
  <!-- This div acts as a flex container for the "Share to" text and icons -->
  <div style="display: flex; align-items: center; gap: 10px; flex-wrap: wrap;">
    <!-- Share Text -->
    <span style="font-weight: bold;">Compartir en:</span>

    <!-- Social Media Links -->
    <a href="https://www.linkedin.com/sharing/share-offsite/?url=https%3A//github/google/blob/main/examples/python/tutorial/agent_team/adk_tutorial.ipynb" target="_blank" title="Share on LinkedIn">
      <img width="20px" src="https://upload.wikimedia.org/wikipedia/commons/8/81/LinkedIn_icon.svg" alt="LinkedIn logo" style="vertical-align: middle;">
    </a>
    <a href="https://bsky.app/intent/compose?text=https%3A//github/google/blob/main/examples/python/tutorial/agent_team/adk_tutorial.ipynb" target="_blank" title="Share on Bluesky">
      <img width="20px" src="https://upload.wikimedia.org/wikipedia/commons/7/7a/Bluesky_Logo.svg" alt="Bluesky logo" style="vertical-align: middle;">
    </a>
    <a href="https://twitter.com/intent/tweet?url=https%3A//github/google/blob/main/examples/python/tutorial/agent_team/adk_tutorial.ipynb" target="_blank" title="Share on X (Twitter)">
      <img width="20px" src="https://upload.wikimedia.org/wikipedia/commons/5/5a/X_icon_2.svg" alt="X logo" style="vertical-align: middle;">
    </a>
    <a href="https://reddit.com/submit?url=https%3A//github/google/blob/main/examples/python/tutorial/agent_team/adk_tutorial.ipynb" target="_blank" title="Share on Reddit">
      <img width="20px" src="https://redditinc.com/hubfs/Reddit%20Inc/Brand/Reddit_Logo.png" alt="Reddit logo" style="vertical-align: middle;">
    </a>
    <a href="https://www.facebook.com/sharer/sharer.php?u=https%3A//github/google/blob/main/examples/python/tutorial/agent_team/adk_tutorial.ipynb" target="_blank" title="Share on Facebook">
      <img width="20px" src="https://upload.wikimedia.org/wikipedia/commons/5/51/Facebook_f_logo_%282019%29.svg" alt="Facebook logo" style="vertical-align: middle;">
    </a>
  </div>

</div>

Este tutorial se extiende desde el [ejemplo de Inicio Rápido](https://google.github.io/get-started/quickstart/) para el [Kit de Desarrollo de Agentes](https://google.github.io/get-started/). Ahora, estás listo para profundizar y construir un **sistema multi-agente** más sofisticado.

Nos embarcaremos en la construcción de un **equipo de agentes Bot del Clima**, agregando progresivamente características avanzadas sobre una base simple. Comenzando con un solo agente que puede consultar el clima, agregaremos incrementalmente capacidades como:

*   Aprovechar diferentes modelos de IA (Gemini, GPT, Claude).
*   Diseñar sub-agentes especializados para tareas distintas (como saludos y despedidas).
*   Habilitar delegación inteligente entre agentes.
*   Dar memoria a los agentes usando estado de sesión persistente.
*   Implementar barreras de seguridad cruciales usando callbacks.

**¿Por qué un Equipo Bot del Clima?**

Este caso de uso, aunque aparentemente simple, proporciona un lienzo práctico y relacionable para explorar conceptos centrales de ADK esenciales para construir aplicaciones agénticas complejas del mundo real. Aprenderás cómo estructurar interacciones, gestionar estado, asegurar seguridad y orquestar múltiples "cerebros" de IA trabajando juntos.

**¿Qué es ADK de nuevo?**

Como recordatorio, ADK es un framework de Python diseñado para agilizar el desarrollo de aplicaciones potenciadas por Modelos de Lenguaje Grande (LLMs). Ofrece bloques de construcción robustos para crear agentes que pueden razonar, planificar, utilizar herramientas, interactuar dinámicamente con usuarios y colaborar efectivamente dentro de un equipo.

**En este tutorial avanzado, dominarás:**

*   ✅ **Definición y Uso de Herramientas:** Crear funciones Python (`tools`) que otorgan a los agentes habilidades específicas (como obtener datos) e instruir a los agentes sobre cómo usarlas efectivamente.
*   ✅ **Flexibilidad Multi-LLM:** Configurar agentes para utilizar varios LLMs líderes (Gemini, GPT-4o, Claude Sonnet) a través de la integración con LiteLLM, permitiéndote elegir el mejor modelo para cada tarea.
*   ✅ **Delegación y Colaboración de Agentes:** Diseñar sub-agentes especializados y habilitar el enrutamiento automático (`auto flow`) de solicitudes de usuario al agente más apropiado dentro de un equipo.
*   ✅ **Estado de Sesión para Memoria:** Utilizar `Session State` y `ToolContext` para permitir que los agentes recuerden información a través de turnos conversacionales, llevando a interacciones más contextuales.
*   ✅ **Barreras de Seguridad con Callbacks:** Implementar `before_model_callback` y `before_tool_callback` para inspeccionar, modificar o bloquear solicitudes/uso de herramientas basándose en reglas predefinidas, mejorando la seguridad y control de la aplicación.

**Expectativa del Estado Final:**

Al completar este tutorial, habrás construido un sistema funcional multi-agente Bot del Clima. Este sistema no solo proporcionará información del clima sino que también manejará cortesías conversacionales, recordará la última ciudad consultada y operará dentro de límites de seguridad definidos, todo orquestado usando ADK.

**Prerequisitos:**

*   ✅ **Comprensión sólida de programación Python.**
*   ✅ **Familiaridad con Modelos de Lenguaje Grande (LLMs), APIs y el concepto de agentes.**
*   ❗ **Crucial: Completar el(los) tutorial(es) de Inicio Rápido de ADK o conocimiento fundamental equivalente de los básicos de ADK (Agent, Runner, SessionService, uso básico de Tool).** Este tutorial se construye directamente sobre esos conceptos.
*   ✅ **Claves API** para los LLMs que pretendes usar (ej., Google AI Studio para Gemini, OpenAI Platform, Anthropic Console).


---

**Nota sobre el Entorno de Ejecución:**

Este tutorial está estructurado para entornos de notebook interactivos como Google Colab, Colab Enterprise o notebooks Jupyter. Por favor ten en cuenta lo siguiente:

*   **Ejecutar Código Async:** Los entornos de notebook manejan código asíncrono de manera diferente. Verás ejemplos usando `await` (adecuado cuando un event loop ya está ejecutándose, común en notebooks) o `asyncio.run()` (a menudo necesario cuando se ejecuta como un script `.py` independiente o en configuraciones específicas de notebook). Los bloques de código proporcionan guía para ambos escenarios.
*   **Configuración Manual de Runner/Session:** Los pasos involucran crear explícitamente instancias de `Runner` y `SessionService`. Este enfoque se muestra porque te da control de grano fino sobre el ciclo de vida de ejecución del agente, gestión de sesión y persistencia de estado.

**Alternativa: Usar Herramientas Integradas de ADK (Web UI / CLI / API Server)**

Si prefieres una configuración que maneje la gestión de runner y sesión automáticamente usando las herramientas estándar de ADK, puedes encontrar el código equivalente estructurado para ese propósito [aquí](https://github.com/google/tree/main/examples/python/tutorial/agent_team/adk-tutorial). Esa versión está diseñada para ejecutarse directamente con comandos como `adk web` (para una interfaz web), `adk run` (para interacción CLI) o `adk api_server` (para exponer una API). Por favor sigue las instrucciones del `README.md` proporcionadas en ese recurso alternativo.

---

**¿Listo para construir tu equipo de agentes? ¡Vamos a sumergirnos!**

> **Nota:** Este tutorial funciona con la versión 1.0.0 de adk y superior

```python
# @title Paso 0: Configuración e Instalación
# Instalar ADK y LiteLLM para soporte multi-modelo

!pip install google-adk -q
!pip install litellm -q

print("Instalación completada.")
```


```python
# @title Importar bibliotecas necesarias
import os
import asyncio
from google.adk.agents import Agent
from google.adk.models.lite_llm import LiteLlm # Para soporte multi-modelo
from google.adk.sessions import InMemorySessionService
from google.adk.runners import Runner
from google.genai import types # Para crear Content/Parts de mensaje

import warnings
# Ignorar todas las advertencias
warnings.filterwarnings("ignore")

import logging
logging.basicConfig(level=logging.ERROR)

print("Bibliotecas importadas.")
```


```python
# @title Configurar Claves API (¡Reemplaza con tus claves reales!)

# --- IMPORTANTE: Reemplaza los marcadores de posición con tus claves API reales ---

# Clave API de Gemini (Obtén de Google AI Studio: https://aistudio.google.com/app/apikey)
os.environ["GOOGLE_API_KEY"] = "YOUR_GOOGLE_API_KEY" # <--- REEMPLAZAR

# [Opcional]
# Clave API de OpenAI (Obtén de OpenAI Platform: https://platform.openai.com/api-keys)
os.environ['OPENAI_API_KEY'] = 'YOUR_OPENAI_API_KEY' # <--- REEMPLAZAR

# [Opcional]
# Clave API de Anthropic (Obtén de Anthropic Console: https://console.anthropic.com/settings/keys)
os.environ['ANTHROPIC_API_KEY'] = 'YOUR_ANTHROPIC_API_KEY' # <--- REEMPLAZAR

# --- Verificar Claves (Comprobación Opcional) ---
print("Claves API Configuradas:")
print(f"Clave API de Google configurada: {'Sí' if os.environ.get('GOOGLE_API_KEY') and os.environ['GOOGLE_API_KEY'] != 'YOUR_GOOGLE_API_KEY' else 'No (¡REEMPLAZAR MARCADOR DE POSICIÓN!)'}")
print(f"Clave API de OpenAI configurada: {'Sí' if os.environ.get('OPENAI_API_KEY') and os.environ['OPENAI_API_KEY'] != 'YOUR_OPENAI_API_KEY' else 'No (¡REEMPLAZAR MARCADOR DE POSICIÓN!)'}")
print(f"Clave API de Anthropic configurada: {'Sí' if os.environ.get('ANTHROPIC_API_KEY') and os.environ['ANTHROPIC_API_KEY'] != 'YOUR_ANTHROPIC_API_KEY' else 'No (¡REEMPLAZAR MARCADOR DE POSICIÓN!)'}")

# Configurar ADK para usar claves API directamente (no Vertex AI para esta configuración multi-modelo)
os.environ["GOOGLE_GENAI_USE_VERTEXAI"] = "False"


# @markdown **Nota de Seguridad:** Es mejor práctica gestionar las claves API de forma segura (ej., usando Colab Secrets o variables de entorno) en lugar de codificarlas directamente en el notebook. Reemplaza las cadenas de marcador de posición arriba.
```


```python
# --- Definir Constantes de Modelo para uso más fácil ---

# Más modelos soportados pueden referenciarse aquí: https://ai.google.dev/gemini-api/docs/models#model-variations
MODEL_GEMINI_2_5_FLASH = "gemini-2.5-flash"

# Más modelos soportados pueden referenciarse aquí: https://docs.litellm.ai/docs/providers/openai#openai-chat-completion-models
MODEL_GPT_4O = "openai/gpt-4.1" # También puedes probar: gpt-4.1-mini, gpt-4o etc.

# Más modelos soportados pueden referenciarse aquí: https://docs.litellm.ai/docs/providers/anthropic
MODEL_CLAUDE_SONNET = "anthropic/claude-sonnet-4-20250514" # También puedes probar: claude-opus-4-20250514 , claude-3-7-sonnet-20250219 etc

print("\nEntorno configurado.")
```

---

## Paso 1: Tu Primer Agente \- Búsqueda Básica del Clima

Comencemos construyendo el componente fundamental de nuestro Bot del Clima: un solo agente capaz de realizar una tarea específica – buscar información del clima. Esto involucra crear dos piezas centrales:

1. **Una Herramienta:** Una función Python que equipa al agente con la *habilidad* de obtener datos del clima.  
2. **Un Agente:** El "cerebro" de IA que entiende la solicitud del usuario, sabe que tiene una herramienta de clima y decide cuándo y cómo usarla.

---

**1\. Definir la Herramienta (`get_weather`)**

En ADK, las **Herramientas** son los bloques de construcción que dan a los agentes capacidades concretas más allá de solo generación de texto. Son típicamente funciones Python regulares que realizan acciones específicas, como llamar una API, consultar una base de datos o realizar cálculos.

Nuestra primera herramienta proporcionará un reporte de clima *simulado*. Esto nos permite enfocarnos en la estructura del agente sin necesitar claves API externas todavía. Más adelante, podrías fácilmente intercambiar esta función simulada con una que llame a un servicio de clima real.

**Concepto Clave: ¡Los Docstrings son Cruciales\!** El LLM del agente se apoya en gran medida en el **docstring** de la función para entender:

* *Qué* hace la herramienta.  
* *Cuándo* usarla.  
* *Qué argumentos* requiere (`city: str`).  
* *Qué información* devuelve.

**Mejor Práctica:** Escribe docstrings claros, descriptivos y precisos para tus herramientas. Esto es esencial para que el LLM use la herramienta correctamente.


```python
# @title Definir la Herramienta get_weather
def get_weather(city: str) -> dict:
    """Recupera el reporte del clima actual para una ciudad especificada.

    Args:
        city (str): El nombre de la ciudad (ej., "New York", "London", "Tokyo").

    Returns:
        dict: Un diccionario conteniendo la información del clima.
              Incluye una clave 'status' ('success' o 'error').
              Si 'success', incluye una clave 'report' con detalles del clima.
              Si 'error', incluye una clave 'error_message'.
    """
    print(f"--- Tool: get_weather llamada para ciudad: {city} ---") # Registrar ejecución de herramienta
    city_normalized = city.lower().replace(" ", "") # Normalización básica

    # Datos de clima simulados
    mock_weather_db = {
        "newyork": {"status": "success", "report": "The weather in New York is sunny with a temperature of 25°C."},
        "london": {"status": "success", "report": "It's cloudy in London with a temperature of 15°C."},
        "tokyo": {"status": "success", "report": "Tokyo is experiencing light rain and a temperature of 18°C."},
    }

    if city_normalized in mock_weather_db:
        return mock_weather_db[city_normalized]
    else:
        return {"status": "error", "error_message": f"Sorry, I don't have weather information for '{city}'."}

# Ejemplo de uso de herramienta (prueba opcional)
print(get_weather("New York"))
print(get_weather("Paris"))
```

---

**2\. Definir el Agente (`weather_agent`)**

Ahora, vamos a crear el **Agente** mismo. Un `Agent` en ADK orquesta la interacción entre el usuario, el LLM y las herramientas disponibles.

Lo configuramos con varios parámetros clave:

* `name`: Un identificador único para este agente (ej., "weather\_agent\_v1").  
* `model`: Especifica qué LLM usar (ej., `MODEL_GEMINI_2_5_FLASH`). Comenzaremos con un modelo Gemini específico.  
* `description`: Un resumen conciso del propósito general del agente. Esto se vuelve crucial más adelante cuando otros agentes necesitan decidir si delegar tareas a *este* agente.  
* `instruction`: Guía detallada para el LLM sobre cómo comportarse, su persona, sus objetivos y específicamente *cómo y cuándo* utilizar sus `tools` asignadas.  
* `tools`: Una lista conteniendo las funciones de herramienta Python reales que el agente está permitido usar (ej., `[get_weather]`).

**Mejor Práctica:** Proporciona prompts de `instruction` claros y específicos. Mientras más detalladas las instrucciones, mejor puede el LLM entender su rol y cómo usar sus herramientas efectivamente. Sé explícito sobre el manejo de errores si es necesario.

**Mejor Práctica:** Elige valores descriptivos para `name` y `description`. Estos son usados internamente por ADK y son vitales para características como delegación automática (cubierta más adelante).


```python
# @title Definir el Agente del Clima
# Usar una de las constantes de modelo definidas anteriormente
AGENT_MODEL = MODEL_GEMINI_2_5_FLASH # Comenzando con Gemini

weather_agent = Agent(
    name="weather_agent_v1",
    model=AGENT_MODEL, # Puede ser una cadena para Gemini o un objeto LiteLlm
    description="Proporciona información del clima para ciudades específicas.",
    instruction="Eres un asistente del clima útil. "
                "Cuando el usuario pregunte por el clima en una ciudad específica, "
                "usa la herramienta 'get_weather' para encontrar la información. "
                "Si la herramienta devuelve un error, informa al usuario cortésmente. "
                "Si la herramienta es exitosa, presenta el reporte del clima claramente.",
    tools=[get_weather], # Pasar la función directamente
)

print(f"Agente '{weather_agent.name}' creado usando modelo '{AGENT_MODEL}'.")
```

---

**3\. Configurar Runner y Session Service**

Para gestionar conversaciones y ejecutar el agente, necesitamos dos componentes más:

* `SessionService`: Responsable de gestionar el historial de conversación y estado para diferentes usuarios y sesiones. El `InMemorySessionService` es una implementación simple que almacena todo en memoria, adecuada para pruebas y aplicaciones simples. Mantiene el registro de los mensajes intercambiados. Exploraremos la persistencia de estado más en el Paso 4\.  
* `Runner`: El motor que orquesta el flujo de interacción. Toma la entrada del usuario, la enruta al agente apropiado, gestiona llamadas al LLM y herramientas basándose en la lógica del agente, maneja actualizaciones de sesión vía el `SessionService` y produce eventos representando el progreso de la interacción.


```python
# @title Configurar Session Service y Runner

# --- Gestión de Sesión ---
# Concepto Clave: SessionService almacena historial de conversación y estado.
# InMemorySessionService es almacenamiento simple, no persistente para este tutorial.
session_service = InMemorySessionService()

# Definir constantes para identificar el contexto de interacción
APP_NAME = "weather_tutorial_app"
USER_ID = "user_1"
SESSION_ID = "session_001" # Usando un ID fijo para simplicidad

# Crear la sesión específica donde ocurrirá la conversación
session = await session_service.create_session(
    app_name=APP_NAME,
    user_id=USER_ID,
    session_id=SESSION_ID
)
print(f"Sesión creada: App='{APP_NAME}', User='{USER_ID}', Session='{SESSION_ID}'")

# --- O ---

# Descomenta las siguientes líneas si ejecutas como un script Python estándar (archivo .py):

# async def init_session(app_name:str,user_id:str,session_id:str) -> InMemorySessionService:
#     session = await session_service.create_session(
#         app_name=app_name,
#         user_id=user_id,
#         session_id=session_id
#     )
#     print(f"Sesión creada: App='{app_name}', User='{user_id}', Session='{session_id}'")
#     return session
# 
# session = asyncio.run(init_session(APP_NAME,USER_ID,SESSION_ID))

# --- Runner ---
# Concepto Clave: Runner orquesta el bucle de ejecución del agente.
runner = Runner(
    agent=weather_agent, # El agente que queremos ejecutar
    app_name=APP_NAME,   # Asocia ejecuciones con nuestra app
    session_service=session_service # Usa nuestro gestor de sesión
)
print(f"Runner creado para agente '{runner.agent.name}'.")
```

---

**4\. Interactuar con el Agente**

Necesitamos una forma de enviar mensajes a nuestro agente y recibir sus respuestas. Dado que las llamadas LLM y ejecuciones de herramientas pueden tomar tiempo, el `Runner` de ADK opera de forma asíncrona.

Definiremos una función auxiliar `async` (`call_agent_async`) que:

1. Toma una cadena de consulta del usuario.  
2. La empaqueta en el formato `Content` de ADK.  
3. Llama a `runner.run_async`, proporcionando el contexto de usuario/sesión y el nuevo mensaje.  
4. Itera a través de los **Eventos** producidos por el runner. Los eventos representan pasos en la ejecución del agente (ej., llamada de herramienta solicitada, resultado de herramienta recibido, pensamiento intermedio del LLM, respuesta final).  
5. Identifica e imprime el evento de **respuesta final** usando `event.is_final_response()`.

**¿Por qué `async`?** Las interacciones con LLMs y potencialmente herramientas (como APIs externas) son operaciones vinculadas a I/O. Usar `asyncio` permite al programa manejar estas operaciones eficientemente sin bloquear la ejecución.


```python
# @title Definir Función de Interacción con el Agente

from google.genai import types # Para crear Content/Parts de mensaje

async def call_agent_async(query: str, runner, user_id, session_id):
  """Envía una consulta al agente e imprime la respuesta final."""
  print(f"\n>>> Consulta del Usuario: {query}")

  # Preparar el mensaje del usuario en formato ADK
  content = types.Content(role='user', parts=[types.Part(text=query)])

  final_response_text = "El agente no produjo una respuesta final." # Por defecto

  # Concepto Clave: run_async ejecuta la lógica del agente y produce Eventos.
  # Iteramos a través de eventos para encontrar la respuesta final.
  async for event in runner.run_async(user_id=user_id, session_id=session_id, new_message=content):
      # Puedes descomentar la línea abajo para ver *todos* los eventos durante la ejecución
      # print(f"  [Event] Author: {event.author}, Type: {type(event).__name__}, Final: {event.is_final_response()}, Content: {event.content}")

      # Concepto Clave: is_final_response() marca el mensaje de conclusión para el turno.
      if event.is_final_response():
          if event.content and event.content.parts:
             # Asumiendo respuesta de texto en la primera parte
             final_response_text = event.content.parts[0].text
          elif event.actions and event.actions.escalate: # Manejar posibles errores/escalaciones
             final_response_text = f"El agente escaló: {event.error_message or 'Sin mensaje específico.'}"
          # Agregar más verificaciones aquí si es necesario (ej., códigos de error específicos)
          break # Detener procesamiento de eventos una vez que se encuentra la respuesta final

  print(f"<<< Respuesta del Agente: {final_response_text}")
```

---

**5\. Ejecutar la Conversación**

Finalmente, probemos nuestra configuración enviando algunas consultas al agente. Envolvemos nuestras llamadas `async` en una función `async` principal y la ejecutamos usando `await`.

Observa la salida:

* Ve las consultas del usuario.  
* Nota los logs `--- Tool: get_weather called... ---` cuando el agente usa la herramienta.  
* Observa las respuestas finales del agente, incluyendo cómo maneja el caso donde los datos del clima no están disponibles (para París).


```python
# @title Ejecutar la Conversación Inicial

# Necesitamos una función async para await nuestro auxiliar de interacción
async def run_conversation():
    await call_agent_async("What is the weather like in London?",
                                       runner=runner,
                                       user_id=USER_ID,
                                       session_id=SESSION_ID)

    await call_agent_async("How about Paris?",
                                       runner=runner,
                                       user_id=USER_ID,
                                       session_id=SESSION_ID) # Esperando el mensaje de error de la herramienta

    await call_agent_async("Tell me the weather in New York",
                                       runner=runner,
                                       user_id=USER_ID,
                                       session_id=SESSION_ID)

# Ejecutar la conversación usando await en un contexto async (como Colab/Jupyter)
await run_conversation()

# --- O ---

# Descomenta las siguientes líneas si ejecutas como un script Python estándar (archivo .py):
# import asyncio
# if __name__ == "__main__":
#     try:
#         asyncio.run(run_conversation())
#     except Exception as e:
#         print(f"Ocurrió un error: {e}")
```

---

¡Felicitaciones\! Has construido e interactuado exitosamente con tu primer agente ADK. Entiende la solicitud del usuario, usa una herramienta para encontrar información y responde apropiadamente basándose en el resultado de la herramienta.

En el siguiente paso, exploraremos cómo cambiar fácilmente el Modelo de Lenguaje subyacente que potencia este agente.

## Paso 2: Yendo Multi-Modelo con LiteLLM [Opcional]

En el Paso 1, construimos un Agente del Clima funcional potenciado por un modelo Gemini específico. Aunque efectivo, las aplicaciones del mundo real a menudo se benefician de la flexibilidad de usar *diferentes* Modelos de Lenguaje Grande (LLMs). ¿Por qué?

*   **Rendimiento:** Algunos modelos sobresalen en tareas específicas (ej., codificación, razonamiento, escritura creativa).
*   **Costo:** Diferentes modelos tienen puntos de precio variados.
*   **Capacidades:** Los modelos ofrecen características diversas, tamaños de ventana de contexto y opciones de ajuste fino.
*   **Disponibilidad/Redundancia:** Tener alternativas asegura que tu aplicación permanezca funcional incluso si un proveedor experimenta problemas.

ADK hace que cambiar entre modelos sea sin costuras a través de su integración con la biblioteca [**LiteLLM**](https://github.com/BerriAI/litellm). LiteLLM actúa como una interfaz consistente para más de 100 LLMs diferentes.

**En este paso, vamos a:**

1.  Aprender cómo configurar un `Agent` de ADK para usar modelos de proveedores como OpenAI (GPT) y Anthropic (Claude) usando el wrapper `LiteLlm`.
2.  Definir, configurar (con sus propias sesiones y runners) y probar inmediatamente instancias de nuestro Agente del Clima, cada una respaldada por un LLM diferente.
3.  Interactuar con estos diferentes agentes para observar variaciones potenciales en sus respuestas, incluso cuando usan la misma herramienta subyacente.

---

**1\. Importar `LiteLlm`**

Importamos esto durante la configuración inicial (Paso 0), pero es el componente clave para soporte multi-modelo:


```python
# @title 1. Importar LiteLlm
from google.adk.models.lite_llm import LiteLlm
```

**2\. Definir y Probar Agentes Multi-Modelo**

En lugar de pasar solo una cadena de nombre de modelo (que por defecto usa modelos Gemini de Google), envolvemos la cadena identificadora del modelo deseado dentro de la clase `LiteLlm`.

*   **Concepto Clave: Wrapper `LiteLlm`:** La sintaxis `LiteLlm(model="provider/model_name")` le dice a ADK que enrute solicitudes para este agente a través de la biblioteca LiteLLM al proveedor de modelo especificado.

Asegúrate de haber configurado las claves API necesarias para OpenAI y Anthropic en el Paso 0. Usaremos la función `call_agent_async` (definida anteriormente, que ahora acepta `runner`, `user_id` y `session_id`) para interactuar con cada agente inmediatamente después de su configuración.

Cada bloque a continuación:

*   Definirá el agente usando un modelo LiteLLM específico (`MODEL_GPT_4O` o `MODEL_CLAUDE_SONNET`).
*   Creará un `InMemorySessionService` y sesión *nuevo, separado* específicamente para la ejecución de prueba de ese agente. Esto mantiene los historiales de conversación aislados para esta demostración.
*   Creará un `Runner` configurado para el agente específico y su servicio de sesión.
*   Llamará inmediatamente a `call_agent_async` para enviar una consulta y probar el agente.

**Mejor Práctica:** Usa constantes para nombres de modelos (como `MODEL_GPT_4O`, `MODEL_CLAUDE_SONNET` definidos en el Paso 0) para evitar errores tipográficos y hacer el código más fácil de gestionar.

**Manejo de Errores:** Envolvemos las definiciones de agente en bloques `try...except`. Esto previene que toda la celda de código falle si una clave API para un proveedor específico falta o es inválida, permitiendo que el tutorial proceda con los modelos que *están* configurados.

Primero, creemos y probemos el agente usando GPT-4o de OpenAI.


```python
# @title Definir y Probar Agente GPT

# Asegúrate de que la función 'get_weather' del Paso 1 esté definida en tu entorno.
# Asegúrate de que 'call_agent_async' esté definida de antes.

# --- Agente usando GPT-4o ---
weather_agent_gpt = None # Inicializar a None
runner_gpt = None      # Inicializar runner a None

try:
    weather_agent_gpt = Agent(
        name="weather_agent_gpt",
        # Cambio clave: Envolver el identificador del modelo LiteLLM
        model=LiteLlm(model=MODEL_GPT_4O),
        description="Proporciona información del clima (usando GPT-4o).",
        instruction="Eres un asistente del clima útil potenciado por GPT-4o. "
                    "Usa la herramienta 'get_weather' para solicitudes de clima de ciudad. "
                    "Presenta claramente reportes exitosos o mensajes de error corteses basados en el estado de salida de la herramienta.",
        tools=[get_weather], # Re-usar la misma herramienta
    )
    print(f"Agente '{weather_agent_gpt.name}' creado usando modelo '{MODEL_GPT_4O}'.")

    # InMemorySessionService es almacenamiento simple, no persistente para este tutorial.
    session_service_gpt = InMemorySessionService() # Crear un servicio dedicado

    # Definir constantes para identificar el contexto de interacción
    APP_NAME_GPT = "weather_tutorial_app_gpt" # Nombre de app único para esta prueba
    USER_ID_GPT = "user_1_gpt"
    SESSION_ID_GPT = "session_001_gpt" # Usando un ID fijo para simplicidad

    # Crear la sesión específica donde ocurrirá la conversación
    session_gpt = await session_service_gpt.create_session(
        app_name=APP_NAME_GPT,
        user_id=USER_ID_GPT,
        session_id=SESSION_ID_GPT
    )
    print(f"Sesión creada: App='{APP_NAME_GPT}', User='{USER_ID_GPT}', Session='{SESSION_ID_GPT}'")

    # Crear un runner específico para este agente y su servicio de sesión
    runner_gpt = Runner(
        agent=weather_agent_gpt,
        app_name=APP_NAME_GPT,       # Usar el nombre de app específico
        session_service=session_service_gpt # Usar el servicio de sesión específico
        )
    print(f"Runner creado para agente '{runner_gpt.agent.name}'.")

    # --- Probar el Agente GPT ---
    print("\n--- Probando Agente GPT ---")
    # Asegurar que call_agent_async use el runner, user_id, session_id correctos
    await call_agent_async(query = "What's the weather in Tokyo?",
                           runner=runner_gpt,
                           user_id=USER_ID_GPT,
                           session_id=SESSION_ID_GPT)
    # --- O ---

    # Descomenta las siguientes líneas si ejecutas como un script Python estándar (archivo .py):
    # import asyncio
    # if __name__ == "__main__":
    #     try:
    #         asyncio.run(call_agent_async(query = "What's the weather in Tokyo?",
    #                      runner=runner_gpt,
    #                       user_id=USER_ID_GPT,
    #                       session_id=SESSION_ID_GPT)
    #     except Exception as e:
    #         print(f"Ocurrió un error: {e}")

except Exception as e:
    print(f"❌ No se pudo crear o ejecutar el agente GPT '{MODEL_GPT_4O}'. Verifica la Clave API y nombre del modelo. Error: {e}")

```

A continuación, haremos lo mismo para Claude Sonnet de Anthropic.


```python
# @title Definir y Probar Agente Claude

# Asegúrate de que la función 'get_weather' del Paso 1 esté definida en tu entorno.
# Asegúrate de que 'call_agent_async' esté definida de antes.

# --- Agente usando Claude Sonnet ---
weather_agent_claude = None # Inicializar a None
runner_claude = None      # Inicializar runner a None

try:
    weather_agent_claude = Agent(
        name="weather_agent_claude",
        # Cambio clave: Envolver el identificador del modelo LiteLLM
        model=LiteLlm(model=MODEL_CLAUDE_SONNET),
        description="Proporciona información del clima (usando Claude Sonnet).",
        instruction="Eres un asistente del clima útil potenciado por Claude Sonnet. "
                    "Usa la herramienta 'get_weather' para solicitudes de clima de ciudad. "
                    "Analiza la salida de diccionario de la herramienta ('status', 'report'/'error_message'). "
                    "Presenta claramente reportes exitosos o mensajes de error corteses.",
        tools=[get_weather], # Re-usar la misma herramienta
    )
    print(f"Agente '{weather_agent_claude.name}' creado usando modelo '{MODEL_CLAUDE_SONNET}'.")

    # InMemorySessionService es almacenamiento simple, no persistente para este tutorial.
    session_service_claude = InMemorySessionService() # Crear un servicio dedicado

    # Definir constantes para identificar el contexto de interacción
    APP_NAME_CLAUDE = "weather_tutorial_app_claude" # Nombre de app único
    USER_ID_CLAUDE = "user_1_claude"
    SESSION_ID_CLAUDE = "session_001_claude" # Usando un ID fijo para simplicidad

    # Crear la sesión específica donde ocurrirá la conversación
    session_claude = await session_service_claude.create_session(
        app_name=APP_NAME_CLAUDE,
        user_id=USER_ID_CLAUDE,
        session_id=SESSION_ID_CLAUDE
    )
    print(f"Sesión creada: App='{APP_NAME_CLAUDE}', User='{USER_ID_CLAUDE}', Session='{SESSION_ID_CLAUDE}'")

    # Crear un runner específico para este agente y su servicio de sesión
    runner_claude = Runner(
        agent=weather_agent_claude,
        app_name=APP_NAME_CLAUDE,       # Usar el nombre de app específico
        session_service=session_service_claude # Usar el servicio de sesión específico
        )
    print(f"Runner creado para agente '{runner_claude.agent.name}'.")

    # --- Probar el Agente Claude ---
    print("\n--- Probando Agente Claude ---")
    # Asegurar que call_agent_async use el runner, user_id, session_id correctos
    await call_agent_async(query = "Weather in London please.",
                           runner=runner_claude,
                           user_id=USER_ID_CLAUDE,
                           session_id=SESSION_ID_CLAUDE)

    # --- O ---

    # Descomenta las siguientes líneas si ejecutas como un script Python estándar (archivo .py):
    # import asyncio
    # if __name__ == "__main__":
    #     try:
    #         asyncio.run(call_agent_async(query = "Weather in London please.",
    #                      runner=runner_claude,
    #                       user_id=USER_ID_CLAUDE,
    #                       session_id=SESSION_ID_CLAUDE)
    #     except Exception as e:
    #         print(f"Ocurrió un error: {e}")


except Exception as e:
    print(f"❌ No se pudo crear o ejecutar el agente Claude '{MODEL_CLAUDE_SONNET}'. Verifica la Clave API y nombre del modelo. Error: {e}")
```

Observa cuidadosamente la salida de ambos bloques de código. Deberías ver:

1.  Cada agente (`weather_agent_gpt`, `weather_agent_claude`) es creado exitosamente (si las claves API son válidas).
2.  Una sesión y runner dedicados se configuran para cada uno.
3.  Cada agente identifica correctamente la necesidad de usar la herramienta `get_weather` al procesar la consulta (verás el log `--- Tool: get_weather called... ---`).
4.  La *lógica de herramienta subyacente* permanece idéntica, siempre devolviendo nuestros datos simulados.
5.  Sin embargo, la **respuesta textual final** generada por cada agente podría diferir ligeramente en fraseo, tono o formato. Esto es porque el prompt de instrucción es interpretado y ejecutado por diferentes LLMs (GPT-4o vs. Claude Sonnet).

Este paso demuestra el poder y flexibilidad que ADK + LiteLLM proporcionan. Puedes experimentar fácilmente y desplegar agentes usando varios LLMs mientras mantienes tu lógica de aplicación central (herramientas, estructura fundamental de agente) consistente.

En el siguiente paso, iremos más allá de un solo agente y construiremos un pequeño equipo donde los agentes pueden delegar tareas entre ellos!

---

## Paso 3: Construyendo un Equipo de Agentes \- Delegación para Saludos y Despedidas

En los Pasos 1 y 2, construimos y experimentamos con un solo agente enfocado únicamente en búsquedas de clima. Aunque efectivo para su tarea específica, las aplicaciones del mundo real a menudo involucran manejar una variedad más amplia de interacciones de usuario. *Podríamos* seguir agregando más herramientas e instrucciones complejas a nuestro único agente de clima, pero esto puede volverse rápidamente inmanejable y menos eficiente.

Un enfoque más robusto es construir un **Equipo de Agentes**. Esto involucra:

1. Crear múltiples **agentes especializados**, cada uno diseñado para una capacidad específica (ej., uno para clima, uno para saludos, uno para cálculos).  
2. Designar un **agente raíz** (u orquestador) que reciba la solicitud inicial del usuario.  
3. Habilitar al agente raíz para **delegar** la solicitud al sub-agente especializado más apropiado basándose en la intención del usuario.

**¿Por qué construir un Equipo de Agentes?**

* **Modularidad:** Más fácil de desarrollar, probar y mantener agentes individuales.  
* **Especialización:** Cada agente puede ser ajustado finamente (instrucciones, elección de modelo) para su tarea específica.  
* **Escalabilidad:** Más simple agregar nuevas capacidades agregando nuevos agentes.  
* **Eficiencia:** Permite usar modelos potencialmente más simples/baratos para tareas más simples (como saludos).

**En este paso, vamos a:**

1. Definir herramientas simples para manejar saludos (`say_hello`) y despedidas (`say_goodbye`).  
2. Crear dos nuevos sub-agentes especializados: `greeting_agent` y `farewell_agent`.  
3. Actualizar nuestro agente principal del clima (`weather_agent_v2`) para actuar como el **agente raíz**.  
4. Configurar el agente raíz con sus sub-agentes, habilitando **delegación automática**.  
5. Probar el flujo de delegación enviando diferentes tipos de solicitudes al agente raíz.

---

**1\. Definir Herramientas para Sub-Agentes**

Primero, creemos las funciones Python simples que servirán como herramientas para nuestros nuevos agentes especialistas. Recuerda, los docstrings claros son vitales para los agentes que los usarán.


```python
# @title Definir Herramientas para Agentes de Saludo y Despedida
from typing import Optional # Asegúrate de importar Optional

# Asegurar que 'get_weather' del Paso 1 esté disponible si ejecutas este paso independientemente.
# def get_weather(city: str) -> dict: ... (del Paso 1)

def say_hello(name: Optional[str] = None) -> str:
    """Proporciona un saludo simple. Si se proporciona un nombre, se usará.

    Args:
        name (str, optional): El nombre de la persona a saludar. Por defecto es un saludo genérico si no se proporciona.

    Returns:
        str: Un mensaje de saludo amigable.
    """
    if name:
        greeting = f"Hello, {name}!"
        print(f"--- Tool: say_hello llamada con nombre: {name} ---")
    else:
        greeting = "Hello there!" # Saludo por defecto si name es None o no se pasa explícitamente
        print(f"--- Tool: say_hello llamada sin un nombre específico (name_arg_value: {name}) ---")
    return greeting

def say_goodbye() -> str:
    """Proporciona un mensaje de despedida simple para concluir la conversación."""
    print(f"--- Tool: say_goodbye llamada ---")
    return "Goodbye! Have a great day."

print("Herramientas de Saludo y Despedida definidas.")

# Auto-prueba opcional
print(say_hello("Alice"))
print(say_hello()) # Probar sin argumento (debería usar por defecto "Hello there!")
print(say_hello(name=None)) # Probar con name explícitamente como None (debería usar por defecto "Hello there!")
```

---

**2\. Definir los Sub-Agentes (Saludo y Despedida)**

Ahora, crea las instancias de `Agent` para nuestros especialistas. Nota su `instruction` altamente enfocada y, críticamente, su `description` clara. La `description` es la información primaria que el *agente raíz* usa para decidir *cuándo* delegar a estos sub-agentes.

**Mejor Práctica:** Los campos `description` de los sub-agentes deben resumir precisa y concisamente su capacidad específica. Esto es crucial para una delegación automática efectiva.

**Mejor Práctica:** Los campos `instruction` de los sub-agentes deben ser adaptados a su alcance limitado, diciéndoles exactamente qué hacer y *qué no* hacer (ej., "Tu *única* tarea es...").


```python
# @title Definir Sub-Agentes de Saludo y Despedida

# Si quieres usar modelos distintos a Gemini, Asegúrate de que LiteLlm esté importado y las claves API estén configuradas (del Paso 0/2)
# from google.adk.models.lite_llm import LiteLlm
# MODEL_GPT_4O, MODEL_CLAUDE_SONNET etc. deberían estar definidos
# O sino, continúa usando: model = MODEL_GEMINI_2_5_FLASH

# --- Agente de Saludo ---
greeting_agent = None
try:
    greeting_agent = Agent(
        # Usando un modelo potencialmente diferente/más barato para una tarea simple
        model = MODEL_GEMINI_2_5_FLASH,
        # model=LiteLlm(model=MODEL_GPT_4O), # Si quisieras experimentar con otros modelos
        name="greeting_agent",
        instruction="Eres el Agente de Saludo. Tu ÚNICA tarea es proporcionar un saludo amigable al usuario. "
                    "Usa la herramienta 'say_hello' para generar el saludo. "
                    "Si el usuario proporciona su nombre, asegúrate de pasarlo a la herramienta. "
                    "No participes en ninguna otra conversación o tarea.",
        description="Maneja saludos simples y holas usando la herramienta 'say_hello'.", # Crucial para delegación
        tools=[say_hello],
    )
    print(f"✅ Agente '{greeting_agent.name}' creado usando modelo '{greeting_agent.model}'.")
except Exception as e:
    print(f"❌ No se pudo crear agente de Saludo. Verifica Clave API ({greeting_agent.model}). Error: {e}")

# --- Agente de Despedida ---
farewell_agent = None
try:
    farewell_agent = Agent(
        # Puede usar el mismo o un modelo diferente
        model = MODEL_GEMINI_2_5_FLASH,
        # model=LiteLlm(model=MODEL_GPT_4O), # Si quisieras experimentar con otros modelos
        name="farewell_agent",
        instruction="Eres el Agente de Despedida. Tu ÚNICA tarea es proporcionar un mensaje de adiós cortés. "
                    "Usa la herramienta 'say_goodbye' cuando el usuario indique que se está yendo o terminando la conversación "
                    "(ej., usando palabras como 'bye', 'goodbye', 'thanks bye', 'see you'). "
                    "No realices ninguna otra acción.",
        description="Maneja despedidas simples y adioses usando la herramienta 'say_goodbye'.", # Crucial para delegación
        tools=[say_goodbye],
    )
    print(f"✅ Agente '{farewell_agent.name}' creado usando modelo '{farewell_agent.model}'.")
except Exception as e:
    print(f"❌ No se pudo crear agente de Despedida. Verifica Clave API ({farewell_agent.model}). Error: {e}")
```

---

**3\. Definir el Agente Raíz (Agente del Clima v2) con Sub-Agentes**

Ahora, actualizamos nuestro `weather_agent`. Los cambios clave son:

* Agregar el parámetro `sub_agents`: Pasamos una lista conteniendo las instancias de `greeting_agent` y `farewell_agent` que acabamos de crear.  
* Actualizar la `instruction`: Le decimos explícitamente al agente raíz *sobre* sus sub-agentes y *cuándo* debería delegar tareas a ellos.

**Concepto Clave: Delegación Automática (Auto Flow)** Al proporcionar la lista `sub_agents`, ADK habilita la delegación automática. Cuando el agente raíz recibe una consulta de usuario, su LLM considera no solo sus propias instrucciones y herramientas sino también la `description` de cada sub-agente. Si el LLM determina que una consulta se alinea mejor con la capacidad descrita de un sub-agente (ej., "Maneja saludos simples"), generará automáticamente una acción interna especial para *transferir control* a ese sub-agente para ese turno. El sub-agente entonces procesa la consulta usando su propio modelo, instrucciones y herramientas.

**Mejor Práctica:** Asegura que las instrucciones del agente raíz guíen claramente sus decisiones de delegación. Menciona los sub-agentes por nombre y describe las condiciones bajo las cuales la delegación debería ocurrir.


```python
# @title Definir el Agente Raíz con Sub-Agentes

# Asegurar que los sub-agentes fueron creados exitosamente antes de definir el agente raíz.
# También asegurar que la herramienta original 'get_weather' esté definida.
root_agent = None
runner_root = None # Inicializar runner

if greeting_agent and farewell_agent and 'get_weather' in globals():
    # Usemos un modelo Gemini capaz para que el agente raíz maneje orquestación
    root_agent_model = MODEL_GEMINI_2_5_FLASH

    weather_agent_team = Agent(
        name="weather_agent_v2", # Darle un nombre de nueva versión
        model=root_agent_model,
        description="El agente coordinador principal. Maneja solicitudes de clima y delega saludos/despedidas a especialistas.",
        instruction="Eres el Agente del Clima principal coordinando un equipo. Tu responsabilidad primaria es proporcionar información del clima. "
                    "Usa la herramienta 'get_weather' SOLO para solicitudes de clima específicas (ej., 'clima en Londres'). "
                    "Tienes sub-agentes especializados: "
                    "1. 'greeting_agent': Maneja saludos simples como 'Hola', 'Hello'. Delega a él para estos. "
                    "2. 'farewell_agent': Maneja despedidas simples como 'Bye', 'Adiós'. Delega a él para estos. "
                    "Analiza la consulta del usuario. Si es un saludo, delega a 'greeting_agent'. Si es una despedida, delega a 'farewell_agent'. "
                    "Si es una solicitud de clima, manéjala tú mismo usando 'get_weather'. "
                    "Para cualquier otra cosa, responde apropiadamente o declara que no puedes manejarlo.",
        tools=[get_weather], # El agente raíz aún necesita la herramienta de clima para su tarea central
        # Cambio clave: ¡Enlazar los sub-agentes aquí!
        sub_agents=[greeting_agent, farewell_agent]
    )
    print(f"✅ Agente Raíz '{weather_agent_team.name}' creado usando modelo '{root_agent_model}' con sub-agentes: {[sa.name for sa in weather_agent_team.sub_agents]}")

else:
    print("❌ No se puede crear agente raíz porque uno o más sub-agentes fallaron en inicializar o la herramienta 'get_weather' falta.")
    if not greeting_agent: print(" - Falta Agente de Saludo.")
    if not farewell_agent: print(" - Falta Agente de Despedida.")
    if 'get_weather' not in globals(): print(" - Falta función get_weather.")


```

---

**4\. Interactuar con el Equipo de Agentes**

Ahora que hemos definido nuestro agente raíz (`weather_agent_team` - *Nota: Asegúrate de que este nombre de variable coincida con el definido en el bloque de código anterior, probablemente `# @title Definir el Agente Raíz con Sub-Agentes`, que podría haberlo nombrado `root_agent`*) con sus sub-agentes especializados, probemos el mecanismo de delegación.

El siguiente bloque de código:

1.  Definirá una función `async` `run_team_conversation`.
2.  Dentro de esta función, creará un `InMemorySessionService` *nuevo, dedicado* y una sesión específica (`session_001_agent_team`) solo para esta ejecución de prueba. Esto aísla el historial de conversación para probar la dinámica del equipo.
3.  Creará un `Runner` (`runner_agent_team`) configurado para usar nuestro `weather_agent_team` (el agente raíz) y el servicio de sesión dedicado.
4.  Usará nuestra función actualizada `call_agent_async` para enviar diferentes tipos de consultas (saludo, solicitud de clima, despedida) al `runner_agent_team`. Pasaremos explícitamente el runner, ID de usuario e ID de sesión para esta prueba específica.
5.  Ejecutará inmediatamente la función `run_team_conversation`.

Esperamos el siguiente flujo:

1.  La consulta "Hello there!" va a `runner_agent_team`.
2.  El agente raíz (`weather_agent_team`) la recibe y, basándose en sus instrucciones y la `description` del `greeting_agent`, delega la tarea.
3.  `greeting_agent` maneja la consulta, llama a su herramienta `say_hello` y genera la respuesta.
4.  La consulta "What is the weather in New York?" *no* es delegada y es manejada directamente por el agente raíz usando su herramienta `get_weather`.
5.  La consulta "Thanks, bye!" es delegada al `farewell_agent`, que usa su herramienta `say_goodbye`.




```python
# @title Interactuar con el Equipo de Agentes
import asyncio # Asegurar que asyncio esté importado

# Asegurar que el agente raíz (ej., 'weather_agent_team' o 'root_agent' de la celda anterior) esté definido.
# Asegurar que la función call_agent_async esté definida.

# Verificar si la variable del agente raíz existe antes de definir la función de conversación
root_agent_var_name = 'root_agent' # Nombre por defecto de la guía del Paso 3
if 'weather_agent_team' in globals(): # Verificar si el usuario usó este nombre en su lugar
    root_agent_var_name = 'weather_agent_team'
elif 'root_agent' not in globals():
    print("⚠️ Agente raíz ('root_agent' o 'weather_agent_team') no encontrado. No se puede definir run_team_conversation.")
    # Asignar un valor ficticio para prevenir NameError más tarde si el bloque de código se ejecuta de todos modos
    root_agent = None # O configurar una bandera para prevenir ejecución

# Solo definir y ejecutar si el agente raíz existe
if root_agent_var_name in globals() and globals()[root_agent_var_name]:
    # Definir la función async principal para la lógica de conversación.
    # Las palabras clave 'await' DENTRO de esta función son necesarias para operaciones async.
    async def run_team_conversation():
        print("\n--- Probando Delegación del Equipo de Agentes ---")
        session_service = InMemorySessionService()
        APP_NAME = "weather_tutorial_agent_team"
        USER_ID = "user_1_agent_team"
        SESSION_ID = "session_001_agent_team"
        session = await session_service.create_session(
            app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID
        )
        print(f"Sesión creada: App='{APP_NAME}', User='{USER_ID}', Session='{SESSION_ID}'")

        actual_root_agent = globals()[root_agent_var_name]
        runner_agent_team = Runner( # O usar InMemoryRunner
            agent=actual_root_agent,
            app_name=APP_NAME,
            session_service=session_service
        )
        print(f"Runner creado para agente '{actual_root_agent.name}'.")

        # --- Interacciones usando await (correcto dentro de async def) ---
        await call_agent_async(query = "Hello there!",