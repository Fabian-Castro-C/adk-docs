---
catalog_title: GitLab
catalog_description: Realiza búsquedas de código semánticas, inspecciona pipelines, gestiona merge requests
catalog_icon: /assets/tools-gitlab.png
---

# GitLab

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El
[Servidor MCP de GitLab](https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/mcp_server/)
conecta tu agente ADK directamente a [GitLab.com](https://gitlab.com/) o tu
instancia de GitLab autogestionada. Esta integración le da a tu agente la capacidad de
gestionar issues y merge requests, inspeccionar pipelines de CI/CD, realizar búsquedas de código
semánticas, y automatizar flujos de trabajo de desarrollo usando lenguaje natural.

## Casos de uso

- **Exploración de Código Semántica**: Navega tu código base usando lenguaje natural.
  A diferencia de la búsqueda de texto estándar, puedes consultar la lógica e intención de tu código
  para comprender rápidamente implementaciones complejas.

- **Acelera Revisiones de Merge Request**: Ponte al día sobre cambios de código
  instantáneamente. Recupera contextos completos de merge requests, analiza diffs específicos, y
  revisa el historial de commits para proporcionar retroalimentación más rápida y significativa a tu
  equipo.

- **Soluciona Problemas de Pipelines de CI/CD**: Diagnostica fallos de compilación sin salir de tu
  chat. Inspecciona estados de pipelines y recupera logs de trabajos detallados para identificar
  exactamente por qué un merge request o commit específico falló sus verificaciones.

## Prerrequisitos

- Una cuenta de GitLab con una suscripción Premium o Ultimate y
  [GitLab Duo](https://docs.gitlab.com/user/gitlab_duo/) habilitado
- [Características beta y experimentales](https://docs.gitlab.com/user/gitlab_duo/turn_on_off/#turn-on-beta-and-experimental-features)
  habilitadas en tu configuración de GitLab

## Usar con agente

=== "Python"

    === "Servidor MCP Local"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        # Reemplaza con la URL de tu instancia si es auto-hospedada (ej., "gitlab.example.com")
        GITLAB_INSTANCE_URL = "gitlab.com"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="gitlab_agent",
            instruction="Help users get information from GitLab",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params = StdioServerParameters(
                            command="npx",
                            args=[
                                "-y",
                                "mcp-remote",
                                f"https://{GITLAB_INSTANCE_URL}/api/v4/mcp",
                                "--static-oauth-client-metadata",
                                "{\"scope\": \"mcp\"}",
                            ],
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

        // Reemplaza con la URL de tu instancia si es auto-hospedada (ej., "gitlab.example.com")
        const GITLAB_INSTANCE_URL = "gitlab.com";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "gitlab_agent",
            instruction: "Help users get information from GitLab",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "npx",
                        args: [
                            "-y",
                            "mcp-remote",
                            `https://${GITLAB_INSTANCE_URL}/api/v4/mcp`,
                            "--static-oauth-client-metadata",
                            '{"scope": "mcp"}',
                        ],
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

!!! note

    Cuando ejecutes este agente por primera vez, se abrirá automáticamente una ventana del navegador
    (y se imprimirá una URL de autorización) solicitando permisos OAuth. Debes aprobar esta solicitud para permitir que el agente acceda a tus
    datos de GitLab.

## Herramientas disponibles

Herramienta | Descripción
---- | -----------
`get_mcp_server_version` | Devuelve la versión actual del servidor MCP de GitLab
`create_issue` | Crea un nuevo issue en un proyecto de GitLab
`get_issue` | Recupera información detallada sobre un issue específico de GitLab
`create_merge_request` | Crea un merge request en un proyecto
`get_merge_request` | Recupera información detallada sobre un merge request específico de GitLab
`get_merge_request_commits` | Recupera la lista de commits en un merge request específico
`get_merge_request_diffs` | Recupera los diffs para un merge request específico
`get_merge_request_pipelines` | Recupera los pipelines para un merge request específico
`get_pipeline_jobs` | Recupera los trabajos para un pipeline de CI/CD específico
`gitlab_search` | Busca un término en toda la instancia de GitLab con la API de búsqueda
`semantic_code_search` | Busca fragmentos de código relevantes en un proyecto

## Recursos adicionales

- [Documentación del Servidor MCP de GitLab](https://docs.gitlab.com/user/gitlab_duo/model_context_protocol/mcp_server/)