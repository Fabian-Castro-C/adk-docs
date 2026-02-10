---
catalog_title: Computer Use
catalog_description: Operate computer user interfaces using Gemini models
catalog_icon: /assets/tools-gemini-spark.svg
---

# Conjunto de Herramientas de Uso de Computadora con la API de Gemini

<div class="language-support-tag">
  <span class="lst-supported">Compatible con ADK</span><span class="lst-python">Python v1.17.0</span><span class="lst-preview">Vista previa</span>
</div>

El Conjunto de Herramientas de Uso de Computadora permite a un agente operar una interfaz de usuario
de una computadora, como navegadores, para completar tareas. Esta herramienta utiliza
un modelo específico de Gemini y la herramienta de pruebas [Playwright](https://playwright.dev/)
para controlar un navegador Chromium y puede interactuar con
páginas web tomando capturas de pantalla, haciendo clic, escribiendo y navegando.

Para más información sobre el modelo de uso de computadora, consulta
la API de Gemini [Computer use](https://ai.google.dev/gemini-api/docs/computer-use)
o la API de Vertex AI de Google Cloud
[Computer use](https://cloud.google.com/vertex-ai/generative-ai/docs/computer-use).

!!! example "Versión de vista previa"
    El modelo y herramienta de Uso de Computadora es una versión de Vista previa. Para
    más información, consulta las
    [descripciones de etapas de lanzamiento](https://cloud.google.com/products#product-launch-stages).

## Configuración

Debes instalar Playwright y sus dependencias, incluyendo Chromium,
para poder usar el Conjunto de Herramientas de Uso de Computadora.

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

Para configurar las bibliotecas de software requeridas para el Conjunto de Herramientas de Uso de Computadora:

1.  Instala las dependencias de Python:
    ```console
    pip install termcolor==3.1.0
    pip install playwright==1.52.0
    pip install browserbase==1.3.0
    pip install rich
    ```
2.  Instala las dependencias de Playwright, incluyendo el navegador Chromium:
    ```console
    playwright install-deps chromium
    playwright install chromium
    ```

## Usar la herramienta

Usa el Conjunto de Herramientas de Uso de Computadora agregándolo como herramienta a tu agente. Cuando
configures la herramienta, debes proporcionar una implementación de la clase `BaseComputer`
que define una interfaz para que un agente use una computadora. En el
siguiente ejemplo, la clase `PlaywrightComputer` se define para este propósito.
Puedes encontrar el código para esta implementación en el archivo `playwright.py` del
proyecto de muestra del agente
[computer_use](https://github.com/google/adk-python/blob/main/contributing/samples/computer_use/playwright.py).

```python
from google.adk import Agent
from google.adk.models.google_llm import Gemini
from google.adk.tools.computer_use.computer_use_toolset import ComputerUseToolset
from typing_extensions import override

from .playwright import PlaywrightComputer

root_agent = Agent(
    model='gemini-2.5-computer-use-preview-10-2025',
    name='hello_world_agent',
    description=(
        'computer use agent that can operate a browser on a computer to finish'
        ' user tasks'
    ),
    instruction='you are a computer use agent',
    tools=[
        ComputerUseToolset(computer=PlaywrightComputer(screen_size=(1280, 936)))
    ],
)
```

Para un ejemplo de código completo, consulta el
proyecto de muestra del agente
[computer_use](https://github.com/google/adk-python/tree/main/contributing/samples/computer_use).