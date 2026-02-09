# Observabilidad de Agentes con Arize AX

[Arize AX](https://arize.com/docs/ax) es una plataforma de observabilidad de nivel de producción para monitorear, depurar y mejorar aplicaciones LLM y Agentes de IA a escala. Proporciona capacidades completas de rastreo, evaluación y monitoreo para tus aplicaciones Google ADK. Para comenzar, regístrate para obtener una [cuenta gratuita](https://app.arize.com/auth/join).

Para una alternativa de código abierto y auto-hospedada, consulta [Phoenix](https://arize.com/docs/phoenix).

## Descripción General

Arize AX puede recopilar automáticamente trazas de Google ADK usando [instrumentación OpenInference](https://github.com/Arize-ai/openinference/tree/main/python/instrumentation/openinference-instrumentation-google-adk), permitiéndote:

- **Rastrear interacciones de agentes** - Capturar automáticamente cada ejecución de agente, llamada a herramienta, solicitud de modelo y respuesta con contexto y metadatos
- **Evaluar rendimiento** - Evaluar el comportamiento del agente usando evaluadores personalizados o predefinidos y ejecutar experimentos para probar configuraciones de agentes
- **Monitorear en producción** - Configurar paneles de control y alertas en tiempo real para rastrear el rendimiento
- **Depurar problemas** - Analizar trazas detalladas para identificar rápidamente cuellos de botella, llamadas a herramientas fallidas y cualquier comportamiento inesperado del agente

![Agent Traces](https://storage.googleapis.com/arize-phoenix-assets/assets/images/google-adk-traces.png)

## Instalación

Instala los paquetes requeridos:

```bash
pip install openinference-instrumentation-google-adk google-adk arize-otel
```

## Configuración

### 1. Configurar Variables de Entorno { #configure-environment-variables }

Establece tu clave de API de Google:

```bash
export GOOGLE_API_KEY=[your_key_here]
```

### 2. Conectar tu aplicación a Arize AX { #connect-your-application-to-arize-ax }

```python
from arize.otel import register

# Registrar con Arize AX
tracer_provider = register(
    space_id="your-space-id",      # Se encuentra en la página de configuración del espacio de la aplicación
    api_key="your-api-key",        # Se encuentra en la página de configuración del espacio de la aplicación
    project_name="your-project-name"  # Nómbralo como prefieras
)

# Importar y configurar el instrumentador automático de OpenInference
from openinference.instrumentation.google_adk import GoogleADKInstrumentor

# Finalizar la instrumentación automática
GoogleADKInstrumentor().instrument(tracer_provider=tracer_provider)
```

## Observar

Ahora que tienes el rastreo configurado, todas las solicitudes del SDK de Google ADK se transmitirán a Arize AX para observabilidad y evaluación.

```python
import nest_asyncio
nest_asyncio.apply()

from google.adk.agents import Agent
from google.adk.runners import InMemoryRunner
from google.genai import types

# Definir una función de herramienta
def get_weather(city: str) -> dict:
    """Recupera el informe meteorológico actual para una ciudad especificada.

    Args:
        city (str): El nombre de la ciudad para la cual recuperar el informe meteorológico.

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
## Ver Resultados en Arize AX
![Traces in Arize AX](https://storage.googleapis.com/arize-phoenix-assets/assets/images/google-adk-dashboard.png)
![Agent Visualization](https://storage.googleapis.com/arize-phoenix-assets/assets/images/google-adk-agent.png)
![Agent Experiments](https://storage.googleapis.com/arize-phoenix-assets/assets/images/google-adk-experiments.png)

## Soporte y Recursos
- [Documentación de Arize AX](https://arize.com/docs/ax/integrations/frameworks-and-platforms/google-adk)
- [Slack de la Comunidad Arize](https://arize-ai.slack.com/join/shared_invite/zt-11t1vbu4x-xkBIHmOREQnYnYDH1GDfCg#/shared-invite/email)
- [Paquete OpenInference](https://github.com/Arize-ai/openinference/tree/main/python/instrumentation/openinference-instrumentation-google-adk)