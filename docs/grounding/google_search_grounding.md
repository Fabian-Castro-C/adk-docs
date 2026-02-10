# Comprendiendo Google Search Grounding

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

[Google Search Grounding tool](/tools/gemini-api/google-search/) es una característica poderosa en el Agent Development Kit (ADK) que permite a los agentes de IA acceder a información autorizada y en tiempo real desde la web. Al conectar tus agentes a Google Search, puedes proporcionar a los usuarios respuestas actualizadas respaldadas por fuentes confiables.

Esta característica es particularmente valiosa para consultas que requieren información actual como actualizaciones del clima, eventos de noticias, precios de acciones, o cualquier hecho que pueda haber cambiado desde la fecha límite de los datos de entrenamiento del modelo. Cuando tu agente determina que se necesita información externa, automáticamente realiza búsquedas web e incorpora los resultados en su respuesta con la atribución adecuada.

## Lo que Aprenderás

En esta guía, descubrirás:

- **Configuración Rápida**: Cómo crear y ejecutar un agente habilitado para Google Search desde cero
- **Arquitectura de Grounding**: El flujo de datos y proceso técnico detrás del grounding web
- **Estructura de Respuesta**: Cómo interpretar respuestas fundamentadas y sus metadatos
- **Mejores Prácticas**: Directrices para mostrar resultados de búsqueda y citaciones a los usuarios

### Recurso adicional

