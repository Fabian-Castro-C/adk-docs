# Herramientas del Model Context Protocol

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Esta guía te lleva a través de dos formas de integrar Model Context Protocol (MCP) con ADK.

## ¿Qué es Model Context Protocol (MCP)?

El Model Context Protocol (MCP) es un estándar abierto diseñado para estandarizar cómo los Modelos de Lenguaje Grande (LLMs) como Gemini y Claude se comunican con aplicaciones externas, fuentes de datos y herramientas. Piensa en él como un mecanismo de conexión universal que simplifica cómo los LLMs obtienen contexto, ejecutan acciones e interactúan con varios sistemas.

MCP sigue una arquitectura cliente-servidor, definiendo cómo los **datos** (recursos), **plantillas interactivas** (prompts) y **funciones accionables** (herramientas) son expuestos por un **servidor MCP** y consumidos por un **cliente MCP** (que podría ser una aplicación host de LLM o un agente de IA).

Esta guía cubre dos patrones de integración principales:

1. **Usar Servidores MCP Existentes dentro de ADK:** Un agente ADK actúa como un cliente MCP, aprovechando las herramientas proporcionadas por servidores MCP externos.
2. **Exponer Herramientas ADK a través de un Servidor MCP:** Construir un servidor MCP que envuelva herramientas ADK, haciéndolas accesibles a cualquier cliente MCP.

## Prerequisitos

Antes de comenzar, asegúrate de tener lo siguiente configurado:

