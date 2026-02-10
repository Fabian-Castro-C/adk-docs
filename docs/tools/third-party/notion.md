---
catalog_title: Notion
catalog_description: Search workspaces, create pages, and manage tasks and databases
catalog_icon: /assets/tools-notion.png
---

# Notion

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de Notion](https://github.com/makenotion/notion-mcp-server)
conecta tu agente ADK a Notion, permitiéndole buscar, crear y gestionar
páginas, bases de datos y más dentro de un espacio de trabajo. Esto le da a tu agente la capacidad
de consultar, crear y organizar contenido en tu espacio de trabajo de Notion usando lenguaje
natural.

## Casos de uso

- **Busca en tu espacio de trabajo**: Encuentra páginas de proyectos, notas de reuniones o documentos
  basándote en el contenido.

- **Crea nuevo contenido**: Genera nuevas páginas para notas de reuniones, planes de proyectos
  o tareas.

- **Gestiona tareas y bases de datos**: Actualiza el estado de una tarea, agrega elementos a una
  base de datos o cambia propiedades.

- **Organiza tu espacio de trabajo**: Mueve páginas, duplica plantillas o agrega comentarios
  a documentos.

## Requisitos previos

- Obtén un token de integración de Notion yendo a
  [Integraciones de Notion](https://www.notion.so/profile/integrations) en tu
  perfil. Consulta la
  [documentación de autorización](https://developers.notion.com/docs/authorization)
  para más detalles.
- Asegúrate de que las páginas y bases de datos relevantes puedan ser accedidas por tu integración. Visita
  la pestaña de Acceso en la configuración de tu
  [Integración de Notion](https://www.notion.so/profile/integrations),
  luego otorga acceso seleccionando las páginas que deseas usar.

## Uso con agente

=== "Python"

    === "Servidor MCP Local"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        NOTION_TOKEN = "YOUR_NOTION_TOKEN"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="notion_agent",
            instruction="Help users get information from Notion", # Ayuda a los usuarios a obtener información de Notion
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params = StdioServerParameters(
                            command="npx",
                            args=[
                                "-y",
                                "@notionhq/notion-mcp-server",
                            ],
                            env={
                                "NOTION_TOKEN": NOTION_TOKEN,
                            }
                        ),
                        timeout=30,
                    ),
                )
            ],
        )
        ```

=== "TypeScript"

    === "Servidor MCP Local"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const NOTION_TOKEN = "YOUR_NOTION_TOKEN";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "notion_agent",
            instruction: "Help users get information from Notion", // Ayuda a los usuarios a obtener información de Notion
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "npx",
                        args: ["-y", "@notionhq/notion-mcp-server"],
                        env: {
                            NOTION_TOKEN: NOTION_TOKEN,
                        },
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

## Herramientas disponibles

Herramienta <img width="200px"/> | Descripción
---- | -----------
`notion-search` | Busca en tu espacio de trabajo de Notion y herramientas conectadas como Slack, Google Drive y Jira. Recurre a la búsqueda básica del espacio de trabajo si las características de IA no están disponibles.
`notion-fetch` | Recupera contenido de una página o base de datos de Notion por su URL
`notion-create-pages` | Crea una o más páginas de Notion con propiedades y contenido especificados.
`notion-update-page` | Actualiza las propiedades o contenido de una página de Notion.
`notion-move-pages` | Mueve una o más páginas o bases de datos de Notion a un nuevo padre.
`notion-duplicate-page` | Duplica una página de Notion dentro de tu espacio de trabajo. Esta acción se completa de forma asíncrona.
`notion-create-database` | Crea una nueva base de datos de Notion, fuente de datos inicial y vista inicial con las propiedades especificadas.
`notion-update-database` | Actualiza las propiedades, nombre, descripción u otros atributos de una fuente de datos de Notion.
`notion-create-comment` | Agrega un comentario a una página
`notion-get-comments` | Lista todos los comentarios en una página específica, incluyendo discusiones en hilos.
`notion-get-teams` | Recupera una lista de equipos (espacios de equipo) en el espacio de trabajo actual.
`notion-get-users` | Lista todos los usuarios en el espacio de trabajo con sus detalles.
`notion-get-user` | Recupera la información de tu usuario por ID
`notion-get-self` | Recupera información sobre tu propio usuario bot y el espacio de trabajo de Notion al que estás conectado.

## Recursos adicionales

- [Documentación del Servidor MCP de Notion](https://developers.notion.com/docs/mcp)
- [Repositorio del Servidor MCP de Notion](https://github.com/makenotion/notion-mcp-server)