# Apps: clase de gestión de flujo de trabajo

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v1.14.0</span>
</div>

La clase ***App*** es un contenedor de nivel superior para un flujo de trabajo completo de agente del Agent Development Kit (ADK). Está diseñada para gestionar el ciclo de vida, la configuración y el estado de una colección de agentes agrupados por un ***agente raíz***. La clase **App** separa las preocupaciones de la infraestructura operativa general de un flujo de trabajo de agente del razonamiento orientado a tareas de los agentes individuales.

Definir un objeto ***App*** en tu flujo de trabajo ADK es opcional y cambia cómo organizas tu código de agente y ejecutas tus agentes. Desde una perspectiva práctica, usas la clase ***App*** para configurar las siguientes características para tu flujo de trabajo de agente:

*   [**Almacenamiento en caché de contexto**](/context/caching/)
*   [**Compresión de contexto**](/context/compaction/)
*   [**Reanudación de agente**](/runtime/resume/)
*   [**Plugins**](/plugins/)

Esta guía explica cómo usar la clase App para configurar y gestionar tus flujos de trabajo de agente ADK.

## Propósito de la clase App

La clase ***App*** aborda varios problemas arquitectónicos que surgen al construir sistemas agénticos complejos:

*   **Configuración centralizada:** Proporciona una ubicación única y centralizada para gestionar recursos compartidos como claves API y clientes de base de datos, evitando la necesidad de pasar la configuración a través de cada agente.
*   **Gestión del ciclo de vida:** La clase ***App*** incluye hooks de ***inicio*** y ***cierre***, que permiten una gestión confiable de recursos persistentes como pools de conexión de base de datos o cachés en memoria que necesitan existir a través de múltiples invocaciones.
*   **Alcance de estado:** Define un límite explícito para el estado a nivel de aplicación con un prefijo `app:*` haciendo que el alcance y la vida útil de este estado sean claros para los desarrolladores.
*   **Unidad de despliegue:** El concepto de ***App*** establece una *unidad desplegable* formal, simplificando el versionado, las pruebas y el servicio de aplicaciones agénticas.

## Definir un objeto App

La clase ***App*** se usa como el contenedor principal de tu flujo de trabajo de agente y contiene el agente raíz del proyecto. El ***agente raíz*** es el contenedor para el agente controlador principal y cualquier sub-agente adicional.

### Definir app con agente raíz

Crea un ***agente raíz*** para tu flujo de trabajo creando una subclase desde la clase base ***Agent***. Luego define un objeto ***App*** y configúralo con el objeto ***agente raíz*** y características opcionales, como se muestra en el siguiente código de ejemplo:

```python title="agent.py"
from google.adk.agents.llm_agent import Agent
from google.adk.apps import App

root_agent = Agent(
    model='gemini-2.5-flash',
    name='greeter_agent',
    description='An agent that provides a friendly greeting.',
    instruction='Reply with Hello, World!',
)

app = App(
    name="agents",
    root_agent=root_agent,
    # Opcionalmente incluye características a nivel de App:
    # plugins, context_cache_config, resumability_config
)
```

!!! tip "Recomendado: Usa el nombre de variable `app`"

    En el código de tu proyecto de agente, establece tu objeto ***App*** al nombre de variable `app` para que sea compatible con las herramientas de ejecución de la interfaz de línea de comandos de ADK.

### Ejecutar tu agente App

Puedes usar la clase ***Runner*** para ejecutar tu flujo de trabajo de agente usando el parámetro `app`, como se muestra en el siguiente código de ejemplo:

```python title="main.py"
import asyncio
from dotenv import load_dotenv
from google.adk.runners import InMemoryRunner
from agent import app # importar código desde agent.py

load_dotenv() # cargar claves API y configuraciones
# Establecer un Runner usando el objeto de aplicación importado
runner = InMemoryRunner(app=app)

async def main():
    try:  # run_debug() requiere ADK Python 1.18 o superior:
        response = await runner.run_debug("Hello there!")
        
    except Exception as e:
        print(f"An error occurred during agent execution: {e}")

if __name__ == "__main__":
    asyncio.run(main())

```

!!! note "Requisito de versión para `Runner.run_debug()` "

    El comando `Runner.run_debug()` requiere ADK Python v1.18.0 o superior.
    También puedes usar `Runner.run()`, que requiere más código de configuración. Para
    más detalles, consulta

Ejecuta tu agente App con el código `main.py` usando el siguiente comando:

```console
python main.py
```

## Próximos pasos

Para una implementación de código de ejemplo más completa, consulta el ejemplo de código [Hello World App](https://github.com/google/adk-python/tree/main/contributing/samples/hello_world_app).