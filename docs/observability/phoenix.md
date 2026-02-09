# Observabilidad de Agentes con Phoenix

[Phoenix](https://arize.com/docs/phoenix) es una plataforma de observabilidad de código abierto y auto-hospedada para monitorear, depurar y mejorar aplicaciones LLM y Agentes de IA a escala. Proporciona capacidades completas de rastreo y evaluación para tus aplicaciones de Google ADK. Para comenzar, regístrate para obtener una [cuenta gratuita](https://phoenix.arize.com/).


## Descripción general

Phoenix puede recopilar automáticamente trazas de Google ADK usando [instrumentación OpenInference](https://github.com/Arize-ai/openinference/tree/main/python/instrumentation/openinference-instrumentation-google-adk), permitiéndote:

- **Rastrear interacciones de agentes** - Captura automáticamente cada ejecución de agente, llamada a herramientas, solicitud de modelo y respuesta con contexto completo y metadatos
- **Evaluar rendimiento** - Evalúa el comportamiento del agente usando evaluadores personalizados o preconstruidos y ejecuta experimentos para probar configuraciones de agentes
- **Depurar problemas** - Analiza trazas detalladas para identificar rápidamente cuellos de botella, llamadas a herramientas fallidas y comportamiento inesperado del agente
- **Control auto-hospedado** - Mantén tus datos en tu propia infraestructura

## Instalación

### 1. Instalar Paquetes Requeridos { #install-required-packages }

```bash
pip install openinference-instrumentation-google-adk google-adk arize-phoenix-otel
```

## Configuración

### 1. Iniciar Phoenix { #launch-phoenix }

Estas instrucciones te muestran cómo usar Phoenix Cloud. También puedes [iniciar Phoenix](https://arize.com/docs/phoenix/integrations/llm-providers/google-gen-ai/google-adk-tracing) en un notebook, desde tu terminal, o auto-hospedarlo usando un contenedor.

1. Regístrate para obtener una [cuenta gratuita de Phoenix](https://phoenix.arize.com/).
2. Desde la página de Configuración de tu nuevo Phoenix Space, crea tu clave API
3. Copia tu endpoint que debería verse como: https://app.phoenix.arize.com/s/[tu-nombre-de-espacio]

**Configura tu endpoint de Phoenix y Clave API:**

```python
import os

os.environ["PHOENIX_API_KEY"] = "AGREGA TU CLAVE API DE PHOENIX"
os.environ["PHOENIX_COLLECTOR_ENDPOINT"] = "AGREGA TU ENDPOINT COLLECTOR DE PHOENIX"

# Si creaste tu instancia de Phoenix Cloud antes del 24 de junio de 2025, configura la clave API como un encabezado:
# os.environ["PHOENIX_CLIENT_HEADERS"] = f"api_key={os.getenv('PHOENIX_API_KEY')}"
```

### 2.  Conecta tu aplicación a Phoenix { #connect-your-application-to-phoenix }

```python
from phoenix.otel import register

# Configurar el rastreador de Phoenix
tracer_provider = register(
    project_name="my-llm-app",  # Por defecto es 'default'
    auto_instrument=True        # Auto-instrumenta tu app basándose en las dependencias OI instaladas
)
```

## Observar

Ahora que tienes el rastreo configurado, todas las solicitudes del SDK de Google ADK serán transmitidas a Phoenix para observabilidad y evaluación.

```python
import nest_asyncio
nest_asyncio.apply()

from google.adk.agents import Agent
from google.adk.runners import InMemoryRunner
from google.genai import types

# Definir una función de herramienta
def get_weather(city: str) -> dict:
    """Recupera el reporte de clima actual para una ciudad especificada.

    Args:
        city (str): El nombre de la ciudad para la cual recuperar el reporte de clima.

    Returns:
        dict: estado y resultado o mensaje de error.
    """
    if city.lower() == "new york":
        return {
            "status": "success",
            "report": (
                "The weather in New York is sunny with a temperature of 25 degrees"
                " Celsius (77 degrees Fahrenheit)."
            ),
        }
    else:
        return {
            "status": "error",
            "error_message": f"Weather information for '{city}' is not available.",
        }

# Crear un agente con herramientas
agent = Agent(
    name="weather_agent",
    model="gemini-2.0-flash-exp",
    description="Agent to answer questions using weather tools.",
    instruction="You must use the available tools to find an answer.",
    tools=[get_weather]
)

app_name = "weather_app"
user_id = "test_user"
session_id = "test_session"
runner = InMemoryRunner(agent=agent, app_name=app_name)
session_service = runner.session_service

await session_service.create_session(
    app_name=app_name,
    user_id=user_id,
    session_id=session_id
)

# Ejecutar el agente (todas las interacciones serán rastreadas)
async for event in runner.run_async(
    user_id=user_id,
    session_id=session_id,
    new_message=types.Content(role="user", parts=[
        types.Part(text="What is the weather in New York?")]
    )
):
    if event.is_final_response():
        print(event.content.parts[0].text.strip())
```

## Soporte y Recursos
- [Documentación de Phoenix](https://arize.com/docs/phoenix/integrations/llm-providers/google-gen-ai/google-adk-tracing)
- [Slack de la Comunidad](https://arize-ai.slack.com/join/shared_invite/zt-11t1vbu4x-xkBIHmOREQnYnYDH1GDfCg#/shared-invite/email)
- [Paquete OpenInference](https://github.com/Arize-ai/openinference/tree/main/python/instrumentation/openinference-instrumentation-google-adk)