Como recurso adicional, el [Deep Search Agent Development Kit (ADK) Quickstart](https://github.com/google/adk-samples/tree/main/python/agents/deep-search) tiene un uso práctico del Google Search grounding como ejemplo de aplicación full stack.

## Inicio Rápido de Google Search Grounding

Este inicio rápido te guía a través de la creación de un agente ADK con la característica de Google Search grounding. Este inicio rápido asume un IDE local (VS Code o PyCharm, etc.) con Python 3.10+ y acceso a terminal.

### 1. Configurar Entorno e Instalar ADK { #set-up-environment-install-adk }

A continuación se muestran los pasos para configurar tu entorno e instalar el ADK tanto para proyectos Python como TypeScript.

=== "Python"

    Crear y Activar Entorno Virtual:

    ```bash
    # Crear
    python -m venv .venv

    # Activar (cada nueva terminal)
    # macOS/Linux: source .venv/bin/activate
    # Windows CMD: .venv\Scripts\activate.bat
    # Windows PowerShell: .venv\Scripts\Activate.ps1
    ```

    Instalar ADK:

    ```bash
    pip install google-adk
    ```

=== "TypeScript"

    Crear un nuevo proyecto Node.js:
    ```bash
    npm init -y
    ```

    Instalar ADK:
    ```bash
    npm install @google/adk
    ```

### 2. Crear Proyecto de Agente { #create-agent-project }

Bajo un directorio de proyecto, ejecuta los siguientes comandos:

=== "OS X &amp; Linux"
    ```bash
    # Paso 1: Crear un nuevo directorio para tu agente
    mkdir google_search_agent

    # Paso 2: Crear __init__.py para el agente
    echo "from . import agent" > google_search_agent/__init__.py

    # Paso 3: Crear un agent.py (la definición del agente) y .env (config de autenticación de Gemini)
    touch google_search_agent/agent.py .env
    ```

=== "Windows"
    ```shell
    # Paso 1: Crear un nuevo directorio para tu agente
    mkdir google_search_agent

    # Paso 2: Crear __init__.py para el agente
    echo "from . import agent" > google_search_agent/__init__.py

    # Paso 3: Crear un agent.py (la definición del agente) y .env (config de autenticación de Gemini)
    type nul > google_search_agent\agent.py
    type nul > google_search_agent\.env
    ```



#### Editar `agent.py` o `agent.ts`

Copia y pega el siguiente código en `agent.py` o `agent.ts`:

=== "Python"

    ```python title="google_search_agent/agent.py"
    from google.adk.agents import Agent
    from google.adk.tools import google_search

    root_agent = Agent(
        name="google_search_agent",
        model="gemini-2.5-flash",
        instruction="Answer questions using Google Search when needed. Always cite sources.",
        description="Professional search assistant with Google Search capabilities",
        tools=[google_search]
    )
    ```

=== "TypeScript"

    ```typescript title="google_search_agent/agent.ts"
    import { LlmAgent, GOOGLE_SEARCH } from '@google/adk';

    const rootAgent = new LlmAgent({
        name: "google_search_agent",
        model: "gemini-2.5-flash",
        instruction: "Answer questions using Google Search when needed. Always cite sources.",
        description: "Professional search assistant with Google Search capabilities",
        tools: [GOOGLE_SEARCH],
    });
    ```

Ahora tendrías la siguiente estructura de directorios:

=== "Python"

    ```console
    my_project/
        google_search_agent/
            __init__.py
            agent.py
        .env
    ```

=== "TypeScript"

    ```console
    my_project/
        google_search_agent/
            agent.ts
        package.json
        tsconfig.json
        .env
    ```

### 3. Elegir una plataforma { #choose-a-platform }

Para ejecutar el agente, necesitas seleccionar una plataforma que el agente usará para llamar al modelo Gemini. Elige una entre Google AI Studio o Vertex AI:

=== "Gemini - Google AI Studio"
    1. Obtén una API key desde [Google AI Studio](https://aistudio.google.com/apikey).
    2. Cuando uses Python, abre el archivo **`.env`** y copia-pega el siguiente código.

        ```env title=".env"
        GOOGLE_GENAI_USE_VERTEXAI=FALSE
        GOOGLE_API_KEY=PASTE_YOUR_ACTUAL_API_KEY_HERE
        ```

    3. Reemplaza `PASTE_YOUR_ACTUAL_API_KEY_HERE` con tu `API KEY` real.

=== "Gemini - Google Cloud Vertex AI"
    1. Necesitas una cuenta existente de
    [Google Cloud](https://cloud.google.com/?e=48754805&hl=en) y un
    proyecto.
        * Configura un
          [proyecto de Google Cloud](https://cloud.google.com/vertex-ai/generative-ai/docs/start/quickstarts/quickstart-multimodal#setup-gcp)
        * Configura el
          [gcloud CLI](https://cloud.google.com/vertex-ai/generative-ai/docs/start/quickstarts/quickstart-multimodal#setup-local)
        * Autentícate en Google Cloud, desde la terminal ejecutando
          `gcloud auth login`.
        * [Habilita la API de Vertex AI](https://console.cloud.google.com/flows/enableapi?apiid=aiplatform.googleapis.com).
    2. Cuando uses Python, abre el archivo **`.env`** y copia-pega el siguiente código y actualiza el ID del proyecto y la ubicación.

        ```env title=".env"
        GOOGLE_GENAI_USE_VERTEXAI=TRUE
        GOOGLE_CLOUD_PROJECT=YOUR_PROJECT_ID
        GOOGLE_CLOUD_LOCATION=LOCATION
        ```

### 4. Ejecutar tu Agente { #run-your-agent }

Hay múltiples formas de interactuar con tu agente:

=== "Dev UI (adk web)"
    Ejecuta el siguiente comando para lanzar la **dev UI**.

    ```shell
    adk web
    ```

    !!!info "Nota para usuarios de Windows"

        Cuando encuentres el `_make_subprocess_transport NotImplementedError`, considera usar `adk web --no-reload` en su lugar.


    **Paso 1:** Abre la URL proporcionada (usualmente `http://localhost:8000` o
    `http://127.0.0.1:8000`) directamente en tu navegador.

    **Paso 2.** En la esquina superior izquierda de la UI, puedes seleccionar tu agente en
    el menú desplegable. Selecciona "google_search_agent".

    !!!note "Solución de problemas"

        Si no ves "google_search_agent" en el menú desplegable, asegúrate de que
        estás ejecutando `adk web` en la **carpeta padre** de la carpeta de tu agente
        (es decir, la carpeta padre de google_search_agent).

    **Paso 3.** Ahora puedes chatear con tu agente usando la caja de texto.

=== "Terminal (adk run)"

    Ejecuta el siguiente comando, para chatear con tu agente Weather.

    ```
    adk run google_search_agent
    ```
    Para salir, usa Cmd/Ctrl+C.

### Prompts de ejemplo para probar

Con esas preguntas, puedes confirmar que el agente está realmente llamando a Google Search
para obtener el clima y la hora más recientes.

* ¿Cuál es el clima en Nueva York?
* ¿Qué hora es en Nueva York?
* ¿Cuál es el clima en París?
* ¿Qué hora es en París?

![Try the agent with adk web](../assets/google_search_grd_adk_web.png)

¡Has creado e interactuado exitosamente con tu agente de Google Search usando ADK!

## Cómo funciona el grounding con Google Search

Grounding es el proceso que conecta tu agente a información en tiempo real desde la web, permitiéndole generar respuestas más precisas y actuales. Cuando el prompt de un usuario requiere información sobre la que el modelo no fue entrenado, o que es sensible al tiempo, el Large Language Model subyacente del agente decide inteligentemente invocar la herramienta google\_search para encontrar los hechos relevantes

### **Diagrama de Flujo de Datos**

Este diagrama ilustra el proceso paso a paso de cómo una consulta de usuario resulta en una respuesta fundamentada.

![](../assets/google_search_grd_dataflow.png)

### **Descripción Detallada**

El agente de grounding utiliza el flujo de datos descrito en el diagrama para recuperar, procesar e incorporar información externa en la respuesta final presentada al usuario.

1. **Consulta del Usuario**: Un usuario final interactúa con tu agente haciendo una pregunta o dando un comando.
2. **Orquestación ADK** : El Agent Development Kit orquesta el comportamiento del agente y pasa el mensaje del usuario al núcleo de tu agente.
3. **Análisis LLM y Llamada a Herramientas** : El LLM del agente (por ejemplo, un modelo Gemini) analiza el prompt. Si determina que se requiere información externa y actualizada, activa el mecanismo de grounding llamando a la
    herramienta google\_search. Esto es ideal para responder consultas sobre noticias recientes, clima o hechos no presentes en los datos de entrenamiento del modelo.
4. **Interacción con el Servicio de Grounding** : La herramienta google\_search interactúa con un servicio interno de grounding que formula y envía una o más consultas al Índice de Google Search.
5. **Inyección de Contexto**: El servicio de grounding recupera las páginas web y fragmentos relevantes. Luego integra estos resultados de búsqueda en el contexto del modelo
    antes de que se genere la respuesta final. Este paso crucial permite que el modelo "razone" sobre datos fácticos y en tiempo real.
6. **Generación de Respuesta Fundamentada**: El LLM, ahora informado por los resultados de búsqueda recientes, genera una respuesta que incorpora la información recuperada.
7. **Presentación de Respuesta con Fuentes** : El ADK recibe la respuesta fundamentada final, que incluye las URLs de fuente necesarias y
   groundingMetadata, y la presenta al usuario con atribución. Esto permite a los usuarios finales verificar la información y construye confianza en las respuestas del agente.

### Comprendiendo la respuesta de grounding con Google Search

Cuando el agente utiliza Google Search para fundamentar una respuesta, devuelve un conjunto detallado de información que incluye no solo el texto de la respuesta final sino también las fuentes que utilizó para generar esa respuesta. Estos metadatos son cruciales para verificar la respuesta y para proporcionar atribución a las fuentes originales.

#### **Ejemplo de una Respuesta Fundamentada**

Lo siguiente es un ejemplo del objeto de contenido devuelto por el modelo después de una consulta fundamentada.

**Texto de Respuesta Final:**

```
"Yes, Inter Miami won their last game in the FIFA Club World Cup. They defeated FC Porto 2-1 in their second group stage match. Their first game in the tournament was a 0-0 draw against Al Ahly FC. Inter Miami is scheduled to play their third group stage match against Palmeiras on Monday, June 23, 2025."
```

**Fragmento de Metadatos de Grounding:**

```json
"groundingMetadata": {
  "groundingChunks": [
    { "web": { "title": "mlssoccer.com", "uri": "..." } },
    { "web": { "title": "intermiamicf.com", "uri": "..." } },
    { "web": { "title": "mlssoccer.com", "uri": "..." } }
  ],
  "groundingSupports": [
    {
      "groundingChunkIndices": [0, 1],
      "segment": {
        "startIndex": 65,
        "endIndex": 126,
        "text": "They defeated FC Porto 2-1 in their second group stage match."
      }
    },
    {
      "groundingChunkIndices": [1],
      "segment": {
        "startIndex": 127,
        "endIndex": 196,
        "text": "Their first game in the tournament was a 0-0 draw against Al Ahly FC."
      }
    },
    {
      "groundingChunkIndices": [0, 2],
      "segment": {
        "startIndex": 197,
        "endIndex": 303,
        "text": "Inter Miami is scheduled to play their third group stage match against Palmeiras on Monday, June 23, 2025."
      }
    }
  ],
  "searchEntryPoint": { ... }
}

```

#### **Cómo Interpretar la Respuesta**

Los metadatos proporcionan un vínculo entre el texto generado por el modelo y las fuentes que lo respaldan. Aquí hay un desglose paso a paso:

1. **groundingChunks**: Esta es una lista de las páginas web que el modelo consultó. Cada fragmento contiene el título de la página web y un uri que enlaza a la fuente.
2. **groundingSupports**: Esta lista conecta oraciones específicas en la respuesta final de vuelta a los groundingChunks.
   * **segment**: Este objeto identifica una porción específica del texto de respuesta final, definida por su startIndex, endIndex, y el texto en sí.
   * **groundingChunkIndices**: Este array contiene los números de índice que corresponden a las fuentes listadas en los groundingChunks. Por ejemplo, la oración "They defeated FC Porto 2-1..." está respaldada por información de groundingChunks en los índices 0 y 1 (ambos de mlssoccer.com e intermiamicf.com).

### Cómo mostrar respuestas de grounding con Google Search

Una parte crítica del uso de grounding es mostrar correctamente la información, incluyendo citaciones y sugerencias de búsqueda, al usuario final. Esto construye confianza y permite a los usuarios verificar la información.

![Responnses from Google Search](../assets/google_search_grd_resp.png)

#### **Mostrando Sugerencias de Búsqueda**

El objeto `searchEntryPoint` en el `groundingMetadata` contiene HTML pre-formateado para mostrar sugerencias de consultas de búsqueda. Como se ve en la imagen de ejemplo, estas se renderizan típicamente como chips clicables que permiten al usuario explorar temas relacionados.

**HTML Renderizado desde searchEntryPoint:** Los metadatos proporcionan el HTML y CSS necesarios para renderizar la barra de sugerencias de búsqueda, que incluye el logo de Google y chips para consultas relacionadas como "When is the next FIFA Club World Cup" y "Inter Miami FIFA Club World Cup history". Integrar este HTML directamente en el front end de tu aplicación mostrará las sugerencias como se pretende.

Para más información, consulta [using Google Search Suggestions](https://cloud.google.com/vertex-ai/generative-ai/docs/grounding/grounding-search-suggestions) en la documentación de Vertex AI.

## Resumen

Google Search Grounding transforma a los agentes de IA de repositorios de conocimiento estáticos en asistentes dinámicos conectados a la web capaces de proporcionar información precisa en tiempo real. Al integrar esta característica en tus agentes ADK, les permites:

- Acceder a información actual más allá de sus datos de entrenamiento
- Proporcionar atribución de fuentes para transparencia y confianza
- Entregar respuestas completas con hechos verificables
- Mejorar la experiencia del usuario con sugerencias de búsqueda relevantes

El proceso de grounding conecta sin problemas las consultas de los usuarios al vasto índice de búsqueda de Google, enriqueciendo las respuestas con contexto actualizado mientras mantiene el flujo conversacional. Con la implementación y visualización adecuada de respuestas fundamentadas, tus agentes se convierten en herramientas poderosas para el descubrimiento de información y la toma de decisiones.