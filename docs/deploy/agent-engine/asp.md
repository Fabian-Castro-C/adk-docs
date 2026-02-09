# Desplegar en Agent Engine con Agent Starter Pack

<div class="language-support-tag" title="Vertex AI Agent Engine currently supports only Python.">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span>
</div>

Este procedimiento de despliegue describe cómo realizar un despliegue utilizando el
[Agent Starter Pack](https://github.com/GoogleCloudPlatform/agent-starter-pack)
(ASP) y la herramienta de interfaz de línea de comandos (CLI) de ADK. Usar ASP para el despliegue en
el entorno de ejecución de Agent Engine es una ruta acelerada, y debes usarlo para
_*desarrollo y pruebas*_ únicamente. La herramienta ASP configura recursos de Google Cloud que
no son estrictamente necesarios para ejecutar un flujo de trabajo de agente ADK, y debes
revisar minuciosamente esa configuración antes de usarla en un despliegue de
producción.

Esta guía de despliegue utiliza la herramienta ASP para aplicar una plantilla de proyecto a tu
proyecto existente, agregar artefactos de despliegue y preparar tu proyecto de agente para
el despliegue. Estas instrucciones te muestran cómo usar ASP para aprovisionar un proyecto de Google
Cloud con los servicios necesarios para desplegar tu proyecto ADK, de la siguiente manera:

-   [Prerrequisitos](#prerequisites-ad): Configurar cuenta de Google Cloud,
    un proyecto e instalar el software requerido.
-   [Preparar tu proyecto ADK](#prepare-ad): Modificar tus
    archivos de proyecto ADK existentes para prepararte para el despliegue.
-   [Conectar a tu proyecto de Google Cloud](#connect-ad):
    Conectar tu entorno de desarrollo a Google Cloud y a tu proyecto de Google Cloud.
-   [Desplegar tu proyecto ADK](#deploy-ad): Aprovisionar
    servicios requeridos en tu proyecto de Google Cloud y cargar el código de tu proyecto ADK.

Para obtener información sobre cómo probar un agente desplegado, consulta [Probar agente desplegado](test.md).
Para obtener más información sobre el uso de Agent Starter Pack y sus herramientas de línea de comandos,
consulta la
[referencia de CLI](https://googlecloudplatform.github.io/agent-starter-pack/cli/enhance.html)
y la
[guía de desarrollo](https://googlecloudplatform.github.io/agent-starter-pack/guide/development-guide.html).

### Prerrequisitos {#prerequisites-ad}

Necesitas los siguientes recursos configurados para usar esta ruta de despliegue:

-   **Cuenta de Google Cloud**: con acceso de administrador a lo siguiente:
    -   **Proyecto de Google Cloud**: Un proyecto de Google Cloud vacío con
        [facturación habilitada](https://cloud.google.com/billing/docs/how-to/modify-project).
        Para obtener información sobre cómo crear proyectos, consulta
        [Crear y gestionar proyectos](https://cloud.google.com/resource-manager/docs/creating-managing-projects).
-   **Entorno de Python**: Una versión de Python compatible con el
    [proyecto ASP](https://googlecloudplatform.github.io/agent-starter-pack/guide/getting-started.html).
-   **Herramienta uv:** Gestionar el entorno de desarrollo de Python y ejecutar herramientas
    ASP. Para detalles de instalación, consulta
    [Instalar uv](https://docs.astral.sh/uv/getting-started/installation/).
-   **Herramienta CLI de Google Cloud**: La interfaz de línea de comandos gcloud. Para
    detalles de instalación, consulta
    [Interfaz de línea de comandos de Google Cloud](https://cloud.google.com/sdk/docs/install).
-   **Herramienta Make**: Herramienta de automatización de compilación. Esta herramienta es parte de la mayoría de
    sistemas basados en Unix, para detalles de instalación, consulta la
    documentación de [herramienta Make](https://www.gnu.org/software/make/).

### Preparar tu proyecto ADK {#prepare-ad}

Cuando despliegas un proyecto ADK en Agent Engine, necesitas algunos archivos adicionales
para admitir la operación de despliegue. El siguiente comando ASP hace una copia de seguridad de tu
proyecto y luego agrega archivos a tu proyecto con fines de despliegue.

Estas instrucciones asumen que tienes un proyecto ADK existente que estás modificando
para el despliegue. Si no tienes un proyecto ADK, o deseas usar un proyecto
de prueba, completa la guía de Python
[Inicio rápido](/adk-docs/get-started/quickstart/),
que crea un proyecto
[multi_tool_agent](https://github.com/google/adk-docs/tree/main/examples/python/snippets/get-started/multi_tool_agent).
Las siguientes instrucciones usan el proyecto `multi_tool_agent` como
ejemplo.

Para preparar tu proyecto ADK para el despliegue en Agent Engine:

1.  En una ventana de terminal de tu entorno de desarrollo, navega al
    **directorio padre** que contiene tu carpeta de agente. Por ejemplo, si tu
    estructura de proyecto es:

    ```
    your-project-directory/
    ├── multi_tool_agent/
    │   ├── __init__.py
    │   ├── agent.py
    │   └── .env
    ```

    Navega a `your-project-directory/`

1.  Ejecuta el comando `enhance` de ASP para agregar los archivos requeridos para el despliegue a
    tu proyecto.

    ```shell
    uvx agent-starter-pack enhance --adk -d agent_engine
    ```

1.  Sigue las instrucciones de la herramienta ASP. En general, puedes aceptar
    las respuestas predeterminadas a todas las preguntas. Sin embargo, para la opción de **región GCP**,
    asegúrate de seleccionar una de las
    [regiones compatibles](https://docs.cloud.google.com/agent-builder/locations#supported-regions-agent-engine)
    para Agent Engine.

Cuando completes este proceso con éxito, la herramienta muestra el siguiente mensaje:

```
> Success! Your agent project is ready.
```

!!! tip "Nota"
    La herramienta ASP puede mostrar un recordatorio para conectarse a Google Cloud mientras
    se ejecuta, pero esa conexión *no es requerida* en esta etapa.

Para obtener más información sobre los cambios que ASP realiza en tu proyecto ADK, consulta
[Cambios en tu proyecto ADK](#adk-asp-changes).

### Conectar a tu proyecto de Google Cloud {#connect-ad}

Antes de desplegar tu proyecto ADK, debes conectarte a Google Cloud y a tu
proyecto. Después de iniciar sesión en tu cuenta de Google Cloud, debes verificar que
tu proyecto de destino de despliegue sea visible desde tu cuenta y que esté
configurado como tu proyecto actual.

Para conectarte a Google Cloud y listar tu proyecto:

1.  En una ventana de terminal de tu entorno de desarrollo, inicia sesión en tu
    cuenta de Google Cloud:

    ```shell
    gcloud auth application-default login
    ```

1.  Establece tu proyecto de destino usando el ID del proyecto de Google Cloud:

    ```shell
    gcloud config set project your-project-id-xxxxx
    ```

1.  Verifica que tu proyecto de destino de Google Cloud esté establecido:

    ```shell
    gcloud config get-value project
    ```

Una vez que te hayas conectado exitosamente a Google Cloud y hayas establecido tu ID de proyecto de Cloud,
estás listo para desplegar los archivos de tu proyecto ADK en Agent Engine.

### Desplegar tu proyecto ADK {#deploy-ad}

Cuando usas la herramienta ASP, despliegas en etapas. En la primera etapa, ejecutas un
comando `make` que aprovisiona los servicios necesarios para ejecutar tu flujo de trabajo ADK en
Agent Engine. En la segunda etapa, la herramienta carga el código de tu proyecto al
servicio Agent Engine y lo ejecuta en el entorno alojado

!!! warning "Importante"
    *Asegúrate de que tu proyecto de destino de despliegue de Google Cloud esté establecido como tu ***proyecto
    actual*** antes de realizar estos pasos*. El comando `make backend` usa
    tu proyecto de Google Cloud actualmente establecido cuando realiza un despliegue. Para
    obtener información sobre cómo establecer y verificar tu proyecto actual, consulta
    [Conectar a tu proyecto de Google Cloud](#connect-ad).

Para desplegar tu proyecto ADK en Agent Engine en tu proyecto de Google Cloud:

1.  En una ventana de terminal, asegúrate de estar en el directorio padre (por ejemplo,
    `your-project-directory/`) que contiene tu carpeta de agente.

1.  Despliega el código del proyecto local actualizado en el entorno de desarrollo de Google Cloud,
    ejecutando el siguiente comando make de ASP:

    ```shell
    make backend
    ```

Una vez que este proceso se complete con éxito, deberías poder interactuar con
el agente que se ejecuta en Google Cloud Agent Engine. Para obtener detalles sobre cómo probar el
agente desplegado, consulta
[Probar agente desplegado](/adk-docs/deploy/agent-engine/test/).

### Cambios en tu proyecto ADK {#adk-asp-changes}

Las herramientas ASP agregan más archivos a tu proyecto para el despliegue. El procedimiento
a continuación hace una copia de seguridad de tus archivos de proyecto existentes antes de modificarlos. Esta guía
usa el proyecto
[multi_tool_agent](https://github.com/google/adk-docs/tree/main/examples/python/snippets/get-started/multi_tool_agent)
como ejemplo de referencia. El proyecto original tiene la siguiente estructura de archivos
para comenzar:

```
multi_tool_agent/
├─ __init__.py
├─ agent.py
└─ .env
```

Después de ejecutar el comando enhance de ASP para agregar información de despliegue de Agent Engine,
la nueva estructura es la siguiente:

```
multi-tool-agent/
├─ app/                 # Core application code
│   ├─ agent.py         # Main agent logic
│   ├─ agent_engine_app.py # Agent Engine application logic
│   └─ utils/           # Utility functions and helpers
├─ .cloudbuild/         # CI/CD pipeline configurations for Google Cloud Build
├─ deployment/          # Infrastructure and deployment scripts
├─ notebooks/           # Jupyter notebooks for prototyping and evaluation
├─ tests/               # Unit, integration, and load tests
├─ Makefile             # Makefile for common commands
├─ GEMINI.md            # AI-assisted development guide
└─ pyproject.toml       # Project dependencies and configuration
```

Consulta el archivo *README.md* en tu carpeta de proyecto ADK actualizada para obtener más información.
Para obtener más información sobre el uso de Agent Starter Pack, consulta la
[guía de desarrollo](https://googlecloudplatform.github.io/agent-starter-pack/guide/development-guide.html).

## Probar agentes desplegados

Después de completar el despliegue de tu agente ADK, debes probar el flujo de trabajo en
su nuevo entorno alojado. Para obtener más información sobre cómo probar un agente ADK
desplegado en Agent Engine, consulta
[Probar agentes desplegados en Agent Engine](/adk-docs/deploy/agent-engine/test/).