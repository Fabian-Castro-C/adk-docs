# Inicio Rápido de Python para ADK

Esta guía te muestra cómo empezar a trabajar con Agent Development Kit
(ADK) para Python. Antes de comenzar, asegúrate de tener lo siguiente instalado:

*   Python 3.10 o posterior
*   `pip` para instalar paquetes

## Instalación

Instala ADK ejecutando el siguiente comando:

```shell
pip install google-adk
```

??? tip "Recomendado: crear y activar un entorno virtual de Python"

    Crea un entorno virtual de Python:

    ```shell
    python -m venv .venv
    ```

    Activa el entorno virtual de Python:

    === "Windows CMD"

        ```console
        .venv\Scripts\activate.bat
        ```

    === "Windows Powershell"

        ```console
        .venv\Scripts\Activate.ps1
        ```

    === "MacOS / Linux"

        ```bash
        source .venv/bin/activate
        ```

## Crear un proyecto de agente

Ejecuta el comando `adk create` para iniciar un nuevo proyecto de agente.

```shell
adk create my_agent
```

### Explorar el proyecto de agente

El proyecto de agente creado tiene la siguiente estructura, con el archivo `agent.py`
que contiene el código de control principal para el agente.

```none
my_agent/
    agent.py      # código principal del agente
    .env          # claves API o IDs de proyecto
    __init__.py
```

## Actualizar tu proyecto de agente

El archivo `agent.py` contiene una definición de `root_agent` que es el único
elemento requerido de un agente ADK. También puedes definir herramientas para que el agente
las use. Actualiza el código generado de `agent.py` para incluir una herramienta `get_current_time`
para uso del agente, como se muestra en el siguiente código:

```python
from google.adk.agents.llm_agent import Agent

# Implementación simulada de herramienta
def get_current_time(city: str) -> dict:
    """Devuelve la hora actual en una ciudad especificada."""
    return {"status": "success", "city": city, "time": "10:30 AM"}

root_agent = Agent(
    model='gemini-3-flash-preview',
    name='root_agent',
    description="Dice la hora actual en una ciudad especificada.",
    instruction="Eres un asistente útil que dice la hora actual en las ciudades. Usa la herramienta 'get_current_time' para este propósito.",
    tools=[get_current_time],
)
```

### Configurar tu clave API

Este proyecto usa la API de Gemini, que requiere una clave API. Si no
tienes ya una clave API de Gemini, crea una clave en Google AI Studio en la
página de [Claves API](https://aistudio.google.com/app/apikey).

En una ventana de terminal, escribe tu clave API en un archivo `.env` como una variable de entorno:

```console title="Actualizar: my_agent/.env"
echo 'GOOGLE_API_KEY="YOUR_API_KEY"' > .env
```

??? tip "Usar otros modelos de IA con ADK"
    ADK soporta el uso de muchos modelos de IA generativa. Para más
    información sobre cómo configurar otros modelos en agentes ADK, consulta
    [Modelos y Autenticación](/agents/models).

## Ejecutar tu agente

Puedes ejecutar tu agente ADK con una interfaz de línea de comandos interactiva usando el
comando `adk run` o la interfaz de usuario web de ADK proporcionada por ADK usando el
comando `adk web`. Ambas opciones te permiten probar e interactuar con tu
agente.

### Ejecutar con interfaz de línea de comandos

Ejecuta tu agente usando la herramienta de línea de comandos `adk run`.

```console
adk run my_agent
```

![adk-run.png](/assets/adk-run.png)

### Ejecutar con interfaz web

El framework ADK proporciona una interfaz web que puedes usar para probar e interactuar con
tu agente. Puedes iniciar la interfaz web usando el siguiente comando:

```console
adk web --port 8000
```

!!! note

    Ejecuta este comando desde el **directorio padre** que contiene tu
    carpeta `my_agent/`. Por ejemplo, si tu agente está dentro de `agents/my_agent/`,
    ejecuta `adk web` desde el directorio `agents/`.

Este comando inicia un servidor web con una interfaz de chat para tu agente. Puedes
acceder a la interfaz web en (http://localhost:8000). Selecciona el agente en la
esquina superior izquierda y escribe una solicitud.

![adk-web-dev-ui-chat.png](/assets/adk-web-dev-ui-chat.png)

!!! warning "Precaución: ADK Web solo para desarrollo"

    ADK Web ***no está diseñado para uso en despliegues de producción***. Deberías
    usar ADK Web solo para propósitos de desarrollo y depuración.

## Siguiente: Construye tu agente

Ahora que tienes ADK instalado y tu primer agente ejecutándose, intenta construir
tu propio agente con nuestras guías de construcción:

*  [Construye tu agente](/tutorials/)