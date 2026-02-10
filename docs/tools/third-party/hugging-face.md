---
catalog_title: Hugging Face
catalog_description: Access models, datasets, research papers, and AI tools
catalog_icon: /assets/tools-hugging-face.png
---

# Hugging Face

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de Hugging Face](https://github.com/huggingface/hf-mcp-server) puede utilizarse para conectar
tu agente ADK al Hugging Face Hub y miles de Aplicaciones de IA Gradio.

## Casos de uso

- **Descubrir Recursos de IA/ML**: Buscar y filtrar el Hub por modelos, conjuntos de datos y
  artículos basándose en tareas, bibliotecas o palabras clave.
- **Construir Flujos de Trabajo Multi-Paso**: Encadenar herramientas juntas, como transcribir
  audio con una herramienta y luego resumir el texto resultante con otra.
- **Encontrar Aplicaciones de IA**: Buscar Espacios de Gradio que puedan realizar una tarea
  específica, como eliminación de fondo o texto a voz.

## Requisitos previos

- Crear un [token de acceso de usuario](https://huggingface.co/settings/tokens) en
  Hugging Face. Consulta la
  [documentación](https://huggingface.co/docs/hub/en/security-tokens) para más
  información.

## Usar con agente

=== "Python"

    === "Servidor MCP Local"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        HUGGING_FACE_TOKEN = "YOUR_HUGGING_FACE_TOKEN"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="hugging_face_agent",
            instruction="Help users get information from Hugging Face",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params = StdioServerParameters(
                            command="npx",
                            args=[
                                "-y",
                                "@llmindset/hf-mcp-server",
                            ],
                            env={
                                "HF_TOKEN": HUGGING_FACE_TOKEN,
                            }
                        ),
                        timeout=30,
                    ),
                )
            ],
        )
        ```

    === "Servidor MCP Remoto"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StreamableHTTPServerParams

        HUGGING_FACE_TOKEN = "YOUR_HUGGING_FACE_TOKEN"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="hugging_face_agent",
            instruction="Help users get information from Hugging Face",
            tools=[
                McpToolset(
                    connection_params=StreamableHTTPServerParams(
                        url="https://huggingface.co/mcp",
                        headers={
                            "Authorization": f"Bearer {HUGGING_FACE_TOKEN}",
                        },
                    ),
                )
            ],
        )
        ```

=== "TypeScript"

    === "Servidor MCP Local"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const HUGGING_FACE_TOKEN = "YOUR_HUGGING_FACE_TOKEN";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "hugging_face_agent",
            instruction: "Help users get information from Hugging Face",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "npx",
                        args: ["-y", "@llmindset/hf-mcp-server"],
                        env: {
                            HF_TOKEN: HUGGING_FACE_TOKEN,
                        },
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

    === "Servidor MCP Remoto"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const HUGGING_FACE_TOKEN = "YOUR_HUGGING_FACE_TOKEN";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "hugging_face_agent",
            instruction: "Help users get information from Hugging Face",
            tools: [
                new MCPToolset({
                    type: "StreamableHTTPConnectionParams",
                    url: "https://huggingface.co/mcp",
                    header: {
                        Authorization: `Bearer ${HUGGING_FACE_TOKEN}`,
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

## Herramientas disponibles

Herramienta | Descripción
---- | -----------
Spaces Semantic Search | Encuentra las mejores Aplicaciones de IA mediante consultas en lenguaje natural
Papers Semantic Search | Encuentra Artículos de Investigación de ML mediante consultas en lenguaje natural
Model Search | Busca modelos de ML con filtros por tarea, biblioteca, etc.
Dataset Search | Busca conjuntos de datos con filtros por autor, etiquetas, etc.
Documentation Semantic Search | Busca en la biblioteca de documentación de Hugging Face
Hub Repository Details | Obtiene información detallada sobre Modelos, Conjuntos de datos y Espacios

## Configuración

Para configurar qué herramientas están disponibles en tu servidor MCP de Hugging Face Hub,
visita la [Página de Configuración MCP](https://huggingface.co/settings/mcp) en tu
cuenta de Hugging Face.


Para configurar el servidor MCP local, puedes usar las siguientes variables de
entorno:

- `TRANSPORT`: El tipo de transporte a utilizar (`stdio`, `sse`, `streamableHttp`, o
  `streamableHttpJson`)
- `DEFAULT_HF_TOKEN`: ⚠️ Las solicitudes se atienden con el `HF_TOKEN` recibido en
  el encabezado Authorization: Bearer. El DEFAULT_HF_TOKEN se usa si no se envió ningún encabezado.
  Solo establece esto en entornos de Desarrollo / Prueba o para Despliegues STDIO locales. ⚠️
- Si se ejecuta con transporte stdio, se usa `HF_TOKEN` si `DEFAULT_HF_TOKEN` no está
  establecido.
- `HF_API_TIMEOUT`: Tiempo de espera para solicitudes de la API de Hugging Face en milisegundos
  (por defecto: 12500ms / 12.5 segundos)
- `USER_CONFIG_API`: URL a usar para configuración de Usuario (por defecto al front-end Local)
- `MCP_STRICT_COMPLIANCE`: establecer en True para rechazos GET 405 en Modo JSON (por defecto
  sirve una página de bienvenida).
- `AUTHENTICATE_TOOL`: si se debe incluir una herramienta Authenticate para emitir un
  desafío OAuth cuando se llama
- `SEARCH_ENABLES_FETCH`: Cuando se establece en true, habilita automáticamente la
  herramienta hf_doc_fetch cada vez que hf_doc_search está habilitada


## Recursos adicionales

- [Repositorio del Servidor MCP de Hugging Face](https://github.com/huggingface/hf-mcp-server)
- [Documentación del Servidor MCP de Hugging Face](https://huggingface.co/docs/hub/en/hf-mcp-server)