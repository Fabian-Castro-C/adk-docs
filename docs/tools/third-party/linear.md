---
catalog_title: Linear
catalog_description: Manage issues, track projects, and streamline development
catalog_icon: /adk-docs/assets/tools-linear.png
---

# Linear

<div class="language-support-tag">
  <span class="lst-supported">Compatible con ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de Linear](https://linear.app/docs/mcp) conecta tu agente ADK a
[Linear](https://linear.app/), una herramienta diseñada específicamente para planificar y construir
productos. Esta integración le da a tu agente la capacidad de gestionar issues, rastrear
ciclos de proyectos y automatizar flujos de trabajo de desarrollo usando lenguaje natural.

## Casos de uso

- **Optimiza la Gestión de Issues**: Crea, actualiza y organiza issues usando
  lenguaje natural. Deja que tu agente se encargue de registrar bugs, asignar tareas y
  actualizar estados.

- **Rastrea Proyectos y Ciclos**: Obtén visibilidad instantánea del progreso de tu equipo.
  Consulta el estado de ciclos activos, verifica hitos de proyectos y
  recupera fechas límite.

- **Búsqueda Contextual y Resumen**: Ponte al día rápidamente con hilos de discusión largos
  o encuentra especificaciones específicas de proyectos. Tu agente puede buscar
  documentación y resumir issues complejos.

## Requisitos previos

- [Regístrate](https://linear.app/signup) para obtener una cuenta de Linear
- Genera una clave API en
  [Configuración de Linear > Seguridad y acceso](https://linear.app/docs/security-and-access)
  (si usas autenticación API)

## Uso con agente

=== "Python"

    === "Servidor MCP Local"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="linear_agent",
            instruction="Help users manage issues, projects, and cycles in Linear",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="npx",
                            args=[
                                "-y",
                                "mcp-remote",
                                "https://mcp.linear.app/mcp",
                            ]
                        ),
                        timeout=30,
                    ),
                )
            ],
        )
        ```

        !!! note

            Cuando ejecutes este agente por primera vez, se abrirá automáticamente una ventana del navegador
            para solicitar acceso vía OAuth. Alternativamente, puedes usar
            la URL de autorización impresa en la consola. Debes aprobar esta
            solicitud para permitir que el agente acceda a tus datos de Linear.

    === "Servidor MCP Remoto"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StreamableHTTPServerParams

        LINEAR_API_KEY = "YOUR_LINEAR_API_KEY"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="linear_agent",
            instruction="Help users manage issues, projects, and cycles in Linear",
            tools=[
                McpToolset(
                    connection_params=StreamableHTTPServerParams(
                        url="https://mcp.linear.app/mcp",
                        headers={
                            "Authorization": f"Bearer {LINEAR_API_KEY}",
                        },
                    ),
                )
            ],
        )
        ```

        !!! note

            Este ejemplo de código usa una clave API para autenticación. Para usar un
            flujo de autenticación OAuth basado en navegador en su lugar, elimina el parámetro `headers`
            y ejecuta el agente.

=== "TypeScript"

    === "Servidor MCP Local"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "linear_agent",
            instruction: "Help users manage issues, projects, and cycles in Linear",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "npx",
                        args: ["-y", "mcp-remote", "https://mcp.linear.app/mcp"],
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

        !!! note

            Cuando ejecutes este agente por primera vez, se abrirá automáticamente una ventana del navegador
            para solicitar acceso vía OAuth. Alternativamente, puedes usar
            la URL de autorización impresa en la consola. Debes aprobar esta
            solicitud para permitir que el agente acceda a tus datos de Linear.

    === "Servidor MCP Remoto"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const LINEAR_API_KEY = "YOUR_LINEAR_API_KEY";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "linear_agent",
            instruction: "Help users manage issues, projects, and cycles in Linear",
            tools: [
                new MCPToolset({
                    type: "StreamableHTTPConnectionParams",
                    url: "https://mcp.linear.app/mcp",
                    header: {
                        Authorization: `Bearer ${LINEAR_API_KEY}`,
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

        !!! note

            Este ejemplo de código usa una clave API para autenticación. Para usar un
            flujo de autenticación OAuth basado en navegador en su lugar, elimina la propiedad `header`
            y ejecuta el agente.

## Herramientas disponibles

Herramienta | Descripción
---- | -----------
`list_comments` | Lista los comentarios en un issue
`create_comment` | Crea un comentario en un issue
`list_cycles` | Lista los ciclos en un proyecto
`get_document` | Obtiene un documento
`list_documents` | Lista los documentos
`get_issue` | Obtiene un issue
`list_issues` | Lista los issues
`create_issue` | Crea un issue
`update_issue` | Actualiza un issue
`list_issue_statuses` | Lista los estados de issues
`get_issue_status` | Obtiene un estado de issue
`list_issue_labels` | Lista las etiquetas de issues
`create_issue_label` | Crea una etiqueta de issue
`list_projects` | Lista los proyectos
`get_project` | Obtiene un proyecto
`create_project` | Crea un proyecto
`update_project` | Actualiza un proyecto
`list_project_labels` | Lista las etiquetas de proyectos
`list_teams` | Lista los equipos
`get_team` | Obtiene un equipo
`list_users` | Lista los usuarios
`get_user` | Obtiene un usuario
`search_documentation` | Busca en la documentación

## Recursos adicionales

- [Documentación del Servidor MCP de Linear](https://linear.app/docs/mcp)
- [Guía de Inicio de Linear](https://linear.app/docs/start-guide)