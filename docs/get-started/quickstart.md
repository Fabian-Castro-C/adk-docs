# Construye un agente multi-herramienta

Esta guía de inicio rápido te orienta a través de la instalación del Agent Development Kit (ADK), la configuración de un agente básico con múltiples herramientas, y su ejecución local, ya sea en la terminal o en la interfaz de desarrollo interactiva basada en navegador.

<!-- <img src="../../assets/quickstart.png" alt="Quickstart setup"> -->

Esta guía de inicio rápido asume un IDE local (VS Code, PyCharm, IntelliJ IDEA, etc.) con Python 3.10+ o Java 17+ y acceso a terminal. Este método ejecuta la aplicación completamente en tu máquina y es recomendado para desarrollo interno.

## 1. Configura el Entorno e Instala ADK { #set-up-environment-install-adk }

=== "Python"

    Crea y Activa el Entorno Virtual (Recomendado):

    ```bash
    # Crear
    python -m venv .venv
    # Activar (cada nueva terminal)
    # macOS/Linux: source .venv/bin/activate
    # Windows CMD: .venv\Scripts\activate.bat
    # Windows PowerShell: .venv\Scripts\Activate.ps1
    ```

    Instala ADK:

    ```bash
    pip install google-adk
    ```

=== "TypeScript"

    Crea un nuevo directorio de proyecto, inicialízalo e instala las dependencias:

    ```bash
    mkdir my-adk-agent
    cd my-adk-agent
    npm init -y
    npm install @google/adk @google/adk-devtools
    npm install -D typescript
    ```

    Crea un archivo `tsconfig.json` con el siguiente contenido. Esta configuración asegura que tu proyecto maneje correctamente los módulos modernos de Node.js.

    ```json title="tsconfig.json"
    {
      "compilerOptions": {
        "target": "es2020",
        "module": "nodenext",
        "moduleResolution": "nodenext",
        "esModuleInterop": true,
        "strict": true,
        "skipLibCheck": true,
        // establecer en false para permitir sintaxis de módulos CommonJS:
        "verbatimModuleSyntax": false
      }
    }
    ```

=== "Java"

    Para instalar ADK y configurar el entorno, procede con los siguientes pasos.

## 2. Crea el Proyecto del Agente { #create-agent-project }

### Estructura del proyecto