* **Configurar ADK:** Sigue las [instrucciones de configuración](../get-started/index.md) estándar de ADK en el inicio rápido.
* **Instalar/actualizar Python/Java:** MCP requiere Python versión 3.9 o superior para Python o Java 17 o superior.
* **Configurar Node.js y npx:** **(Solo Python)** Muchos servidores MCP de la comunidad se distribuyen como paquetes Node.js y se ejecutan usando `npx`. Instala Node.js (que incluye npx) si aún no lo has hecho. Para más detalles, consulta [https://nodejs.org/en](https://nodejs.org/en).
* **Verificar Instalaciones:** **(Solo Python)** Confirma que `adk` y `npx` están en tu PATH dentro del entorno virtual activado:

```shell
# Ambos comandos deberían imprimir la ruta a los ejecutables.
which adk
which npx
```

## 1. Usar servidores MCP con agentes ADK (ADK como cliente MCP) en `adk web`

Esta sección demuestra cómo integrar herramientas de servidores MCP (Model Context Protocol) externos en tus agentes ADK. Este es el patrón de integración **más común** cuando tu agente ADK necesita usar capacidades proporcionadas por un servicio existente que expone una interfaz MCP. Verás cómo la clase `McpToolset` puede agregarse directamente a la lista `tools` de tu agente, permitiendo una conexión fluida a un servidor MCP, descubrimiento de sus herramientas y poniéndolas disponibles para que tu agente las use. Estos ejemplos se centran principalmente en interacciones dentro del entorno de desarrollo `adk web`.

### Clase `McpToolset`

La clase `McpToolset` es el mecanismo principal de ADK para integrar herramientas de un servidor MCP. Cuando incluyes una instancia de `McpToolset` en la lista `tools` de tu agente, automáticamente maneja la interacción con el servidor MCP especificado. Así es como funciona:

1.  **Gestión de Conexión:** Al inicializarse, `McpToolset` establece y gestiona la conexión al servidor MCP. Esta puede ser un proceso de servidor local (usando `StdioConnectionParams` para comunicación a través de entrada/salida estándar) o un servidor remoto (usando `SseConnectionParams` para Server-Sent Events). El toolset también maneja el cierre elegante de esta conexión cuando el agente o la aplicación terminan.
2.  **Descubrimiento y Adaptación de Herramientas:** Una vez conectado, `McpToolset` consulta al servidor MCP por sus herramientas disponibles (a través del método MCP `list_tools`). Luego convierte los esquemas de estas herramientas MCP descubiertas en instancias `BaseTool` compatibles con ADK.
3.  **Exposición al Agente:** Estas herramientas adaptadas se hacen disponibles a tu `LlmAgent` como si fueran herramientas ADK nativas.
4.  **Proxying de Llamadas a Herramientas:** Cuando tu `LlmAgent` decide usar una de estas herramientas, `McpToolset` transparentemente envía como proxy la llamada (usando el método MCP `call_tool`) al servidor MCP, envía los argumentos necesarios y devuelve la respuesta del servidor al agente.
5.  **Filtrado (Opcional):** Puedes usar el parámetro `tool_filter` al crear un `McpToolset` para seleccionar un subconjunto específico de herramientas del servidor MCP, en lugar de exponer todas ellas a tu agente.

Los siguientes ejemplos demuestran cómo usar `McpToolset` dentro del entorno de desarrollo `adk web`. Para escenarios donde necesitas un control más detallado sobre el ciclo de vida de la conexión MCP o no estás usando `adk web`, consulta la sección "Usar Herramientas MCP en tu propio Agente fuera de `adk web`" más adelante en esta página.

### Ejemplo 1: Servidor MCP del Sistema de Archivos

Este ejemplo en Python demuestra la conexión a un servidor MCP local que proporciona operaciones del sistema de archivos.

#### Paso 1: Define tu Agente con `McpToolset`

Crea un archivo `agent.py` (por ejemplo, en `./adk_agent_samples/mcp_agent/agent.py`). El `McpToolset` se instancia directamente dentro de la lista `tools` de tu `LlmAgent`.

*   **Importante:** Reemplaza `"/path/to/your/folder"` en la lista `args` con la **ruta absoluta** a una carpeta real en tu sistema local a la que el servidor MCP pueda acceder.
*   **Importante:** Coloca el archivo `.env` en el directorio padre del directorio `./adk_agent_samples`.

```python
# ./adk_agent_samples/mcp_agent/agent.py
import os # Requerido para operaciones de rutas
from google.adk.agents import LlmAgent
from google.adk.tools.mcp_tool import McpToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
from mcp import StdioServerParameters

# Es una buena práctica definir rutas dinámicamente si es posible,
# o asegurarse de que el usuario entienda la necesidad de una ruta ABSOLUTA.
# Para este ejemplo, construiremos una ruta relativa a este archivo,
# asumiendo que '/path/to/your/folder' está en el mismo directorio que agent.py.
# REEMPLAZA ESTO con una ruta absoluta real si es necesario para tu configuración.
TARGET_FOLDER_PATH = os.path.join(os.path.dirname(os.path.abspath(__file__)), "/path/to/your/folder")
# Asegúrate de que TARGET_FOLDER_PATH sea una ruta absoluta para el servidor MCP.
# Si creaste ./adk_agent_samples/mcp_agent/your_folder,

root_agent = LlmAgent(
    model='gemini-2.0-flash',
    name='filesystem_assistant_agent',
    instruction='Help the user manage their files. You can list files, read files, etc.',
    tools=[
        McpToolset(
            connection_params=StdioConnectionParams(
                server_params = StdioServerParameters(
                    command='npx',
                    args=[
                        "-y",  # Argumento para npx para auto-confirmar instalación
                        "@modelcontextprotocol/server-filesystem",
                        # IMPORTANTE: Esta DEBE ser una ruta ABSOLUTA a una carpeta a la que
                        # el proceso npx pueda acceder.
                        # Reemplaza con una ruta absoluta válida en tu sistema.
                        # Por ejemplo: "/Users/youruser/accessible_mcp_files"
                        # o usa una ruta absoluta construida dinámicamente:
                        os.path.abspath(TARGET_FOLDER_PATH),
                    ],
                ),
            ),
            # Opcional: Filtra qué herramientas del servidor MCP se exponen
            # tool_filter=['list_directory', 'read_file']
        )
    ],
)
```


#### Paso 2: Crea un archivo `__init__.py`

Asegúrate de tener un `__init__.py` en el mismo directorio que `agent.py` para convertirlo en un paquete Python descubrible para ADK.

```python
# ./adk_agent_samples/mcp_agent/__init__.py
from . import agent
```

#### Paso 3: Ejecuta `adk web` e Interactúa

Navega al directorio padre de `mcp_agent` (por ejemplo, `adk_agent_samples`) en tu terminal y ejecuta:

```shell
cd ./adk_agent_samples # O tu directorio padre equivalente
adk web
```

!!!info "Nota para usuarios de Windows"

    Cuando encuentres el error `_make_subprocess_transport NotImplementedError`, considera usar `adk web --no-reload` en su lugar.


Una vez que la interfaz web de ADK se cargue en tu navegador:

1.  Selecciona el `filesystem_assistant_agent` del menú desplegable de agentes.
2.  Prueba prompts como:
    *   "List files in the current directory."
    *   "Can you read the file named sample.txt?" (asumiendo que lo creaste en `TARGET_FOLDER_PATH`).
    *   "What is the content of `another_file.md`?"

Deberías ver al agente interactuando con el servidor MCP del sistema de archivos, y las respuestas del servidor (listados de archivos, contenido de archivos) transmitidas a través del agente. La consola de `adk web` (terminal donde ejecutaste el comando) también podría mostrar logs del proceso `npx` si produce salida a stderr.

<img src="../../assets/adk-tool-mcp-filesystem-adk-web-demo.png" alt="MCP with ADK Web - FileSystem Example">



Para Java, consulta el siguiente ejemplo para definir un agente que inicialice el `McpToolset`:

```java
package agents;

import com.google.adk.JsonBaseModel;
import com.google.adk.agents.LlmAgent;
import com.google.adk.agents.RunConfig;
import com.google.adk.runner.InMemoryRunner;
import com.google.adk.tools.mcp.McpTool;
import com.google.adk.tools.mcp.McpToolset;
import com.google.adk.tools.mcp.McpToolset.McpToolsAndToolsetResult;
import com.google.genai.types.Content;
import com.google.genai.types.Part;
import io.modelcontextprotocol.client.transport.ServerParameters;

import java.util.List;
import java.util.concurrent.CompletableFuture;

public class McpAgentCreator {

    /**
     * Inicializa un McpToolset, recupera herramientas de un servidor MCP usando stdio,
     * crea un LlmAgent con estas herramientas, envía un prompt al agente,
     * y asegura que el toolset se cierre.
     * @param args Argumentos de línea de comandos (no utilizados).
     */
    public static void main(String[] args) {
        //Nota: puedes tener problemas de permisos si la carpeta está fuera de home
        String yourFolderPath = "~/path/to/folder";

        ServerParameters connectionParams = ServerParameters.builder("npx")
                .args(List.of(
                        "-y",
                        "@modelcontextprotocol/server-filesystem",
                        yourFolderPath
                ))
                .build();

        try {
            CompletableFuture<McpToolsAndToolsetResult> futureResult =
                    McpToolset.fromServer(connectionParams, JsonBaseModel.getMapper());

            McpToolsAndToolsetResult result = futureResult.join();

            try (McpToolset toolset = result.getToolset()) {
                List<McpTool> tools = result.getTools();

                LlmAgent agent = LlmAgent.builder()
                        .model("gemini-2.0-flash")
                        .name("enterprise_assistant")
                        .description("An agent to help users access their file systems")
                        .instruction(
                                "Help user accessing their file systems. You can list files in a directory."
                        )
                        .tools(tools)
                        .build();

                System.out.println("Agent created: " + agent.name());

                InMemoryRunner runner = new InMemoryRunner(agent);
                String userId = "user123";
                String sessionId = "1234";
                String promptText = "Which files are in this directory - " + yourFolderPath + "?";

                // Crear explícitamente la sesión primero
                try {
                    // appName para InMemoryRunner por defecto es agent.name() si no se especifica en el constructor
                    runner.sessionService().createSession(runner.appName(), userId, null, sessionId).blockingGet();
                    System.out.println("Session created: " + sessionId + " for user: " + userId);
                } catch (Exception sessionCreationException) {
                    System.err.println("Failed to create session: " + sessionCreationException.getMessage());
                    sessionCreationException.printStackTrace();
                    return;
                }

                Content promptContent = Content.fromParts(Part.fromText(promptText));

                System.out.println("\nSending prompt: \"" + promptText + "\" to agent...\n");

                runner.runAsync(userId, sessionId, promptContent, RunConfig.builder().build())
                        .blockingForEach(event -> {
                            System.out.println("Event received: " + event.toJson());
                        });
            }
        } catch (Exception e) {
            System.err.println("An error occurred: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

Asumiendo una carpeta que contiene tres archivos llamados `first`, `second` y `third`, una respuesta exitosa se verá así:

```shell
Event received: {"id":"163a449e-691a-48a2-9e38-8cadb6d1f136","invocationId":"e-c2458c56-e57a-45b2-97de-ae7292e505ef","author":"enterprise_assistant","content":{"parts":[{"functionCall":{"id":"adk-388b4ac2-d40e-4f6a-bda6-f051110c6498","args":{"path":"~/home-test"},"name":"list_directory"}}],"role":"model"},"actions":{"stateDelta":{},"artifactDelta":{},"requestedAuthConfigs":{}},"timestamp":1747377543788}

Event received: {"id":"8728380b-bfad-4d14-8421-fa98d09364f1","invocationId":"e-c2458c56-e57a-45b2-97de-ae7292e505ef","author":"enterprise_assistant","content":{"parts":[{"functionResponse":{"id":"adk-388b4ac2-d40e-4f6a-bda6-f051110c6498","name":"list_directory","response":{"text_output":[{"text":"[FILE] first\n[FILE] second\n[FILE] third"}]}}}],"role":"user"},"actions":{"stateDelta":{},"artifactDelta":{},"requestedAuthConfigs":{}},"timestamp":1747377544679}

Event received: {"id":"8fe7e594-3e47-4254-8b57-9106ad8463cb","invocationId":"e-c2458c56-e57a-45b2-97de-ae7292e505ef","author":"enterprise_assistant","content":{"parts":[{"text":"There are three files in the directory: first, second, and third."}],"role":"model"},"actions":{"stateDelta":{},"artifactDelta":{},"requestedAuthConfigs":{}},"timestamp":1747377544689}
```

Para Typescript, puedes definir un agente que inicialice el `MCPToolset` como sigue:

```typescript
import 'dotenv/config';
import {LlmAgent, MCPToolset} from "@google/adk";

// REEMPLAZA ESTO con una ruta absoluta real para tu configuración.
const TARGET_FOLDER_PATH = "/path/to/your/folder";

export const rootAgent = new LlmAgent({
    model: "gemini-2.5-flash",
    name: "filesystem_assistant_agent",
    instruction: "Help the user manage their files. You can list files, read files, etc.",
    tools: [
        // Para filtrar herramientas, pasa una lista de nombres de herramientas como segundo argumento
        // al constructor MCPToolset.
        // ej., new MCPToolset(connectionParams, ['list_directory', 'read_file'])
        new MCPToolset(
            {
                type: "StdioConnectionParams",
                serverParams: {
                    command: "npx",
                    args: [
                        "-y",
                        "@modelcontextprotocol/server-filesystem",
                        // IMPORTANTE: Esta DEBE ser una ruta ABSOLUTA a una carpeta a la que
                        // el proceso npx pueda acceder.
                        // Reemplaza con una ruta absoluta válida en tu sistema.
                        // Por ejemplo: "/Users/youruser/accessible_mcp_files"
                        TARGET_FOLDER_PATH,
                    ],
                },
            }
        )
    ],
});
```



### Ejemplo 2: Servidor MCP de Google Maps

Este ejemplo demuestra la conexión al servidor MCP de Google Maps.

#### Paso 1: Obtén la Clave API y Habilita las APIs

1.  **Clave API de Google Maps:** Sigue las instrucciones en [Use API keys](https://developers.google.com/maps/documentation/javascript/get-api-key#create-api-keys) para obtener una Clave API de Google Maps.
2.  **Habilitar APIs:** En tu proyecto de Google Cloud, asegúrate de que las siguientes APIs estén habilitadas:
    *   Directions API
    *   Routes API
    Para instrucciones, consulta la documentación [Getting started with Google Maps Platform](https://developers.google.com/maps/get-started#enable-api-sdk).

#### Paso 2: Define tu Agente con `McpToolset` para Google Maps

Modifica tu archivo `agent.py` (por ejemplo, en `./adk_agent_samples/mcp_agent/agent.py`). Reemplaza `YOUR_GOOGLE_MAPS_API_KEY` con la clave API real que obtuviste.

```python
# ./adk_agent_samples/mcp_agent/agent.py
import os
from google.adk.agents import LlmAgent
from google.adk.tools.mcp_tool import McpToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
from mcp import StdioServerParameters

# Recupera la clave API de una variable de entorno o insértala directamente.
# Usar una variable de entorno es generalmente más seguro.
# Asegúrate de que esta variable de entorno esté configurada en el terminal donde ejecutas 'adk web'.
# Ejemplo: export GOOGLE_MAPS_API_KEY="YOUR_ACTUAL_KEY"
google_maps_api_key = os.environ.get("GOOGLE_MAPS_API_KEY")

if not google_maps_api_key:
    # Respaldo o asignación directa para pruebas - NO RECOMENDADO PARA PRODUCCIÓN
    google_maps_api_key = "YOUR_GOOGLE_MAPS_API_KEY_HERE" # Reemplaza si no usas variable de entorno
    if google_maps_api_key == "YOUR_GOOGLE_MAPS_API_KEY_HERE":
        print("WARNING: GOOGLE_MAPS_API_KEY is not set. Please set it as an environment variable or in the script.")
        # Podrías querer lanzar un error o salir si la clave es crucial y no se encuentra.

root_agent = LlmAgent(
    model='gemini-2.0-flash',
    name='maps_assistant_agent',
    instruction='Help the user with mapping, directions, and finding places using Google Maps tools.',
    tools=[
        McpToolset(
            connection_params=StdioConnectionParams(
                server_params = StdioServerParameters(
                    command='npx',
                    args=[
                        "-y",
                        "@modelcontextprotocol/server-google-maps",
                    ],
                    # Pasa la clave API como una variable de entorno al proceso npx
                    # Así es como el servidor MCP para Google Maps espera la clave.
                    env={
                        "GOOGLE_MAPS_API_KEY": google_maps_api_key
                    }
                ),
            ),
            # Puedes filtrar herramientas específicas de Maps si es necesario:
            # tool_filter=['get_directions', 'find_place_by_id']
        )
    ],
)
```

#### Paso 3: Asegúrate de que Existe `__init__.py`

Si lo creaste en el Ejemplo 1, puedes omitir esto. De lo contrario, asegúrate de tener un `__init__.py` en el directorio `./adk_agent_samples/mcp_agent/`:

```python
# ./adk_agent_samples/mcp_agent/__init__.py
from . import agent
```

#### Paso 4: Ejecuta `adk web` e Interactúa

1.  **Configurar Variable de Entorno (Recomendado):**
    Antes de ejecutar `adk web`, es mejor configurar tu clave API de Google Maps como una variable de entorno en tu terminal:
    ```shell
    export GOOGLE_MAPS_API_KEY="YOUR_ACTUAL_GOOGLE_MAPS_API_KEY"
    ```
    Reemplaza `YOUR_ACTUAL_GOOGLE_MAPS_API_KEY` con tu clave.

2.  **Ejecutar `adk web`**:
    Navega al directorio padre de `mcp_agent` (por ejemplo, `adk_agent_samples`) y ejecuta:
    ```shell
    cd ./adk_agent_samples # O tu directorio padre equivalente
    adk web
    ```

3.  **Interactuar en la Interfaz**:
    *   Selecciona el `maps_assistant_agent`.
    *   Prueba prompts como:
        *   "Get directions from GooglePlex to SFO."
        *   "Find coffee shops near Golden Gate Park."
        *   "What's the route from Paris, France to Berlin, Germany?"

Deberías ver al agente usar las herramientas MCP de Google Maps para proporcionar direcciones o información basada en ubicación.

<img src="../../assets/adk-tool-mcp-maps-adk-web-demo.png" alt="MCP with ADK Web - Google Maps Example">


Para Java, consulta el siguiente ejemplo para definir un agente que inicialice el `McpToolset`:

```java
package agents;

import com.google.adk.JsonBaseModel;
import com.google.adk.agents.LlmAgent;
import com.google.adk.agents.RunConfig;
import com.google.adk.runner.InMemoryRunner;
import com.google.adk.tools.mcp.McpTool;
import com.google.adk.tools.mcp.McpToolset;
import com.google.adk.tools.mcp.McpToolset.McpToolsAndToolsetResult;


import com.google.genai.types.Content;
import com.google.genai.types.Part;

import io.modelcontextprotocol.client.transport.ServerParameters;

import java.util.List;
import java.util.Map;
import java.util.Collections;
import java.util.HashMap;
import java.util.concurrent.CompletableFuture;
import java.util.Arrays;

public class MapsAgentCreator {

    /**
     * Inicializa un McpToolset para Google Maps, recupera herramientas,
     * crea un LlmAgent, envía un prompt relacionado con mapas y cierra el toolset.
     * @param args Argumentos de línea de comandos (no utilizados).
     */
    public static void main(String[] args) {
        // TODO: Reemplaza con tu clave API real de Google Maps, en un proyecto con la Places API habilitada.
        String googleMapsApiKey = "YOUR_GOOGLE_MAPS_API_KEY";

        Map<String, String> envVariables = new HashMap<>();
        envVariables.put("GOOGLE_MAPS_API_KEY", googleMapsApiKey);

        ServerParameters connectionParams = ServerParameters.builder("npx")
                .args(List.of(
                        "-y",
                        "@modelcontextprotocol/server-google-maps"
                ))
                .env(Collections.unmodifiableMap(envVariables))
                .build();

        try {
            CompletableFuture<McpToolsAndToolsetResult> futureResult =
                    McpToolset.fromServer(connectionParams, JsonBaseModel.getMapper());

            McpToolsAndToolsetResult result = futureResult.join();

            try (McpToolset toolset = result.getToolset()) {
                List<McpTool> tools = result.getTools();

                LlmAgent agent = LlmAgent.builder()
                        .model("gemini-2.0-flash")
                        .name("maps_assistant")
                        .description("Maps assistant")
                        .instruction("Help user with mapping and directions using available tools.")
                        .tools(tools)
                        .build();

                System.out.println("Agent created: " + agent.name());

                InMemoryRunner runner = new InMemoryRunner(agent);
                String userId = "maps-user-" + System.currentTimeMillis();
                String sessionId = "maps-session-" + System.currentTimeMillis();

                String promptText = "Please give me directions to the nearest pharmacy to Madison Square Garden.";

                try {
                    runner.sessionService().createSession(runner.appName(), userId, null, sessionId).blockingGet();
                    System.out.println("Session created: " + sessionId + " for user: " + userId);
                } catch (Exception sessionCreationException) {
                    System.err.println("Failed to create session: " + sessionCreationException.getMessage());
                    sessionCreationException.printStackTrace();
                    return;
                }

                Content promptContent = Content.fromParts(Part.fromText(promptText))

                System.out.println("\nSending prompt: \"" + promptText + "\" to agent...\n");

                runner.runAsync(userId, sessionId, promptContent, RunConfig.builder().build())
                        .blockingForEach(event -> {
                            System.out.println("Event received: " + event.toJson());
                        });
            }
        } catch (Exception e) {
            System.err.println("An error occurred: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

Una respuesta exitosa se verá así:
```shell
Event received: {"id":"1a4deb46-c496-4158-bd41-72702c773368","invocationId":"e-48994aa0-531c-47be-8c57-65215c3e0319","author":"maps_assistant","content":{"parts":[{"text":"OK. I see a few options. The closest one is CVS Pharmacy at 5 Pennsylvania Plaza, New York, NY 10001, United States. Would you like directions?\n"}],"role":"model"},"actions":{"stateDelta":{},"artifactDelta":{},"requestedAuthConfigs":{}},"timestamp":1747380026642}
```

Para TypeScript, consulta el siguiente ejemplo para definir un agente que inicialice el `MCPToolset`:

```typescript
import 'dotenv/config';
import {LlmAgent, MCPToolset} from "@google/adk";

// Recupera la clave API de una variable de entorno.
// Asegúrate de que esta variable de entorno esté configurada en el terminal donde ejecutas 'adk web'.
// Ejemplo: export GOOGLE_MAPS_API_KEY="YOUR_ACTUAL_KEY"
const googleMapsApiKey = process.env.GOOGLE_MAPS_API_KEY;
if (!googleMapsApiKey) {
    throw new Error('GOOGLE_MAPS_API_KEY is not provided, please run "export GOOGLE_MAPS_API_KEY=YOUR_ACTUAL_KEY" to add that.');
}

export const rootAgent = new LlmAgent({
    model: "gemini-2.5-flash",
    name: "maps_assistant_agent",
    instruction: "Help the user with mapping, directions, and finding places using Google Maps tools.",
    tools: [
        new MCPToolset(
            {
                type: "StdioConnectionParams",
                serverParams: {
                    command: "npx",
                    args: [
                        "-y",
                        "@modelcontextprotocol/server-google-maps",
                    ],
                    // Pasa la clave API como una variable de entorno al proceso npx
                    // Así es como el servidor MCP para Google Maps espera la clave.
                    env: {
                        "GOOGLE_MAPS_API_KEY": googleMapsApiKey
                    }
                },
            },
            // Puedes filtrar herramientas específicas de Maps si es necesario:
            // ['get_directions', 'find_place_by_id']
        )
    ],
});
```

Una respuesta exitosa se verá así:
```shell
Event received: {"id":"1a4deb46-c496-4158-bd41-72702c773368","invocationId":"e-48994aa0-531c-47be-8c57-65215c3e0319","author":"maps_assistant","content":{"parts":[{"text":"OK. I see a few options. The closest one is CVS Pharmacy at 5 Pennsylvania Plaza, New York, NY 10001, United States. Would you like directions?\n"}],"role":"model"},"actions":{"stateDelta":{},"artifactDelta":{},"requestedAuthConfigs":{}},"timestamp":1747380026642}
```

## 2. Construir un servidor MCP con herramientas ADK (servidor MCP exponiendo ADK)

Este patrón te permite envolver herramientas ADK existentes y hacerlas disponibles a cualquier aplicación cliente MCP estándar. El ejemplo en esta sección expone la herramienta ADK `load_web_page` a través de un servidor MCP construido a medida.

### Resumen de pasos

Crearás una aplicación de servidor MCP estándar en Python usando la biblioteca `mcp`. Dentro de este servidor, deberás:

1.  Instanciar la(s) herramienta(s) ADK que quieres exponer (por ejemplo, `FunctionTool(load_web_page)`).
2.  Implementar el manejador `@app.list_tools()` del servidor MCP para anunciar la(s) herramienta(s) ADK. Esto involucra convertir la definición de la herramienta ADK al esquema MCP usando la utilidad `adk_to_mcp_tool_type` de `google.adk.tools.mcp_tool.conversion_utils`.
3.  Implementar el manejador `@app.call_tool()` del servidor MCP. Este manejador deberá:
    *   Recibir solicitudes de llamadas a herramientas de clientes MCP.
    *   Identificar si la solicitud apunta a una de tus herramientas ADK envueltas.
    *   Ejecutar el método `.run_async()` de la herramienta ADK.
    *   Formatear el resultado de la herramienta ADK en una respuesta compatible con MCP (por ejemplo, `mcp.types.TextContent`).

### Prerequisitos

Instala la biblioteca del servidor MCP en el mismo entorno Python que tu instalación de ADK:

```shell
pip install mcp
```

### Paso 1: Crea el Script del Servidor MCP

Crea un nuevo archivo Python para tu servidor MCP, por ejemplo, `my_adk_mcp_server.py`.

### Paso 2: Implementa la Lógica del Servidor

Agrega el siguiente código a `my_adk_mcp_server.py`. Este script configura un servidor MCP que expone la herramienta ADK `load_web_page`.

```python
# my_adk_mcp_server.py
import asyncio
import json
import os
from dotenv import load_dotenv

# Importaciones del Servidor MCP
from mcp import types as mcp_types # Usa alias para evitar conflictos
from mcp.server.lowlevel import Server, NotificationOptions
from mcp.server.models import InitializationOptions
import mcp.server.stdio # Para ejecutar como servidor stdio

# Importaciones de Herramientas ADK
from google.adk.tools.function_tool import FunctionTool
from google.adk.tools.load_web_page import load_web_page # Herramienta ADK de ejemplo
# Utilidad de Conversión ADK <-> MCP
from google.adk.tools.mcp_tool.conversion_utils import adk_to_mcp_tool_type

# --- Cargar Variables de Entorno (Si las herramientas ADK las necesitan, ej., claves API) ---
load_dotenv() # Crea un archivo .env en el mismo directorio si es necesario

# --- Preparar la Herramienta ADK ---
# Instancia la herramienta ADK que quieres exponer.
# Esta herramienta será envuelta y llamada por el servidor MCP.
print("Initializing ADK load_web_page tool...")
adk_tool_to_expose = FunctionTool(load_web_page)
print(f"ADK tool '{adk_tool_to_expose.name}' initialized and ready to be exposed via MCP.")
# --- Fin Preparación Herramienta ADK ---

# --- Configuración del Servidor MCP ---
print("Creating MCP Server instance...")
# Crea una instancia de Servidor MCP nombrada usando la biblioteca mcp.server
app = Server("adk-tool-exposing-mcp-server")

# Implementa el manejador del servidor MCP para listar herramientas disponibles
@app.list_tools()
async def list_mcp_tools() -> list[mcp_types.Tool]:
    """Manejador MCP para listar herramientas que este servidor expone."""
    print("MCP Server: Received list_tools request.")
    # Convierte la definición de la herramienta ADK al formato de esquema de Herramienta MCP
    mcp_tool_schema = adk_to_mcp_tool_type(adk_tool_to_expose)
    print(f"MCP Server: Advertising tool: {mcp_tool_schema.name}")
    return [mcp_tool_schema]

# Implementa el manejador del servidor MCP para ejecutar una llamada a herramienta
@app.call_tool()
async def call_mcp_tool(
    name: str, arguments: dict
) -> list[mcp_types.Content]: # MCP usa mcp_types.Content
    """Manejador MCP para ejecutar una llamada a herramienta solicitada por un cliente MCP."""
    print(f"MCP Server: Received call_tool request for '{name}' with args: {arguments}")

    # Verifica si el nombre de la herramienta solicitada coincide con nuestra herramienta ADK envuelta
    if name == adk_tool_to_expose.name:
        try:
            # Ejecuta el método run_async de la herramienta ADK.
            # Nota: tool_context es None aquí porque este servidor MCP está
            # ejecutando la herramienta ADK fuera de una invocación completa de ADK Runner.
            # Si la herramienta ADK requiere características de ToolContext (como estado o autenticación),
            # esta invocación directa podría necesitar un manejo más sofisticado.
            adk_tool_response = await adk_tool_to_expose.run_async(
                args=arguments,
                tool_context=None,
            )
            print(f"MCP Server: ADK tool '{name}' executed. Response: {adk_tool_response}")

            # Formatea la respuesta de la herramienta ADK (a menudo un dict) en un formato compatible con MCP.
            # Aquí, serializamos el diccionario de respuesta como una cadena JSON dentro de TextContent.
            # Ajusta el formato según la salida de la herramienta ADK y las necesidades del cliente.
            response_text = json.dumps(adk_tool_response, indent=2)
            # MCP espera una lista de partes mcp_types.Content
            return [mcp_types.TextContent(type="text", text=response_text)]

        except Exception as e:
            print(f"MCP Server: Error executing ADK tool '{name}': {e}")
            # Devuelve un mensaje de error en formato MCP
            error_text = json.dumps({"error": f"Failed to execute tool '{name}': {str(e)}"})
            return [mcp_types.TextContent(type="text", text=error_text)]
    else:
        # Maneja llamadas a herramientas desconocidas
        print(f"MCP Server: Tool '{name}' not found/exposed by this server.")
        error_text = json.dumps({"error": f"Tool '{name}' not implemented by this server."})
        return [mcp_types.TextContent(type="text", text=error_text)]

# --- Ejecutor del Servidor MCP ---
async def run_mcp_stdio_server():
    """Ejecuta el servidor MCP, escuchando conexiones a través de entrada/salida estándar."""
    # Usa el gestor de contexto stdio_server de la biblioteca mcp.server.stdio
    async with mcp.server.stdio.stdio_server() as (read_stream, write_stream):
        print("MCP Stdio Server: Starting handshake with client...")
        await app.run(
            read_stream,
            write_stream,
            InitializationOptions(
                server_name=app.name, # Usa el nombre del servidor definido arriba
                server_version="0.1.0",
                capabilities=app.get_capabilities(
                    # Define capacidades del servidor - consulta documentación MCP para opciones
                    notification_options=NotificationOptions(),
                    experimental_capabilities={},
                ),
            ),
        )
        print("MCP Stdio Server: Run loop finished or client disconnected.")

if __name__ == "__main__":
    print("Launching MCP Server to expose ADK tools via stdio...")
    try:
        asyncio.run(run_mcp_stdio_server())
    except KeyboardInterrupt:
        print("\nMCP Server (stdio) stopped by user.")
    except Exception as e:
        print(f"MCP Server (stdio) encountered an error: {e}")
    finally:
        print("MCP Server (stdio) process exiting.")
# --- Fin Servidor MCP ---
```

### Paso 3: Prueba tu Servidor MCP Personalizado con un Agente ADK

Ahora, crea un agente ADK que actuará como cliente del servidor MCP que acabas de construir. Este agente ADK usará `McpToolset` para conectarse a tu script `my_adk_mcp_server.py`.

Crea un `agent.py` (por ejemplo, en `./adk_agent_samples/mcp_client_agent/agent.py`):

```python
# ./adk_agent_samples/mcp_client_agent/agent.py
import os
from google.adk.agents import LlmAgent
from google.adk.tools.mcp_tool import McpToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
from mcp import StdioServerParameters

# IMPORTANTE: Reemplaza esto con la ruta ABSOLUTA a tu script my_adk_mcp_server.py
PATH_TO_YOUR_MCP_SERVER_SCRIPT = "/path/to/your/my_adk_mcp_server.py" # <<< REEMPLAZAR

if PATH_TO_YOUR_MCP_SERVER_SCRIPT == "/path/to/your/my_adk_mcp_server.py":
    print("WARNING: PATH_TO_YOUR_MCP_SERVER_SCRIPT is not set. Please update it in agent.py.")
    # Opcionalmente, lanza un error si la ruta es crítica

root_agent = LlmAgent(
    model='gemini-2.0-flash',
    name='web_reader_mcp_client_agent',
    instruction="Use the 'load_web_page' tool to fetch content from a URL provided by the user.",
    tools=[
        McpToolset(
            connection_params=StdioConnectionParams(
                server_params = StdioServerParameters(
                    command='python3', # Comando para ejecutar tu script del servidor MCP
                    args=[PATH_TO_YOUR_MCP_SERVER_SCRIPT], # El argumento es la ruta al script
                )
            )
            # tool_filter=['load_web_page'] # Opcional: asegura que solo se carguen herramientas específicas
        )
    ],
)
```

Y un `__init__.py` en el mismo directorio:
```python
# ./adk_agent_samples/mcp_client_agent/__init__.py
from . import agent
```

**Para ejecutar la prueba:**

1.  **Inicia tu servidor MCP personalizado (opcional, para observación separada):**
    Puedes ejecutar tu `my_adk_mcp_server.py` directamente en una terminal para ver sus logs:
    ```shell
    python3 /path/to/your/my_adk_mcp_server.py
    ```
    Imprimirá "Launching MCP Server..." y esperará. El agente ADK (ejecutado vía `adk web`) se conectará a este proceso si el `command` en `StdioConnectionParams` está configurado para ejecutarlo.
    *(Alternativamente, `McpToolset` iniciará este script de servidor como un subproceso automáticamente cuando el agente se inicialice).*

2.  **Ejecutar `adk web` para el agente cliente:**
    Navega al directorio padre de `mcp_client_agent` (por ejemplo, `adk_agent_samples`) y ejecuta:
    ```shell
    cd ./adk_agent_samples # O tu directorio padre equivalente
    adk web
    ```

3.  **Interactuar en la Interfaz Web de ADK:**
    *   Selecciona el `web_reader_mcp_client_agent`.
    *   Prueba un prompt como: "Load the content from https://example.com"

El agente ADK (`web_reader_mcp_client_agent`) usará `McpToolset` para iniciar y conectarse a tu `my_adk_mcp_server.py`. Tu servidor MCP recibirá la solicitud `call_tool`, ejecutará la herramienta ADK `load_web_page` y devolverá el resultado. El agente ADK luego transmitirá esta información. Deberías ver logs tanto de la interfaz web de ADK (y su terminal) como potencialmente de tu terminal `my_adk_mcp_server.py` si lo ejecutaste por separado.

Este ejemplo demuestra cómo las herramientas ADK pueden encapsularse dentro de un servidor MCP, haciéndolas accesibles a una gama más amplia de clientes compatibles con MCP, no solo agentes ADK.

Consulta la [documentación](https://modelcontextprotocol.io/quickstart/server#core-mcp-concepts), para probarlo con Claude Desktop.

## Usar Herramientas MCP en tu propio Agente fuera de `adk web`

Esta sección es relevante para ti si:

* Estás desarrollando tu propio Agente usando ADK
* Y **NO** estás usando `adk web`,
* Y estás exponiendo el agente a través de tu propia interfaz de usuario


Usar Herramientas MCP requiere una configuración diferente a usar herramientas regulares, debido al hecho de que las especificaciones para las Herramientas MCP se obtienen de forma asíncrona
del Servidor MCP ejecutándose de forma remota o en otro proceso.

El siguiente ejemplo es modificado del ejemplo "Ejemplo 1: Servidor MCP del Sistema de Archivos" anterior. Las principales diferencias son:

1. Tu herramienta y agente se crean de forma asíncrona
2. Necesitas gestionar adecuadamente la pila de salida, para que tus agentes y herramientas se destruyan adecuadamente cuando la conexión al Servidor MCP se cierre.

```python
# agent.py (modifica get_tools_async y otras partes según sea necesario)
# ./adk_agent_samples/mcp_agent/agent.py
import os
import asyncio
from dotenv import load_dotenv
from google.genai import types
from google.adk.agents.llm_agent import LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.artifacts.in_memory_artifact_service import InMemoryArtifactService # Opcional
from google.adk.tools.mcp_tool import McpToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
from mcp import StdioServerParameters

# Cargar variables de entorno del archivo .env en el directorio padre
# Coloca esto cerca del inicio, antes de usar variables de entorno como claves API
load_dotenv('../.env')

# Asegúrate de que TARGET_FOLDER_PATH sea una ruta absoluta para el servidor MCP.
TARGET_FOLDER_PATH = os.path.join(os.path.dirname(os.path.abspath(__file__)), "/path/to/your/folder")

# --- Paso 1: Definición del Agente ---
async def get_agent_async():
  """Crea un Agente ADK equipado con herramientas del Servidor MCP."""
  toolset = McpToolset(
      # Usa StdioConnectionParams para comunicación de procesos locales
      connection_params=StdioConnectionParams(
          server_params = StdioServerParameters(
            command='npx', # Comando para ejecutar el servidor
            args=["-y",    # Argumentos para el comando
                "@modelcontextprotocol/server-filesystem",
                TARGET_FOLDER_PATH],
          ),
      ),
      tool_filter=['read_file', 'list_directory'] # Opcional: filtra herramientas específicas
      # Para servidores remotos, usarías SseConnectionParams en su lugar:
      # connection_params=SseConnectionParams(url="http://remote-server:port/path", headers={...})
  )

  # Usa en un agente
  root_agent = LlmAgent(
      model='gemini-2.0-flash', # Ajusta el nombre del modelo si es necesario según disponibilidad
      name='enterprise_assistant',
      instruction='Help user accessing their file systems',
      tools=[toolset], # Proporciona las herramientas MCP al agente ADK
  )
  return root_agent, toolset

# --- Paso 2: Lógica de Ejecución Principal ---
async def async_main():
  session_service = InMemorySessionService()
  # El servicio de artefactos podría no ser necesario para este ejemplo
  artifacts_service = InMemoryArtifactService()

  session = await session_service.create_session(
      state={}, app_name='mcp_filesystem_app', user_id='user_fs'
  )

  # TODO: Cambia la consulta para que sea relevante a TU carpeta especificada.
  # ej., "list files in the 'documents' subfolder" o "read the file 'notes.txt'"
  query = "list files in the tests folder"
  print(f"User Query: '{query}'")
  content = types.Content(role='user', parts=[types.Part(text=query)])

  root_agent, toolset = await get_agent_async()

  runner = Runner(
      app_name='mcp_filesystem_app',
      agent=root_agent,
      artifact_service=artifacts_service, # Opcional
      session_service=session_service,
  )

  print("Running agent...")
  events_async = runner.run_async(
      session_id=session.id, user_id=session.user_id, new_message=content
  )

  async for event in events_async:
    print(f"Event received: {event}")

  # La limpieza es manejada automáticamente por el framework del agente
  # Pero también puedes cerrar manualmente si es necesario:
  print("Closing MCP server connection...")
  await toolset.close()
  print("Cleanup complete.")

if __name__ == '__main__':
  try:
    asyncio.run(async_main())
  except Exception as e:
    print(f"An error occurred: {e}")
```


## Consideraciones clave

Al trabajar con MCP y ADK, ten en cuenta estos puntos:

* **Protocolo vs. Biblioteca:** MCP es una especificación de protocolo, que define reglas de comunicación. ADK es una biblioteca/framework de Python para construir agentes. McpToolset une estos implementando el lado cliente del protocolo MCP dentro del framework ADK. Por el contrario, construir un servidor MCP en Python requiere usar la biblioteca model-context-protocol.

* **Herramientas ADK vs. Herramientas MCP:**

    * Las Herramientas ADK (BaseTool, FunctionTool, AgentTool, etc.) son objetos Python diseñados para uso directo dentro del LlmAgent y Runner de ADK.
    * Las Herramientas MCP son capacidades expuestas por un Servidor MCP según el esquema del protocolo. McpToolset hace que estas parezcan herramientas ADK para un LlmAgent.

* **Naturaleza asíncrona:** Tanto ADK como la biblioteca MCP de Python están fuertemente basadas en la biblioteca asyncio de Python. Las implementaciones de herramientas y manejadores de servidor generalmente deberían ser funciones async.

* **Sesiones con estado (MCP):** MCP establece conexiones persistentes con estado entre una instancia de cliente y servidor. Esto difiere de las APIs REST típicas sin estado.

    * **Despliegue:** Esta naturaleza con estado puede plantear desafíos para el escalado y despliegue, especialmente para servidores remotos que manejan muchos usuarios. El diseño MCP original a menudo asumía que el cliente y el servidor estaban colocados juntos. Gestionar estas conexiones persistentes requiere consideraciones cuidadosas de infraestructura (por ejemplo, balanceo de carga, afinidad de sesión).
    * **ADK McpToolset:** Gestiona este ciclo de vida de conexión. El patrón exit\_stack mostrado en los ejemplos es crucial para asegurar que la conexión (y potencialmente el proceso del servidor) se termine adecuadamente cuando el agente ADK termine.

## Desplegar Agentes con Herramientas MCP

Al desplegar agentes ADK que usan herramientas MCP en entornos de producción como Cloud Run, GKE o Vertex AI Agent Engine, necesitas considerar cómo funcionarán las conexiones MCP en entornos en contenedores y distribuidos.

### Requisito Crítico de Despliegue: Definición Síncrona del Agente

**⚠️ Importante:** Al desplegar agentes con herramientas MCP, el agente y su McpToolset deben definirse **síncronamente** en tu archivo `agent.py`. Mientras que `adk web` permite la creación asíncrona de agentes, los entornos de despliegue requieren instanciación síncrona.

```python
# ✅ CORRECTO: Definición síncrona del agente para despliegue
import os
from google.adk.agents.llm_agent import LlmAgent
from google.adk.tools.mcp_tool import McpToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
from mcp import StdioServerParameters

_allowed_path = os.path.dirname(os.path.abspath(__file__))

root_agent = LlmAgent(
    model='gemini-2.0-flash',
    name='enterprise_assistant',
    instruction=f'Help user accessing their file systems. Allowed directory: {_allowed_path}',
    tools=[
        McpToolset(
            connection_params=StdioConnectionParams(
                server_params=StdioServerParameters(
                    command='npx',
                    args=['-y', '@modelcontextprotocol/server-filesystem', _allowed_path],
                ),
                timeout=5,  # Configura timeouts apropiados
            ),
            # Filtra herramientas por seguridad en producción
            tool_filter=[
                'read_file', 'read_multiple_files', 'list_directory',
                'directory_tree', 'search_files', 'get_file_info',
                'list_allowed_directories',
            ],
        )
    ],
)
```

```python
# ❌ INCORRECTO: Los patrones asíncronos no funcionan en despliegue
async def get_agent():  # Esto no funcionará para despliegue
    toolset = await create_mcp_toolset_async()
    return LlmAgent(tools=[toolset])
```

### Comandos Rápidos de Despliegue

#### Vertex AI Agent Engine
```bash
uv run adk deploy agent_engine \
  --project=<your-gcp-project-id> \
  --region=<your-gcp-region> \
  --staging_bucket="gs://<your-gcs-bucket>" \
  --display_name="My MCP Agent" \
  ./path/to/your/agent_directory
```

#### Cloud Run
```bash
uv run adk deploy cloud_run \
  --project=<your-gcp-project-id> \
  --region=<your-gcp-region> \
  --service_name=<your-service-name> \
  ./path/to/your/agent_directory
```

### Patrones de Despliegue

#### Patrón 1: Servidores MCP Stdio Autocontenidos

Para servidores MCP que pueden empaquetarse como paquetes npm o módulos Python (como `@modelcontextprotocol/server-filesystem`), puedes incluirlos directamente en el contenedor de tu agente:

**Requisitos del Contenedor:**
```dockerfile
# Ejemplo para servidores MCP basados en npm
FROM python:3.13-slim

# Instala Node.js y npm para servidores MCP
RUN apt-get update && apt-get install -y nodejs npm && rm -rf /var/lib/apt/lists/*

# Instala tus dependencias Python
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copia el código de tu agente
COPY . .

# Tu agente ahora puede usar StdioConnectionParams con comandos 'npx'
CMD ["python", "main.py"]
```

**Configuración del Agente:**
```python
# Esto funciona en contenedores porque npx y el servidor MCP se ejecutan en el mismo entorno
McpToolset(
    connection_params=StdioConnectionParams(
        server_params=StdioServerParameters(
            command='npx',
            args=["-y", "@modelcontextprotocol/server-filesystem", "/app/data"],
        ),
    ),
)
```

#### Patrón 2: Servidores MCP Remotos (HTTP Streamable)

Para despliegues de producción que requieren escalabilidad, despliega servidores MCP como servicios separados y conéctate a través de HTTP Streamable:

**Despliegue del Servidor MCP (Cloud Run):**
```python
# deploy_mcp_server.py - Servicio Cloud Run separado usando HTTP Streamable
import contextlib
import logging
from collections.abc import AsyncIterator
from typing import Any

import anyio
import click
import mcp.types as types
from mcp.server.lowlevel import Server
from mcp.server.streamable_http_manager import StreamableHTTPSessionManager
from starlette.applications import Starlette
from starlette.routing import Mount
from starlette.types import Receive, Scope, Send

logger = logging.getLogger(__name__)

def create_mcp_server():
    """Crea y configura el servidor MCP."""
    app = Server("adk-mcp-streamable-server")

    @app.call_tool()
    async def call_tool(name: str, arguments: dict[str, Any]) -> list[types.ContentBlock]:
        """Maneja llamadas a herramientas de clientes MCP."""
        # Implementación de herramienta de ejemplo - reemplaza con tus herramientas ADK reales
        if name == "example_tool":
            result = arguments.get("input", "No input provided")
            return [
                types.TextContent(
                    type="text",
                    text=f"Processed: {result}"
                )
            ]
        else:
            raise ValueError(f"Unknown tool: {name}")

    @app.list_tools()
    async def list_tools() -> list[types.Tool]:
        """Lista herramientas disponibles."""
        return [
            types.Tool(
                name="example_tool",
                description="Example tool for demonstration",
                inputSchema={
                    "type": "object",
                    "properties": {
                        "input": {
                            "type": "string",
                            "description": "Input text to process"
                        }
                    },
                    "required": ["input"]
                }
            )
        ]

    return app

def main(port: int = 8080, json_response: bool = False):
    """Función principal del servidor."""
    logging.basicConfig(level=logging.INFO)

    app = create_mcp_server()

    # Crea gestor de sesión con modo sin estado para escalabilidad
    session_manager = StreamableHTTPSessionManager(
        app=app,
        event_store=None,
        json_response=json_response,
        stateless=True,  # Importante para escalabilidad en Cloud Run
    )

    async def handle_streamable_http(scope: Scope, receive: Receive, send: Send) -> None:
        await session_manager.handle_request(scope, receive, send)

    @contextlib.asynccontextmanager
    async def lifespan(app: Starlette) -> AsyncIterator[None]:
        """Gestiona el ciclo de vida del gestor de sesión."""
        async with session_manager.run():
            logger.info("MCP Streamable HTTP server started!")
            try:
                yield
            finally:
                logger.info("MCP server shutting down...")

    # Crea aplicación ASGI
    starlette_app = Starlette(
        debug=False,  # Configura a False para producción
        routes=[
            Mount("/mcp", app=handle_streamable_http),
        ],
        lifespan=lifespan,
    )

    import uvicorn
    uvicorn.run(starlette_app, host="0.0.0.0", port=port)

if __name__ == "__main__":
    main()
```

**Configuración del Agente para MCP Remoto:**
```python
# Tu agente ADK se conecta al servicio MCP remoto a través de HTTP Streamable
McpToolset(
    connection_params=StreamableHTTPConnectionParams(
        url="https://your-mcp-server-url.run.app/mcp",
        headers={"Authorization": "Bearer your-auth-token"}
    ),
)
```

#### Patrón