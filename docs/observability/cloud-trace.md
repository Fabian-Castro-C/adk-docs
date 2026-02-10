# Observabilidad de Agentes con Cloud Trace

Con ADK, ya eres capaz de inspeccionar y observar la interacción de tu agente localmente utilizando la potente interfaz de usuario de desarrollo web discutida en [aquí](https://google.github.io/evaluate/#debugging-with-the-trace-view). Sin embargo, si nuestro objetivo es el despliegue en la nube, necesitaremos un panel centralizado para observar el tráfico real.

Cloud Trace es un componente de Google Cloud Observability. Es una herramienta poderosa para monitorear, depurar y mejorar el rendimiento de tus aplicaciones enfocándose específicamente en capacidades de rastreo. Para aplicaciones del Agent Development Kit (ADK), Cloud Trace habilita rastreo integral, ayudándote a entender cómo fluyen las solicitudes a través de las interacciones de tu agente e identificar cuellos de botella de rendimiento o errores dentro de tus agentes de IA.

## Descripción General

Cloud Trace está construido sobre [OpenTelemetry](https://opentelemetry.io/), un estándar de código abierto que soporta muchos lenguajes y métodos de ingesta para generar datos de rastreo. Esto se alinea con las prácticas de observabilidad para aplicaciones ADK, que también aprovechan la instrumentación compatible con OpenTelemetry, permitiéndote:

- Rastrear interacciones de agentes: Cloud Trace recopila y analiza continuamente datos de rastreo de tu proyecto, permitiéndote diagnosticar rápidamente problemas de latencia y errores dentro de tus aplicaciones ADK. Esta recopilación automática de datos simplifica el proceso de identificar problemas en flujos de trabajo complejos de agentes.
- Depurar problemas: Diagnostica rápidamente problemas de latencia y errores analizando rastros detallados. Crucial para entender problemas que se manifiestan como aumento en la latencia de comunicación entre diferentes servicios o durante acciones específicas del agente como llamadas a herramientas.
- Análisis y Visualización Profundos: Trace Explorer es la herramienta principal para analizar rastros, ofreciendo ayudas visuales como mapas de calor para la duración de spans y gráficos de líneas para tasas de solicitud/error. También proporciona una tabla de spans, agrupable por servicio y operación, que brinda acceso con un clic a rastros representativos y una vista en cascada para identificar fácilmente cuellos de botella y fuentes de errores dentro de la ruta de ejecución de tu agente.

El siguiente ejemplo asumirá la siguiente estructura de directorios del agente:

```
working_dir/
├── weather_agent/
│   ├── agent.py
│   └── __init__.py
└── deploy_agent_engine.py
└── deploy_fast_api_app.py
└── agent_runner.py
```

```python
# weather_agent/agent.py

import os
from google.adk.agents import Agent

os.environ.setdefault("GOOGLE_CLOUD_PROJECT", "{your-project-id}")
os.environ.setdefault("GOOGLE_CLOUD_LOCATION", "global")
os.environ.setdefault("GOOGLE_GENAI_USE_VERTEXAI", "True")


# Define una función de herramienta
def get_weather(city: str) -> dict:
    """Recupera el reporte de clima actual para una ciudad especificada.

    Args:
        city (str): El nombre de la ciudad para la cual recuperar el reporte del clima.

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


# Crea un agente con herramientas
root_agent = Agent(
    name="weather_agent",
    model="gemini-2.5-flash",
    description="Agent to answer questions using weather tools.",
    instruction="You must use the available tools to find an answer.",
    tools=[get_weather],
)
```

## Configuración de Cloud Trace

### Configuración para Despliegue en Agent Engine

#### Despliegue en Agent Engine - desde ADK CLI

Puedes habilitar el rastreo en la nube agregando la bandera `--trace_to_cloud` al desplegar tu agente usando el comando `adk deploy agent_engine` para el despliegue en agent engine.

```bash
adk deploy agent_engine \
    --project=$GOOGLE_CLOUD_PROJECT \
    --region=$GOOGLE_CLOUD_LOCATION \
    --staging_bucket=$STAGING_BUCKET \
    --trace_to_cloud \
    $AGENT_PATH
```

#### Despliegue en Agent Engine - desde Python SDK

Si prefieres usar Python SDK, puedes habilitar el rastreo en la nube agregando `enable_tracing=True` al inicializar el objeto `AdkApp`:

```python
# deploy_agent_engine.py

from vertexai.preview import reasoning_engines
from vertexai import agent_engines
from weather_agent.agent import root_agent

import vertexai

PROJECT_ID = "{your-project-id}"
LOCATION = "{your-preferred-location}"
STAGING_BUCKET = "{your-staging-bucket}"

vertexai.init(
    project=PROJECT_ID,
    location=LOCATION,
    staging_bucket=STAGING_BUCKET,
)

adk_app = reasoning_engines.AdkApp(
    agent=root_agent,
    enable_tracing=True,
)


remote_app = agent_engines.create(
    agent_engine=adk_app,
    extra_packages=[
        "./weather_agent",
    ],
    requirements=[
        "google-cloud-aiplatform[adk,agent_engines]",
    ],
)
```

### Configuración para Despliegue en Cloud Run

#### Despliegue en Cloud Run - desde ADK CLI

Puedes habilitar el rastreo en la nube agregando la bandera `--trace_to_cloud` al desplegar tu agente usando el comando `adk deploy cloud_run` para el despliegue en cloud run.

```bash
adk deploy cloud_run \
    --project=$GOOGLE_CLOUD_PROJECT \
    --region=$GOOGLE_CLOUD_LOCATION \
    --trace_to_cloud \
    $AGENT_PATH
```

Si deseas habilitar el rastreo en la nube y usar un despliegue de servicio de agente personalizado en Cloud Run, puedes consultar la sección [Configuración para Despliegue Personalizado](#setup-for-customized-deployment) a continuación.

### Configuración para Despliegue Personalizado

#### Desde el Módulo Integrado `get_fast_api_app`

Si deseas personalizar tu propio servicio de agente, puedes habilitar el rastreo en la nube inicializando la aplicación FastAPI usando el módulo integrado `get_fast_api_app` y configurando `trace_to_cloud=True`:

```python
# deploy_fast_api_app.py

import os
from google.adk.cli.fast_api import get_fast_api_app
from fastapi import FastAPI

# Establece la variable de entorno GOOGLE_CLOUD_PROJECT para rastreo en la nube
os.environ.setdefault("GOOGLE_CLOUD_PROJECT", "alvin-exploratory-2")

# Descubre el directorio `weather_agent` en el directorio de trabajo actual
AGENT_DIR = os.path.dirname(os.path.abspath(__file__))

# Crea la aplicación FastAPI con rastreo en la nube habilitado
app: FastAPI = get_fast_api_app(
    agents_dir=AGENT_DIR,
    web=True,
    trace_to_cloud=True,
)

app.title = "weather-agent"
app.description = "API for interacting with the Agent weather-agent"


# Ejecución principal
if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8080)
```


#### Desde un Agent Runner Personalizado

Si deseas personalizar completamente el runtime de tu agente ADK, puedes habilitar el rastreo en la nube usando el módulo `CloudTraceSpanExporter` de Opentelemetry.

```python
# agent_runner.py

from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from weather_agent.agent import root_agent as weather_agent
from google.genai.types import Content, Part
from opentelemetry import trace
from opentelemetry.exporter.cloud_trace import CloudTraceSpanExporter
from opentelemetry.sdk.trace import export
from opentelemetry.sdk.trace import TracerProvider

APP_NAME = "weather_agent"
USER_ID = "u_123"
SESSION_ID = "s_123"

provider = TracerProvider()
processor = export.BatchSpanProcessor(
    CloudTraceSpanExporter(project_id="{your-project-id}")
)
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)

session_service = InMemorySessionService()
runner = Runner(agent=weather_agent, app_name=APP_NAME, session_service=session_service)


async def main():
    session = await session_service.get_session(
        app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID
    )
    if session is None:
        session = await session_service.create_session(
            app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID
        )

    user_content = Content(
        role="user", parts=[Part(text="what's weather in paris?")]
    )

    final_response_content = "No response"
    async for event in runner.run_async(
        user_id=USER_ID, session_id=SESSION_ID, new_message=user_content
    ):
        if event.is_final_response() and event.content and event.content.parts:
            final_response_content = event.content.parts[0].text

    print(final_response_content)


if __name__ == "__main__":
    import asyncio

    asyncio.run(main())
```

## Inspeccionar Cloud Traces

Una vez completada la configuración, cada vez que interactúes con el agente, automáticamente enviará datos de rastreo a Cloud Trace. Puedes inspeccionar los rastros yendo a [console.cloud.google.com](https://console.cloud.google.com) y visitando el Trace Explorer en el Google Cloud Project configurado.

![cloud-trace](../assets/cloud-trace1.png)

Y luego verás todos los rastros disponibles producidos por el agente ADK que están configurados en varios nombres de span como `invocation`, `agent_run`, `call_llm` y `execute_tool`.

![cloud-trace](../assets/cloud-trace2.png)

Si haces clic en uno de los rastros, verás la vista en cascada del proceso detallado, similar a lo que vemos en la interfaz de usuario de desarrollo web con el comando `adk web`.

![cloud-trace](../assets/cloud-trace3.png)

## Recursos

- [Documentación de Google Cloud Trace](https://cloud.google.com/trace)