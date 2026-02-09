# Construir agentes con Agent Config

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v1.11.0</span><span class="lst-preview">Experimental</span>
</div>

La característica Agent Config de ADK te permite construir un flujo de trabajo de ADK sin escribir
código. Un Agent Config utiliza un archivo de texto en formato YAML con una breve descripción del
agente, permitiendo que casi cualquier persona pueda ensamblar y ejecutar un agente ADK. El
siguiente es un ejemplo simple de una definición básica de Agent Config:

```
name: assistant_agent
model: gemini-2.5-flash
description: A helper agent that can answer users' questions.
instruction: You are an agent to help answer users' various questions.
```

Puedes usar archivos Agent Config para construir agentes más complejos que pueden
incorporar Funciones, Herramientas, Sub-Agentes y más. Esta página describe cómo
construir y ejecutar flujos de trabajo de ADK con la característica Agent Config. Para información detallada
sobre la sintaxis y configuraciones soportadas por el formato Agent Config,
consulta la
[referencia de sintaxis de Agent Config](/adk-docs/api-reference/agentconfig/).

!!! example "Experimental"
    La característica Agent Config es experimental y tiene algunas
    [limitaciones conocidas](#known-limitations). ¡Agradecemos tus
    [comentarios](https://github.com/google/adk-python/issues/new?template=feature_request.md&labels=agent%20config)!

## Comenzar

Esta sección describe cómo configurar y comenzar a construir agentes con el ADK y
la característica Agent Config, incluyendo la configuración de instalación, construcción de un agente, y
ejecución de tu agente.

### Configuración

Necesitas instalar las bibliotecas del Google Agent Development Kit, y proporcionar una
clave de acceso para un modelo de IA generativa como la API de Gemini. Esta sección proporciona
detalles sobre lo que debes instalar y configurar antes de poder ejecutar agentes con
los archivos Agent Config.

!!! note
    La característica Agent Config actualmente solo soporta modelos Gemini. Para más
    información sobre restricciones funcionales adicionales, consulta
    [Limitaciones conocidas](#known-limitations).

Para configurar ADK para usar con Agent Config:

1.  Instala las bibliotecas Python de ADK siguiendo las
    instrucciones de [Instalación](/adk-docs/get-started/installation/#python).
    *Python es actualmente requerido.* Para más información, consulta las
    [Limitaciones conocidas](#known-limitations).
1.  Verifica que ADK esté instalado ejecutando el siguiente comando en tu
    terminal:

        adk --version

    Este comando debería mostrar la versión de ADK que tienes instalada.

!!! Tip
    Si el comando `adk` falla al ejecutarse y la versión no aparece en el paso 2, asegúrate
    de que tu entorno Python esté activo. Ejecuta `source .venv/bin/activate` en
    tu terminal en Mac y Linux. Para comandos de otras plataformas, consulta la
    página de [Instalación](/adk-docs/get-started/installation/#python).

### Construir un agente

Construyes un agente con Agent Config usando el comando `adk create` para crear
los archivos del proyecto para un agente, y luego editando el archivo `root_agent.yaml` que
genera para ti.

Para crear un proyecto ADK para usar con Agent Config:

1.  En tu ventana de terminal, ejecuta el siguiente comando para crear un
    agente basado en config:

        adk create --type=config my_agent

    Este comando genera una carpeta `my_agent/`, que contiene un
    archivo `root_agent.yaml` y un archivo `.env`.

1.  En el archivo `my_agent/.env`, establece variables de entorno para que tu agente
    acceda a modelos de IA generativa y otros servicios:

    1.  Para acceso al modelo Gemini a través de la API de Google, agrega una línea al
        archivo con tu clave de API:

            GOOGLE_GENAI_USE_VERTEXAI=0
            GOOGLE_API_KEY=<your-Google-Gemini-API-key>

        Puedes obtener una clave de API desde la página de
        [Claves de API](https://aistudio.google.com/app/apikey) de Google AI Studio.

    1.  Para acceso al modelo Gemini a través de Google Cloud, agrega estas líneas al archivo:

            GOOGLE_GENAI_USE_VERTEXAI=1
            GOOGLE_CLOUD_PROJECT=<your_gcp_project>
            GOOGLE_CLOUD_LOCATION=us-central1

        Para información sobre cómo crear un Proyecto Cloud, consulta la documentación de Google Cloud
        para
        [Crear y administrar proyectos](https://cloud.google.com/resource-manager/docs/creating-managing-projects).

1.  Usando un editor de texto, edita el archivo Agent Config
    `my_agent/root_agent.yaml`, como se muestra a continuación:

```
# yaml-language-server: $schema=https://raw.githubusercontent.com/google/adk-python/refs/heads/main/src/google/adk/agents/config_schemas/AgentConfig.json
name: assistant_agent
model: gemini-2.5-flash
description: A helper agent that can answer users' questions.
instruction: You are an agent to help answer users' various questions.
```

Puedes descubrir más opciones de configuración para tu archivo de
configuración de agente `root_agent.yaml` consultando el
[repositorio de ejemplos](https://github.com/search?q=repo%3Agoogle%2Fadk-python+path%3A%2F%5Econtributing%5C%2Fsamples%5C%2F%2F+.yaml&type=code) de ADK
o la referencia de
[sintaxis de Agent Config](/adk-docs/api-reference/agentconfig/).

### Ejecutar el agente

Una vez que hayas completado la edición de tu Agent Config, puedes ejecutar tu agente usando
la interfaz web, ejecución en terminal de línea de comandos, o modo de servidor API.

Para ejecutar tu agente definido con Agent Config:

1.  En tu terminal, navega al directorio `my_agent/` que contiene el
    archivo `root_agent.yaml`.
1.  Escribe uno de los siguientes comandos para ejecutar tu agente:
    -   `adk web` - Ejecuta la interfaz de usuario web para tu agente.
    -   `adk run` - Ejecuta tu agente en la terminal sin una interfaz
        de usuario.
    -   `adk api_server` - Ejecuta tu agente como un servicio que puede ser
        usado por otras aplicaciones.

Para más información sobre las formas de ejecutar tu agente, consulta el tema *Ejecutar Tu Agente*
en el
[Inicio rápido](/adk-docs/get-started/quickstart/#run-your-agent).
Para más información sobre las opciones de línea de comandos de ADK, consulta la
[referencia de CLI de ADK](/adk-docs/api-reference/cli/).

## Ejemplos de configs

Esta sección muestra ejemplos de archivos Agent Config para ayudarte a comenzar a construir
agentes. Para ejemplos adicionales y más completos, consulta el
[repositorio de ejemplos](https://github.com/search?q=repo%3Agoogle%2Fadk-python+path%3A%2F%5Econtributing%5C%2Fsamples%5C%2F%2F+root_agent.yaml&type=code) de ADK.

### Ejemplo de herramienta integrada

El siguiente ejemplo usa una función de herramienta integrada de ADK para usar la búsqueda de Google
para proporcionar funcionalidad al agente. Este agente automáticamente usa la herramienta de
búsqueda para responder a las solicitudes del usuario.

```
# yaml-language-server: $schema=https://raw.githubusercontent.com/google/adk-python/refs/heads/main/src/google/adk/agents/config_schemas/AgentConfig.json
name: search_agent
model: gemini-2.0-flash
description: 'an agent whose job it is to perform Google search queries and answer questions about the results.'
instruction: You are an agent whose job is to perform Google search queries and answer questions about the results.
tools:
  - name: google_search
```

Para más detalles, consulta el código completo de este ejemplo en el
[repositorio de ejemplos de ADK](https://github.com/google/adk-python/blob/main/contributing/samples/tool_builtin_config/root_agent.yaml).

### Ejemplo de herramienta personalizada

El siguiente ejemplo usa una herramienta personalizada construida con código Python y listada en
la sección `tools:` del archivo config. El agente usa esta herramienta para verificar si una
lista de números proporcionados por el usuario son números primos.

```
# yaml-language-server: $schema=https://raw.githubusercontent.com/google/adk-python/refs/heads/main/src/google/adk/agents/config_schemas/AgentConfig.json
agent_class: LlmAgent
model: gemini-2.5-flash
name: prime_agent
description: Handles checking if numbers are prime.
instruction: |
  You are responsible for checking whether numbers are prime.
  When asked to check primes, you must call the check_prime tool with a list of integers.
  Never attempt to determine prime numbers manually.
  Return the prime number results to the root agent.
tools:
  - name: ma_llm.check_prime
```

Para más detalles, consulta el código completo de este ejemplo en el
[repositorio de ejemplos de ADK](https://github.com/google/adk-python/blob/main/contributing/samples/multi_agent_llm_config/prime_agent.yaml).

### Ejemplo de sub-agentes

El siguiente ejemplo muestra un agente definido con dos sub-agentes en la
sección `sub_agents:`, y un ejemplo de herramienta en la sección `tools:` del archivo config.
Este agente determina lo que el usuario quiere, y delega a uno de los
sub-agentes para resolver la solicitud. Los sub-agentes se definen usando archivos
YAML de Agent Config.

```
# yaml-language-server: $schema=https://raw.githubusercontent.com/google/adk-python/refs/heads/main/src/google/adk/agents/config_schemas/AgentConfig.json
agent_class: LlmAgent
model: gemini-2.5-flash
name: root_agent
description: Learning assistant that provides tutoring in code and math.
instruction: |
  You are a learning assistant that helps students with coding and math questions.

  You delegate coding questions to the code_tutor_agent and math questions to the math_tutor_agent.

  Follow these steps:
  1. If the user asks about programming or coding, delegate to the code_tutor_agent.
  2. If the user asks about math concepts or problems, delegate to the math_tutor_agent.
  3. Always provide clear explanations and encourage learning.
sub_agents:
  - config_path: code_tutor_agent.yaml
  - config_path: math_tutor_agent.yaml
```

Para más detalles, consulta el código completo de este ejemplo en el
[repositorio de ejemplos de ADK](https://github.com/google/adk-python/blob/main/contributing/samples/multi_agent_basic_config/root_agent.yaml).

## Desplegar agent configs

Puedes desplegar agentes Agent Config con
[Cloud Run](/adk-docs/deploy/cloud-run/) y
[Agent Engine](/adk-docs/deploy/agent-engine/),
usando el mismo procedimiento que los agentes basados en código. Para más información sobre cómo
preparar y desplegar agentes basados en Agent Config, consulta las
guías de despliegue de
[Cloud Run](/adk-docs/deploy/cloud-run/) y
[Agent Engine](/adk-docs/deploy/agent-engine/).

## Limitaciones conocidas {#known-limitations}

La característica Agent Config es experimental e incluye las siguientes
limitaciones:

-   **Soporte de modelos:** Actualmente solo se soportan modelos Gemini.
    La integración con modelos de terceros está en progreso.
-   **Lenguaje de programación:** La característica Agent Config actualmente soporta
    solo código Python para herramientas y otras funcionalidades que requieren código de programación.
-   **Soporte de herramientas ADK:** Las siguientes herramientas ADK son soportadas por la
    característica Agent Config, pero *no todas las herramientas están completamente soportadas*:
    -   `google_search`
    -   `load_artifacts`
    -   `url_context`
    -   `exit_loop`
    -   `preload_memory`
    -   `get_user_choice`
    -   `enterprise_web_search`
    -   `load_web_page`: Requiere una ruta completamente calificada para acceder a páginas
        web.
-   **Soporte de tipos de agente:** Los tipos `LangGraphAgent` y `A2aAgent` aún
    no están soportados.
    -   `AgentTool`
    -   `LongRunningFunctionTool`
    -   `VertexAiSearchTool`
    -   `McpToolset`
    -   `ExampleTool`

## Siguientes pasos

Para ideas sobre cómo y qué construir con ADK Agent Configs, consulta las
definiciones de agentes basadas en yaml en el
repositorio [adk-samples](https://github.com/search?q=repo:google/adk-python+path:/%5Econtributing%5C/samples%5C//+root_agent.yaml&type=code) de ADK.
Para información detallada sobre la sintaxis y configuraciones soportadas por
el formato Agent Config, consulta la
[referencia de sintaxis de Agent Config](/adk-docs/api-reference/agentconfig/).