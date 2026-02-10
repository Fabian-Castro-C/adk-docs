---
catalog_title: n8n
catalog_description: Trigger automated workflows, connect apps, and process data
catalog_icon: /assets/tools-n8n.png
---

# n8n

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de n8n](https://docs.n8n.io/advanced-ai/accessing-n8n-mcp-server/)
conecta tu agente ADK a [n8n](https://n8n.io/), una herramienta de automatización
de flujos de trabajo extensible. Esta integración permite que tu agente se conecte
de forma segura a una instancia de n8n para buscar, inspeccionar y activar flujos
de trabajo directamente desde una interfaz de lenguaje natural.

!!! note "Alternativa: Servidor MCP a nivel de flujo de trabajo"

    La guía de configuración en esta página cubre el **acceso MCP a nivel de instancia**,
    que conecta tu agente a un centro central de flujos de trabajo habilitados.
    Alternativamente, puedes usar el
    [nodo de activación del Servidor MCP](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-langchain.mcptrigger/)
    para hacer que un **único flujo de trabajo** actúe como su propio servidor MCP independiente. Este
    método es útil si deseas crear comportamientos específicos del servidor o exponer
    herramientas aisladas a un flujo de trabajo.

## Casos de uso

- **Ejecutar flujos de trabajo complejos**: Activa procesos de negocio de múltiples pasos definidos
  en n8n directamente desde tu agente, aprovechando lógica de ramificación confiable, bucles
  y manejo de errores para garantizar la consistencia.

- **Conectar a aplicaciones externas**: Accede a integraciones preconfiguradas a través de n8n
  sin escribir herramientas personalizadas para cada servicio, eliminando la necesidad de gestionar
  autenticación de API, encabezados o código repetitivo.

- **Procesamiento de datos**: Delega tareas complejas de transformación de datos a flujos de trabajo
  de n8n, como convertir lenguaje natural en llamadas API o extraer y
  resumir páginas web, utilizando nodos personalizados de Python o JavaScript para dar forma precisa
  a los datos.

## Requisitos previos

- Una instancia activa de n8n
- Acceso MCP habilitado en la configuración
- Un token de acceso MCP válido

Consulta la
[documentación MCP de n8n](https://docs.n8n.io/advanced-ai/accessing-n8n-mcp-server/)
para obtener instrucciones detalladas de configuración.

## Uso con agente

=== "Python"

    === "Servidor MCP local"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        N8N_INSTANCE_URL = "https://localhost:5678"
        N8N_MCP_TOKEN = "YOUR_N8N_MCP_TOKEN"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="n8n_agent",
            instruction="Help users manage and execute workflows in n8n",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="npx",
                            args=[
                                "-y",
                                "supergateway",
                                "--streamableHttp",
                                f"{N8N_INSTANCE_URL}/mcp-server/http",
                                "--header",
                                f"authorization:Bearer {N8N_MCP_TOKEN}"
                            ]
                        ),
                        timeout=300,
                    ),
                )
            ],
        )
        ```

    === "Servidor MCP remoto"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StreamableHTTPServerParams

        N8N_INSTANCE_URL = "https://localhost:5678"
        N8N_MCP_TOKEN = "YOUR_N8N_MCP_TOKEN"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="n8n_agent",
            instruction="Help users manage and execute workflows in n8n",
            tools=[
                McpToolset(
                    connection_params=StreamableHTTPServerParams(
                        url=f"{N8N_INSTANCE_URL}/mcp-server/http",
                        headers={
                            "Authorization": f"Bearer {N8N_MCP_TOKEN}",
                        },
                    ),
                )
            ],
        )
        ```

=== "TypeScript"

    === "Servidor MCP local"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const N8N_INSTANCE_URL = "https://localhost:5678";
        const N8N_MCP_TOKEN = "YOUR_N8N_MCP_TOKEN";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "n8n_agent",
            instruction: "Help users manage and execute workflows in n8n",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "npx",
                        args: [
                            "-y",
                            "supergateway",
                            "--streamableHttp",
                            `${N8N_INSTANCE_URL}/mcp-server/http`,
                            "--header",
                            `authorization:Bearer ${N8N_MCP_TOKEN}`,
                        ],
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

    === "Servidor MCP remoto"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const N8N_INSTANCE_URL = "https://localhost:5678";
        const N8N_MCP_TOKEN = "YOUR_N8N_MCP_TOKEN";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "n8n_agent",
            instruction: "Help users manage and execute workflows in n8n",
            tools: [
                new MCPToolset({
                    type: "StreamableHTTPConnectionParams",
                    url: `${N8N_INSTANCE_URL}/mcp-server/http`,
                    header: {
                        Authorization: `Bearer ${N8N_MCP_TOKEN}`,
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

## Herramientas disponibles

Herramienta | Descripción
---- | -----------
`search_workflows` | Buscar flujos de trabajo disponibles
`execute_workflow` | Ejecutar un flujo de trabajo específico
`get_workflow_details` | Recuperar metadatos e información de esquema para un flujo de trabajo

## Configuración

Para que los flujos de trabajo sean accesibles para tu agente, deben cumplir con los siguientes
criterios:

- **Estar activos**: El flujo de trabajo debe estar activado en n8n.

- **Activador soportado**: Contener un nodo de activación Webhook, Schedule, Chat o Form.

- **Habilitado para MCP**: Debes activar "Available in MCP" en la configuración
  del flujo de trabajo o seleccionar "Enable MCP access" desde el menú de la tarjeta del flujo de trabajo.

## Recursos adicionales

- [Documentación del Servidor MCP de n8n](https://docs.n8n.io/advanced-ai/accessing-n8n-mcp-server/)