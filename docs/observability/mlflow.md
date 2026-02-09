# Observabilidad de Agentes con MLflow

[MLflow Tracing](https://mlflow.org/docs/latest/genai/tracing/) proporciona
soporte de primera clase para ingerir trazas de OpenTelemetry (OTel). Google ADK emite
spans de OTel para ejecuciones de agentes, llamadas a herramientas y solicitudes de modelos, que puedes enviar
directamente a un MLflow Tracking Server para análisis y depuración.

## Prerequisitos

- MLflow versión 3.6.0 o superior. La ingesta de OpenTelemetry solo está soportada en
  MLflow 3.6.0+.
- Un almacenamiento backend basado en SQL (por ejemplo, SQLite, PostgreSQL, MySQL). Los almacenamientos basados en archivos
  no soportan ingesta OTLP.
- Google ADK instalado en tu entorno.

## Instalar dependencias

```bash
pip install "mlflow>=3.6.0" google-adk opentelemetry-sdk opentelemetry-exporter-otlp-proto-http
```

## Iniciar el MLflow Tracking Server

Inicia MLflow con un backend SQL y un puerto (5000 en este ejemplo):

```bash
mlflow server --backend-store-uri sqlite:///mlflow.db --port 5000
```

Puedes apuntar `--backend-store-uri` a otros backends SQL (PostgreSQL, MySQL,
MSSQL). La ingesta OTLP no está soportada con backends basados en archivos.

## Configurar OpenTelemetry (requerido)

Debes configurar un exportador OTLP y establecer un proveedor de trazas global antes
de usar cualquier componente de ADK para que los spans sean emitidos a MLflow.

Inicializa el exportador OTLP y el proveedor de trazas global en código antes de importar
o construir agentes/herramientas de ADK:

```python
# my_agent/agent.py
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import SimpleSpanProcessor

exporter = OTLPSpanExporter(
    endpoint="http://localhost:5000/v1/traces",
    headers={"x-mlflow-experiment-id": "123"}  # reemplaza con tu id de experimento
)

provider = TracerProvider()
provider.add_span_processor(SimpleSpanProcessor(exporter))
trace.set_tracer_provider(provider)  # establece ANTES de importar/usar ADK
```

Esto configura el pipeline de OpenTelemetry y envía los spans de ADK al servidor de MLflow
en cada ejecución.

## Ejemplo: Trazar un agente de ADK

Ahora puedes agregar el código del agente para un agente matemático simple, después del código que
configura el exportador OTLP y el proveedor de trazas:

```python
# my_agent/agent.py
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool


def calculator(a: float, b: float) -> str:
    """Suma dos números y devuelve el resultado."""
    return str(a + b)


calculator_tool = FunctionTool(func=calculator)

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

Ejecuta el agente con:

```bash
adk run my_agent
```

Y pregúntale un problema matemático:

```console
What is 12 + 34?
```

Luego deberías ver una salida similar a:

```console
[MathAgent]: The answer is 46.
```

## Ver trazas en MLflow

Abre la interfaz de MLflow en `http://localhost:5000`, selecciona tu experimento e
inspecciona el árbol de trazas y los spans generados por tu agente de ADK.

![MLflow Traces](https://mlflow.org/docs/latest/images/llms/tracing/google-adk-tracing.png)

## Consejos

- Establece el proveedor de trazas antes de importar o inicializar objetos de ADK para que todos
  los spans sean capturados.
- Detrás de un proxy o en un host remoto, reemplaza `localhost:5000` con la dirección de tu servidor.

## Recursos

- [Documentación de MLflow Tracing](https://mlflow.org/docs/latest/genai/tracing/):
  Documentación oficial para MLflow Tracing que cubre otras integraciones de bibliotecas
  y uso posterior de trazas, como evaluación, monitoreo,
  búsqueda y más.
- [OpenTelemetry en MLflow](https://mlflow.org/docs/latest/genai/tracing/opentelemetry/):
  Guía detallada sobre cómo usar OpenTelemetry con MLflow.
- [MLflow para Agentes](https://mlflow.org/docs/latest/genai/): Guía completa
  sobre cómo usar MLflow para construir agentes listos para producción.