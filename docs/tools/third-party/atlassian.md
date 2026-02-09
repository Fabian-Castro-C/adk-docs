---
catalog_title: Atlassian
catalog_description: Manage issues, search pages, and update team content
catalog_icon: /adk-docs/assets/tools-atlassian.png
---

# Atlassian

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de Atlassian](https://github.com/atlassian/atlassian-mcp-server)
conecta tu agente ADK al ecosistema de [Atlassian](https://www.atlassian.com/),
cerrando la brecha entre el seguimiento de proyectos en Jira y la gestión del
conocimiento en Confluence. Esta integración le da a tu agente la capacidad de
gestionar incidencias, buscar y actualizar páginas de documentación, y optimizar
flujos de trabajo de colaboración usando lenguaje natural.

## Casos de uso

- **Búsqueda Unificada de Conocimiento**: Busca simultáneamente en incidencias de Jira y
  páginas de Confluence para encontrar especificaciones de proyectos, decisiones o contexto histórico.

- **Automatizar Gestión de Incidencias**: Crea, edita y transiciona incidencias de Jira, o
  agrega comentarios a tickets existentes.

- **Asistente de Documentación**: Recupera contenido de páginas, genera borradores, o agrega
  comentarios en línea a documentos de Confluence directamente desde tu agente.

## Requisitos previos

- Regístrate para una [cuenta de Atlassian](https://id.atlassian.com/signup)
- Un sitio de Atlassian Cloud con Jira y/o Confluence

## Usar con agente

=== "Python"

    === "Local MCP Server"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters


        root_agent = Agent(
            model="gemini-2.5-pro",
            name="atlassian_agent",
            instruction="Help users work with data in Atlassian products",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="npx",
                            args=[
                                "-y",
                                "mcp-remote",
                                "https://mcp.atlassian.com/v1/mcp",
                            ]
                        ),
                        timeout=30,
                    ),
                )
            ],
        )
        ```

=== "TypeScript"

    === "Local MCP Server"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "atlassian_agent",
            instruction: "Help users work with data in Atlassian products",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "npx",
                        args: [
                            "-y",
                            "mcp-remote",
                            "https://mcp.atlassian.com/v1/mcp",
                        ],
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

!!! note

    Cuando ejecutes este agente por primera vez, una ventana del navegador se abre
    automáticamente para solicitar acceso vía OAuth. Alternativamente, puedes usar la
    URL de autorización impresa en la consola. Debes aprobar esta solicitud para
    permitir que el agente acceda a tus datos de Atlassian.

## Herramientas disponibles

Tool | Description
---- | -----------
`atlassianUserInfo` | Obtener información sobre el usuario
`getAccessibleAtlassianResources` | Obtener información sobre recursos de Atlassian accesibles
`getJiraIssue` | Obtener información sobre una incidencia de Jira
`editJiraIssue` | Editar una incidencia de Jira
`createJiraIssue` | Crear una nueva incidencia de Jira
`getTransitionsForJiraIssue` | Obtener transiciones para una incidencia de Jira
`transitionJiraIssue` | Transicionar una incidencia de Jira
`lookupJiraAccountId` | Buscar un ID de cuenta de Jira
`searchJiraIssuesUsingJql` | Buscar incidencias de Jira usando JQL
`addCommentToJiraIssue` | Agregar un comentario a una incidencia de Jira
`getJiraIssueRemoteIssueLinks` | Obtener enlaces de incidencias remotas para una incidencia de Jira
`getVisibleJiraProjects` | Obtener proyectos de Jira visibles
`getJiraProjectIssueTypesMetadata` | Obtener metadatos de tipos de incidencias para un proyecto de Jira
`getJiraIssueTypeMetaWithFields` | Obtener metadatos de tipo de incidencia con campos para una incidencia de Jira
`getConfluenceSpaces` | Obtener información sobre espacios de Confluence
`getConfluencePage` | Obtener información sobre una página de Confluence
`getPagesInConfluenceSpace` | Obtener información sobre páginas en un espacio de Confluence
`getConfluencePageFooterComments` | Obtener información sobre comentarios de pie de página en una página de Confluence
`getConfluencePageInlineComments` | Obtener información sobre comentarios en línea en una página de Confluence
`getConfluencePageDescendants` | Obtener información sobre descendientes de una página de Confluence
`createConfluencePage` | Crear una nueva página de Confluence
`updateConfluencePage` | Actualizar una página de Confluence existente
`createConfluenceFooterComment` | Crear un comentario de pie de página en una página de Confluence
`createConfluenceInlineComment` | Crear un comentario en línea en una página de Confluence
`searchConfluenceUsingCql` | Buscar en Confluence usando CQL
`search` | Buscar información
`fetch` | Obtener información

## Recursos adicionales

- [Repositorio del Servidor MCP de Atlassian](https://github.com/atlassian/atlassian-mcp-server)
- [Documentación del Servidor MCP de Atlassian](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/)