=== "Python"

    Necesitarás crear la siguiente estructura de proyecto:

    ```console
    parent_folder/
        multi_tool_agent/
            __init__.py
            agent.py
            .env
    ```

    Crea la carpeta `multi_tool_agent`:

    ```bash
    mkdir multi_tool_agent/
    ```

    !!! info "Nota para usuarios de Windows"

        Al usar ADK en Windows para los próximos pasos, recomendamos crear archivos Python usando el Explorador de archivos o un IDE porque los siguientes comandos (`mkdir`, `echo`) típicamente generan archivos con bytes nulos y/o codificación incorrecta.

    ### `__init__.py`

    Ahora crea un archivo `__init__.py` en la carpeta:

    ```shell
    echo "from . import agent" > multi_tool_agent/__init__.py
    ```

    Tu `__init__.py` ahora debería verse así:

    ```python title="multi_tool_agent/__init__.py"
    --8<-- "examples/python/snippets/get-started/multi_tool_agent/__init__.py"
    ```

    ### `agent.py`

    Crea un archivo `agent.py` en la misma carpeta:

    === "OS X &amp; Linux"
        ```shell
        touch multi_tool_agent/agent.py
        ```

    === "Windows"
        ```shell
        type nul > multi_tool_agent/agent.py
        ```

    Copia y pega el siguiente código en `agent.py`:

    ```python title="multi_tool_agent/agent.py"
    --8<-- "examples/python/snippets/get-started/multi_tool_agent/agent.py"
    ```

    ### `.env`

    Crea un archivo `.env` en la misma carpeta:

    === "OS X &amp; Linux"
        ```shell
        touch multi_tool_agent/.env
        ```

    === "Windows"
        ```shell
        type nul > multi_tool_agent\.env
        ```

    Más instrucciones sobre este archivo se describen en la siguiente sección sobre [Configura el modelo](#set-up-the-model).

=== "TypeScript"

    Necesitarás crear la siguiente estructura de proyecto en tu directorio `my-adk-agent`:

    ```console
    my-adk-agent/
        agent.ts
        .env
        package.json
        tsconfig.json
    ```

    ### `agent.ts`

    Crea un archivo `agent.ts` en tu carpeta de proyecto:

    === "OS X &amp; Linux"
        ```shell
        touch agent.ts
        ```

    === "Windows"
        ```shell
        type nul > agent.ts
        ```

    Copia y pega el siguiente código en `agent.ts`:

    ```typescript title="agent.ts"
    --8<-- "examples/typescript/snippets/get-started/multi_tool_agent/agent.ts"
    ```

    ### `.env`

    Crea un archivo `.env` en la misma carpeta:

    === "OS X &amp; Linux"
        ```shell
        touch .env
        ```

    === "Windows"
        ```shell
        type nul > .env
        ```

    Más instrucciones sobre este archivo se describen en la siguiente sección sobre [Configura el modelo](#set-up-the-model).

=== "Java"

    Los proyectos Java generalmente presentan la siguiente estructura de proyecto:

    ```console
    project_folder/
    ├── pom.xml (or build.gradle)
    ├── src/
    ├── └── main/
    │       └── java/
    │           └── agents/
    │               └── multitool/
    └── test/
    ```

    ### Crea `MultiToolAgent.java`

    Crea un archivo fuente `MultiToolAgent.java` en el paquete `agents.multitool` en el directorio `src/main/java/agents/multitool/`.

    Copia y pega el siguiente código en `MultiToolAgent.java`:

    ```java title="agents/multitool/MultiToolAgent.java"
    --8<-- "examples/java/cloud-run/src/main/java/agents/multitool/MultiToolAgent.java:full_code"
    ```

![intro_components.png](../assets/quickstart-flow-tool.png)

## 3. Configura el modelo { #set-up-the-model }

La capacidad de tu agente para comprender las solicitudes de los usuarios y generar respuestas es impulsada por un Modelo de Lenguaje Grande (LLM). Tu agente necesita realizar llamadas seguras a este servicio LLM externo, lo cual **requiere credenciales de autenticación**. Sin autenticación válida, el servicio LLM denegará las solicitudes del agente, y el agente no podrá funcionar.

!!!tip "Guía de Autenticación del Modelo"
    Para una guía detallada sobre cómo autenticarse con diferentes modelos, consulta la [Guía de Autenticación](/agents/models/google-gemini#google-ai-studio).
    Este es un paso crítico para asegurar que tu agente pueda realizar llamadas al servicio LLM.

=== "Gemini - Google AI Studio"
    1. Obtén una clave API de [Google AI Studio](https://aistudio.google.com/apikey).
    2. Al usar Python, abre el archivo **`.env`** ubicado dentro de (`multi_tool_agent/`) y copia-pega el siguiente código.

        ```env title="multi_tool_agent/.env"
        GOOGLE_GENAI_USE_VERTEXAI=FALSE
        GOOGLE_API_KEY=PASTE_YOUR_ACTUAL_API_KEY_HERE
        ```

        Al usar Java, define las variables de entorno:

        ```console title="terminal"
        export GOOGLE_GENAI_USE_VERTEXAI=FALSE
        export GOOGLE_API_KEY=PASTE_YOUR_ACTUAL_API_KEY_HERE
        ```

        Al usar TypeScript, el archivo `.env` se carga automáticamente por la línea `import 'dotenv/config';` al inicio de tu archivo `agent.ts`.

        ```env title=""multi_tool_agent/.env"
        GOOGLE_GENAI_USE_VERTEXAI=FALSE
        GOOGLE_GENAI_API_KEY=PASTE_YOUR_ACTUAL_API_KEY_HERE
        ```

    3. Reemplaza `PASTE_YOUR_ACTUAL_API_KEY_HERE` con tu `API KEY` real.

=== "Gemini - Google Cloud Vertex AI"
    1. Configura un [proyecto de Google Cloud](https://cloud.google.com/vertex-ai/generative-ai/docs/start/quickstarts/quickstart-multimodal#setup-gcp) y [habilita la API de Vertex AI](https://console.cloud.google.com/flows/enableapi?apiid=aiplatform.googleapis.com).
    2. Configura la [CLI de gcloud](https://cloud.google.com/vertex-ai/generative-ai/docs/start/quickstarts/quickstart-multimodal#setup-local).
    3. Autentícate en Google Cloud desde la terminal ejecutando `gcloud auth application-default login`.
    4. Al usar Python, abre el archivo **`.env`** ubicado dentro de (`multi_tool_agent/`). Copia-pega el siguiente código y actualiza el ID del proyecto y la ubicación.

        ```env title="multi_tool_agent/.env"
        GOOGLE_GENAI_USE_VERTEXAI=TRUE
        GOOGLE_CLOUD_PROJECT=YOUR_PROJECT_ID
        GOOGLE_CLOUD_LOCATION=LOCATION
        ```

        Al usar Java, define las variables de entorno:

        ```console title="terminal"
        export GOOGLE_GENAI_USE_VERTEXAI=TRUE
        export GOOGLE_CLOUD_PROJECT=YOUR_PROJECT_ID
        export GOOGLE_CLOUD_LOCATION=LOCATION
        ```

        Al usar TypeScript, el archivo `.env` se carga automáticamente por la línea `import 'dotenv/config';` al inicio de tu archivo `agent.ts`.

        ```env title=".env"
        GOOGLE_GENAI_USE_VERTEXAI=TRUE
        GOOGLE_CLOUD_PROJECT=YOUR_PROJECT_ID
        GOOGLE_CLOUD_LOCATION=LOCATION
        ```

=== "Gemini - Google Cloud Vertex AI con Modo Express"
    1. ¡Puedes registrarte para un proyecto gratuito de Google Cloud y usar Gemini gratis con una cuenta elegible!
        * Configura un
          [proyecto de Google Cloud con Modo Express de Vertex AI](https://cloud.google.com/vertex-ai/generative-ai/docs/start/express-mode/overview)
        * Obtén una clave API de tu proyecto en modo Express. Esta clave puede usarse con ADK para usar modelos Gemini gratis, así como acceso a servicios de Agent Engine.
    2. Al usar Python, abre el archivo **`.env`** ubicado dentro de (`multi_tool_agent/`). Copia-pega el siguiente código y actualiza el ID del proyecto y la ubicación.

        ```env title="multi_tool_agent/.env"
        GOOGLE_GENAI_USE_VERTEXAI=TRUE
        GOOGLE_API_KEY=PASTE_YOUR_ACTUAL_EXPRESS_MODE_API_KEY_HERE
        ```

        Al usar Java, define las variables de entorno:

        ```console title="terminal"
        export GOOGLE_GENAI_USE_VERTEXAI=TRUE
        export GOOGLE_API_KEY=PASTE_YOUR_ACTUAL_EXPRESS_MODE_API_KEY_HERE
        ```

        Al usar TypeScript, el archivo `.env` se carga automáticamente por la línea `import 'dotenv/config';` al inicio de tu archivo `agent.ts`.

        ```env title=".env"
        GOOGLE_GENAI_USE_VERTEXAI=TRUE
        GOOGLE_GENAI_API_KEY=PASTE_YOUR_ACTUAL_EXPRESS_MODE_API_KEY_HERE
        ```

## 4. Ejecuta tu Agente { #run-your-agent }

=== "Python"

    Usando la terminal, navega al directorio padre de tu proyecto de agente (ej. usando `cd ..`):

    ```console
    parent_folder/      <-- navega a este directorio
        multi_tool_agent/
            __init__.py
            agent.py
            .env
    ```

    Hay múltiples formas de interactuar con tu agente:

    === "Dev UI (adk web)"

        !!! success "Configuración de Autenticación para Usuarios de Vertex AI"
            Si seleccionaste **"Gemini - Google Cloud Vertex AI"** en el paso anterior, debes autenticarte con Google Cloud antes de iniciar la interfaz de desarrollo.

            Ejecuta este comando y sigue las instrucciones:
            ```bash
            gcloud auth application-default login
            ```

            **Nota:** Omite este paso si estás usando "Gemini - Google AI Studio".

        Ejecuta el siguiente comando para iniciar la **interfaz de desarrollo**.

        ```shell
        adk web
        ```

        !!! warning "Precaución: ADK Web solo para desarrollo"

            ADK Web ***no está destinado para uso en despliegues de producción***. Debes usar ADK Web solo para propósitos de desarrollo y depuración.

        !!!info "Nota para usuarios de Windows"

            Al encontrar el error `_make_subprocess_transport NotImplementedError`, considera usar `adk web --no-reload` en su lugar.


        **Paso 1:** Abre la URL proporcionada (usualmente `http://localhost:8000` o `http://127.0.0.1:8000`) directamente en tu navegador.

        **Paso 2.** En la esquina superior izquierda de la interfaz, puedes seleccionar tu agente en el menú desplegable. Selecciona "multi_tool_agent".

        !!!note "Solución de problemas"

            Si no ves "multi_tool_agent" en el menú desplegable, asegúrate de estar ejecutando `adk web` en la **carpeta padre** de tu carpeta de agente (es decir, la carpeta padre de multi_tool_agent).

        **Paso 3.** Ahora puedes chatear con tu agente usando el cuadro de texto:

        ![adk-web-dev-ui-chat.png](../assets/adk-web-dev-ui-chat.png)


        **Paso 4.** Usando la pestaña `Events` a la izquierda, puedes inspeccionar llamadas de función individuales, respuestas y respuestas del modelo haciendo clic en las acciones:

        ![adk-web-dev-ui-function-call.png](../assets/adk-web-dev-ui-function-call.png)

        En la pestaña `Events`, también puedes hacer clic en el botón `Trace` para ver los registros de seguimiento de cada evento que muestra la latencia de cada llamada de función:

        ![adk-web-dev-ui-trace.png](../assets/adk-web-dev-ui-trace.png)

        **Paso 5.** También puedes habilitar tu micrófono y hablar con tu agente:

        !!!note "Soporte de modelo para streaming de voz/video"

            Para usar streaming de voz/video en ADK, necesitarás usar modelos Gemini que soporten la API Live. Puedes encontrar el/los **ID(s) de modelo** que soportan la API Live de Gemini en la documentación:

            - [Google AI Studio: API Live de Gemini](https://ai.google.dev/gemini-api/docs/models#live-api)
            - [Vertex AI: API Live de Gemini](https://cloud.google.com/vertex-ai/generative-ai/docs/live-api)

            Luego puedes reemplazar la cadena `model` en `root_agent` en el archivo `agent.py` que creaste anteriormente ([saltar a la sección](#agentpy)). Tu código debería verse algo así:

            ```py
            root_agent = Agent(
                name="weather_time_agent",
                model="replace-me-with-model-id", #ej. gemini-2.0-flash-live-001
                ...
            ```

        ![adk-web-dev-ui-audio.png](../assets/adk-web-dev-ui-audio.png)

    === "Terminal (adk run)"

        !!! tip

            Al usar `adk run` puedes inyectar prompts en el agente para comenzar canalizando texto al comando de esta manera:

            ```shell
            echo "Please start by listing files" | adk run file_listing_agent
            ```

        Ejecuta el siguiente comando para chatear con tu agente Weather.

        ```
        adk run multi_tool_agent
        ```

        ![adk-run.png](../assets/adk-run.png)

        Para salir, usa Cmd/Ctrl+C.

    === "API Server (adk api_server)"

        `adk api_server` te permite crear un servidor FastAPI local en un solo comando, permitiéndote probar solicitudes cURL locales antes de desplegar tu agente.

        ![adk-api-server.png](../assets/adk-api-server.png)

        Para aprender cómo usar `adk api_server` para pruebas, consulta la [documentación sobre el uso del servidor API](/runtime/api-server/).

=== "TypeScript"

    Usando la terminal, navega al directorio de tu proyecto de agente:

    ```console
    my-adk-agent/      <-- navega a este directorio
        agent.ts
        .env
        package.json
        tsconfig.json
    ```

    Hay múltiples formas de interactuar con tu agente:

    === "Dev UI (adk web)"

        Ejecuta el siguiente comando para iniciar la **interfaz de desarrollo**.

        ```shell
        npx adk web
        ```

        **Paso 1:** Abre la URL proporcionada (usualmente `http://localhost:8000` o `http://127.0.0.1:8000`) directamente en tu navegador.

        **Paso 2.** En la esquina superior izquierda de la interfaz, selecciona tu agente del menú desplegable. Los agentes se listan por sus nombres de archivo, por lo que deberías seleccionar "agent".

        !!!note "Solución de problemas"

            Si no ves "agent" en el menú desplegable, asegúrate de estar ejecutando `npx adk web` en el directorio que contiene tu archivo `agent.ts`.

        **Paso 3.** Ahora puedes chatear con tu agente usando el cuadro de texto:

        ![adk-web-dev-ui-chat.png](../assets/adk-web-dev-ui-chat.png)


        **Paso 4.** Usando la pestaña `Events` a la izquierda, puedes inspeccionar llamadas de función individuales, respuestas y respuestas del modelo haciendo clic en las acciones:

        ![adk-web-dev-ui-function-call.png](../assets/adk-web-dev-ui-function-call.png)

        En la pestaña `Events`, también puedes hacer clic en el botón `Trace` para ver los registros de seguimiento de cada evento que muestra la latencia de cada llamada de función:

        ![adk-web-dev-ui-trace.png](../assets/adk-web-dev-ui-trace.png)

    === "Terminal (adk run)"

        Ejecuta el siguiente comando para chatear con tu agente.

        ```
        npx adk run agent.ts
        ```

        ![adk-run.png](../assets/adk-run.png)

        Para salir, usa Cmd/Ctrl+C.

    === "API Server (adk api_server)"

        `npx adk api_server` te permite crear un servidor Express.js local en un solo comando, permitiéndote probar solicitudes cURL locales antes de desplegar tu agente.

        ![adk-api-server.png](../assets/adk-api-server.png)

        Para aprender cómo usar `api_server` para pruebas, consulta la [documentación sobre pruebas](/runtime/api-server/).

=== "Java"

    Usando la terminal, navega al directorio padre de tu proyecto de agente (ej. usando `cd ..`):

    ```console
    project_folder/                <-- navega a este directorio
    ├── pom.xml (or build.gradle)
    ├── src/
    ├── └── main/
    │       └── java/
    │           └── agents/
    │               └── multitool/
    │                   └── MultiToolAgent.java
    └── test/
    ```

    === "Dev UI"

        Ejecuta el siguiente comando desde la terminal para iniciar la interfaz de desarrollo.

        **NO cambies el nombre de la clase principal del servidor Dev UI.**

        ```console title="terminal"
        mvn exec:java \
            -Dexec.mainClass="com.google.adk.web.AdkWebServer" \
            -Dexec.args="--adk.agents.source-dir=src/main/java" \
            -Dexec.classpathScope="compile"
        ```

        **Paso 1:** Abre la URL proporcionada (usualmente `http://localhost:8080` o `http://127.0.0.1:8080`) directamente en tu navegador.

        **Paso 2.** En la esquina superior izquierda de la interfaz, puedes seleccionar tu agente en el menú desplegable. Selecciona "multi_tool_agent".

        !!!note "Solución de problemas"

            Si no ves "multi_tool_agent" en el menú desplegable, asegúrate de estar ejecutando el comando `mvn` en la ubicación donde se encuentra tu código fuente Java (usualmente `src/main/java`).

        **Paso 3.** Ahora puedes chatear con tu agente usando el cuadro de texto:

        ![adk-web-dev-ui-chat.png](../assets/adk-web-dev-ui-chat.png)

        **Paso 4.** También puedes inspeccionar llamadas de función individuales, respuestas y respuestas del modelo haciendo clic en las acciones:

        ![adk-web-dev-ui-function-call.png](../assets/adk-web-dev-ui-function-call.png)

        !!! warning "Precaución: ADK Web solo para desarrollo"

            ADK Web ***no está destinado para uso en despliegues de producción***. Debes usar ADK Web solo para propósitos de desarrollo y depuración.

    === "Maven"

        Con Maven, ejecuta el método `main()` de tu clase Java con el siguiente comando:

        ```console title="terminal"
        mvn compile exec:java -Dexec.mainClass="agents.multitool.MultiToolAgent"
        ```

    === "Gradle"

        Con Gradle, el archivo de construcción `build.gradle` o `build.gradle.kts` debe tener el siguiente plugin de Java en su sección `plugins`:

        ```groovy
        plugins {
            id('java')
            // otros plugins
        }
        ```

        Luego, en otra parte del archivo de construcción, al nivel superior, crea una nueva tarea para ejecutar el método `main()` de tu agente:

        ```groovy
        tasks.register('runAgent', JavaExec) {
            classpath = sourceSets.main.runtimeClasspath
            mainClass = 'agents.multitool.MultiToolAgent'
        }
        ```

        Finalmente, en la línea de comandos, ejecuta el siguiente comando:

        ```console
        gradle runAgent
        ```

### 📝 Ejemplos de prompts para probar

* ¿Cuál es el clima en Nueva York?
* ¿Qué hora es en Nueva York?
* ¿Cuál es el clima en París?
* ¿Qué hora es en París?

## 🎉 ¡Felicitaciones!

¡Has creado e interactuado exitosamente con tu primer agente usando ADK!

---

## 🛣️ Próximos pasos

* **Ve al tutorial**: Aprende cómo agregar memoria, sesión, estado a tu agente: [tutorial](../tutorials/index.md).
* **Profundiza en la configuración avanzada:** Explora la sección de [configuración](installation.md) para inmersiones más profundas en estructura de proyecto, configuración y otras interfaces.
* **Comprende los Conceptos Fundamentales:** Aprende sobre [conceptos de agentes](../agents/index.md).