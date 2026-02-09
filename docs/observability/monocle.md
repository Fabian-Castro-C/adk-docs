# Observabilidad de Agentes con Monocle

[Monocle](https://github.com/monocle2ai/monocle) es una plataforma de observabilidad de código abierto para monitorear, depurar y mejorar aplicaciones LLM y Agentes de IA. Proporciona capacidades de trazado integrales para tus aplicaciones de Google ADK mediante instrumentación automática. Monocle genera trazas compatibles con OpenTelemetry que se pueden exportar a varios destinos, incluyendo archivos locales o salida por consola.

## Descripción General

Monocle instrumenta automáticamente las aplicaciones de Google ADK, permitiéndote:

- **Rastrear interacciones de agentes** - Captura automáticamente cada ejecución de agente, llamada a herramienta y solicitud de modelo con contexto completo y metadatos
- **Monitorear el flujo de ejecución** - Rastrea el estado del agente, eventos de delegación y flujo de ejecución mediante trazas detalladas
- **Depurar problemas** - Analiza trazas detalladas para identificar rápidamente cuellos de botella, llamadas a herramientas fallidas y comportamiento inesperado del agente
- **Opciones flexibles de exportación** - Exporta trazas a archivos locales o consola para su análisis
- **Compatible con OpenTelemetry** - Genera trazas estándar de OpenTelemetry que funcionan con cualquier backend compatible con OTLP

Monocle instrumenta automáticamente los siguientes componentes de Google ADK:

- **`BaseAgent.run_async`** - Captura la ejecución del agente, el estado del agente y los eventos de delegación
- **`FunctionTool.run_async`** - Captura la ejecución de herramientas, incluyendo nombre de herramienta, parámetros y resultados
- **`Runner.run_async`** - Captura la ejecución del runner, incluyendo el contexto de solicitud y el flujo de ejecución

## Instalación

### 1. Instalar Paquetes Requeridos { #install-required-packages }

```bash
pip install monocle_apptrace google-adk
```

## Configuración

### 1. Configurar Telemetría de Monocle { #configure-monocle-telemetry }

Monocle instrumenta automáticamente Google ADK cuando inicializas la telemetría. Simplemente llama a `setup_monocle_telemetry()` al inicio de tu aplicación:

```python
from monocle_apptrace import setup_monocle_telemetry

# Inicializar telemetría de Monocle - instrumenta automáticamente Google ADK
setup_monocle_telemetry(workflow_name="my-adk-app")
```

¡Eso es todo! Monocle detectará e instrumentará automáticamente tus agentes, herramientas y runners de Google ADK.

### 2. Configurar Exportadores (Opcional) { #configure-exporters }

Por defecto, Monocle exporta trazas a archivos JSON locales. Puedes configurar diferentes exportadores usando variables de entorno.

#### Exportar a Consola (para depuración)

Define la variable de entorno:

```bash
export MONOCLE_EXPORTER="console"
```

#### Exportar a Archivos Locales (predeterminado)

```bash
export MONOCLE_EXPORTER="file"
```

O simplemente omite la variable `MONOCLE_EXPORTER` - por defecto es `file`.

## Observar

Ahora que tienes configurado el trazado, todas las solicitudes del SDK de Google ADK serán rastreadas automáticamente por Monocle.

```python
from monocle_apptrace import setup_monocle_telemetry
from google.adk.agents import Agent
from google.adk.runners import InMemoryRunner
from google.genai import types

# Inicializar telemetría de Monocle - debe llamarse antes de usar ADK
setup_monocle_telemetry(workflow_name="weather_app")

# Definir una función de herramienta
def get_weather(city: str) -> dict:
    """Recupera el reporte del clima actual para una ciudad especificada.

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

# Ejecutar el agente (todas las interacciones serán rastreadas automáticamente)
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

## Accediendo a las Trazas

Por defecto, Monocle genera trazas en archivos JSON en el directorio local `./monocle`. El formato del nombre de archivo es:

```
monocle_trace_{workflow_name}_{trace_id}_{timestamp}.json
```

Cada archivo de traza contiene un array de spans compatibles con OpenTelemetry que capturan:

- **Spans de ejecución de agente** - Estado del agente, eventos de delegación y flujo de ejecución
- **Spans de ejecución de herramienta** - Nombre de herramienta, parámetros de entrada y resultados de salida
- **Spans de interacción LLM** - Llamadas al modelo, prompts, respuestas y uso de tokens (si usas Gemini u otros LLMs)

Puedes analizar estos archivos de traza usando cualquier herramienta compatible con OpenTelemetry o escribir scripts de análisis personalizados.

## Visualizando Trazas con la Extensión de VS Code

La extensión de VS Code [Okahu Trace Visualizer](https://marketplace.visualstudio.com/items?itemName=OkahuAI.okahu-ai-observability) proporciona una forma interactiva de visualizar y analizar trazas generadas por Monocle directamente en Visual Studio Code.

### Instalación

1. Abre VS Code
2. Presiona `Ctrl+P` (o `Cmd+P` en Mac) para abrir Quick Open
3. Pega el siguiente comando y presiona Enter:

```
ext install OkahuAI.okahu-ai-observability
```

Alternativamente, puedes instalarlo desde el [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=OkahuAI.okahu-ai-observability).

### Características

La extensión proporciona:

- **Panel Personalizado de Barra de Actividad** - Barra lateral dedicada para la gestión de archivos de traza
- **Árbol de Archivos Interactivo** - Navega y selecciona archivos de traza con interfaz React personalizada
- **Análisis en Vista Dividida** - Visualización de gráfico Gantt junto al visor de datos JSON
- **Comunicación en Tiempo Real** - Flujo de datos sin interrupciones entre VS Code y componentes React
- **Temas de VS Code** - Totalmente integrado con los temas claro/oscuro de VS Code

### Uso

1. Después de ejecutar tu aplicación ADK con el trazado de Monocle habilitado, los archivos de traza se generarán en el directorio `./monocle`
2. Abre el panel Okahu Trace Visualizer desde la Barra de Actividad de VS Code
3. Navega y selecciona archivos de traza desde el árbol de archivos interactivo
4. Visualiza tus trazas con:
   - **Visualización de gráfico Gantt** - Ve la línea de tiempo y jerarquía de spans
   - **Visor de datos JSON** - Inspecciona atributos y eventos de span detallados
   - **Conteos de tokens** - Ve el uso de tokens para llamadas LLM
   - **Badges de error** - Identifica rápidamente operaciones fallidas

![Monocle VS Code Extension](../assets/monocle-vs-code-ext.png)

## Qué se Rastrea

Monocle captura automáticamente la siguiente información de Google ADK:

- **Ejecución de Agente**: Estado del agente, eventos de delegación y flujo de ejecución
- **Llamadas a Herramientas**: Nombre de herramienta, parámetros de entrada y resultados de salida
- **Ejecución de Runner**: Contexto de solicitud y flujo de ejecución general
- **Información de Tiempo**: Hora de inicio, hora de fin y duración de cada operación
- **Información de Error**: Excepciones y estados de error

Todas las trazas se generan en formato OpenTelemetry, haciéndolas compatibles con cualquier backend de observabilidad compatible con OTLP.

## Soporte y Recursos

- [Documentación de Monocle](https://docs.okahu.ai/monocle_overview/)
- [Repositorio GitHub de Monocle](https://github.com/monocle2ai/monocle)
- [Ejemplo de Agente de Viajes de Google ADK](https://github.com/okahu-demos/adk-travel-agent)
- [Comunidad de Discord](https://discord.gg/D8vDbSUhJX)