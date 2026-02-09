# Observabilidad de Agentes con Weave by WandB

[Weave by Weights & Biases (WandB)](https://weave-docs.wandb.ai/) proporciona una plataforma potente para registrar y visualizar llamadas de modelo. Al integrar Google ADK con Weave, puedes rastrear y analizar el rendimiento y comportamiento de tu agente usando trazas de OpenTelemetry (OTEL).

## Requisitos Previos

1. Regístrate para obtener una cuenta en [WandB](https://wandb.ai).

2. Obtén tu clave API desde [WandB Authorize](https://wandb.ai/authorize).

3. Configura tu entorno con las claves API requeridas:

   ```bash
   export WANDB_API_KEY=<your-wandb-api-key>
   export GOOGLE_API_KEY=<your-google-api-key>
   ```

## Instalar Dependencias

Asegúrate de tener los paquetes necesarios instalados:

```bash
pip install google-adk opentelemetry-sdk opentelemetry-exporter-otlp-proto-http
```

## Enviando Trazas a Weave

Este ejemplo demuestra cómo configurar OpenTelemetry para enviar trazas de Google ADK a Weave.

```python
# math_agent/agent.py

import base64
import os
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk import trace as trace_sdk
from opentelemetry.sdk.trace.export import SimpleSpanProcessor
from opentelemetry import trace

from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool

from dotenv import load_dotenv

load_dotenv()

# Configurar endpoint de Weave y autenticación
WANDB_BASE_URL = "https://trace.wandb.ai"
PROJECT_ID = "your-entity/your-project"  # ej., "teamid/projectid"
OTEL_EXPORTER_OTLP_ENDPOINT = f"{WANDB_BASE_URL}/otel/v1/traces"

# Configurar autenticación
WANDB_API_KEY = os.getenv("WANDB_API_KEY")
AUTH = base64.b64encode(f"api:{WANDB_API_KEY}".encode()).decode()

OTEL_EXPORTER_OTLP_HEADERS = {
    "Authorization": f"Basic {AUTH}",
    "project_id": PROJECT_ID,
}

# Crear el exportador de spans OTLP con endpoint y headers
exporter = OTLPSpanExporter(
    endpoint=OTEL_EXPORTER_OTLP_ENDPOINT,
    headers=OTEL_EXPORTER_OTLP_HEADERS,
)

# Crear un proveedor de trazador y agregar el exportador
tracer_provider = trace_sdk.TracerProvider()
tracer_provider.add_span_processor(SimpleSpanProcessor(exporter))

# Configurar el proveedor de trazador global ANTES de importar/usar ADK
trace.set_tracer_provider(tracer_provider)

# Definir una herramienta simple para demostración
def calculator(a: float, b: float) -> str:
    """Suma dos números y devuelve el resultado.

    Args:
        a: Primer número
        b: Segundo número

    Returns:
        La suma de a y b
    """
    return str(a + b)

calculator_tool = FunctionTool(func=calculator)

# Crear un agente LLM
root_agent = LlmAgent(
    name="MathAgent",
    model="gemini-2.0-flash-exp",
    instruction=(
        "You are a helpful assistant that can do math. "
        "When asked a math problem, use the calculator tool to solve it."
    ),
    tools=[calculator_tool],
)
```

## Ver Trazas en el panel de Weave

Una vez que el agente se ejecuta, todas sus trazas se registran en el proyecto correspondiente en [el panel de Weave](https://wandb.ai/home).

![Traces in Weave](https://wandb.github.io/weave-public-assets/google-adk/traces-overview.png)

Puedes ver una línea de tiempo de las llamadas que tu agente ADK realizó durante la ejecución -

![Timeline view](https://wandb.github.io/weave-public-assets/google-adk/adk-weave-timeline.gif)


## Notas

- **Variables de Entorno**: Asegúrate de que tus variables de entorno estén configuradas correctamente tanto para WandB como para las claves API de Google.
- **Configuración de Proyecto**: Reemplaza `<your-entity>/<your-project>` con tu entidad y nombre de proyecto real de WandB.
- **Nombre de Entidad**: Puedes encontrar tu nombre de entidad visitando tu [panel de WandB](https://wandb.ai/home) y verificando el campo **Teams** en la barra lateral izquierda.
- **Proveedor de Trazador**: Es crítico configurar el proveedor de trazador global antes de usar cualquier componente de ADK para asegurar un trazado adecuado.

Siguiendo estos pasos, puedes integrar efectivamente Google ADK con Weave, habilitando el registro y visualización completa de las llamadas de modelo de tus agentes de IA, invocaciones de herramientas y procesos de razonamiento.

## Recursos

- **[Send OpenTelemetry Traces to Weave](https://weave-docs.wandb.ai/guides/tracking/otel)** - Guía completa sobre la configuración de OTEL con Weave, incluyendo autenticación y opciones de configuración avanzadas.

- **[Navigate the Trace View](https://weave-docs.wandb.ai/guides/tracking/trace-tree)** - Aprende cómo analizar y depurar efectivamente tus trazas en la interfaz de Weave, incluyendo la comprensión de jerarquías de trazas y detalles de spans.

- **[Weave Integrations](https://weave-docs.wandb.ai/guides/integrations/)** - Explora otras integraciones de frameworks y ve cómo Weave puede funcionar con todo tu stack de IA.