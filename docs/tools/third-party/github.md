---
catalog_title: GitHub
catalog_description: Analyze code, manage issues and PRs, and automate workflows
catalog_icon: /assets/tools-github.png
---

# GitHub

<div class="language-support-tag">
  <span class="lst-supported">Compatible con ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de GitHub](https://github.com/github/github-mcp-server) conecta herramientas
de IA directamente a la plataforma de GitHub. Esto le da a tu agente ADK la capacidad de
leer repositorios y archivos de código, gestionar issues y PRs, analizar código, y
automatizar flujos de trabajo usando lenguaje natural.

## Casos de uso

- **Gestión de Repositorios**: Navega y consulta código, busca archivos, analiza
  commits, y comprende la estructura del proyecto en cualquier repositorio al que tengas
  acceso.
- **Automatización de Issues y PRs**: Crea, actualiza y gestiona issues y pull
  requests. Deja que la IA ayude a clasificar bugs, revisar cambios de código y mantener tableros de
  proyectos.
- **Análisis de Código**: Examina hallazgos de seguridad, revisa alertas de Dependabot,
  comprende patrones de código, y obtén información completa sobre tu base de código.

## Requisitos previos

- Crea un
  [Token de Acceso Personal](https://github.com/settings/personal-access-tokens/new) en GitHub. Consulta la [documentación](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) para más información.

## Uso con agente

=== "Python"

    === "Servidor MCP Remoto"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StreamableHTTPServerParams

        GITHUB_TOKEN = "YOUR_GITHUB_TOKEN"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="github_agent",
            instruction="Help users get information from GitHub",
            tools=[
                McpToolset(
                    connection_params=StreamableHTTPServerParams(
                        url="https://api.githubcopilot.com/mcp/",
                        headers={
                            "Authorization": f"Bearer {GITHUB_TOKEN}",
                            "X-MCP-Toolsets": "all",
                            "X-MCP-Readonly": "true"
                        },
                    ),
                )
            ],
        )
        ```

=== "TypeScript"

    === "Servidor MCP Remoto"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const GITHUB_TOKEN = "YOUR_GITHUB_TOKEN";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "github_agent",
            instruction: "Help users get information from GitHub",
            tools: [
                new MCPToolset({
                    type: "StreamableHTTPConnectionParams",
                    url: "https://api.githubcopilot.com/mcp/",
                    header: {
                        Authorization: `Bearer ${GITHUB_TOKEN}`,
                        "X-MCP-Toolsets": "all",
                        "X-MCP-Readonly": "true",
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

## Herramientas disponibles

Herramienta | Descripción
---- | -----------
`context` | Herramientas que proporcionan contexto sobre el usuario actual y el contexto de GitHub en el que estás operando
`copilot` | Herramientas relacionadas con Copilot (ej. Agente de Codificación de Copilot)
`copilot_spaces` | Herramientas relacionadas con Copilot Spaces
`actions` | Flujos de trabajo de GitHub Actions y operaciones de CI/CD
`code_security` | Herramientas relacionadas con seguridad de código, como GitHub Code Scanning
`dependabot` | Herramientas de Dependabot
`discussions` | Herramientas relacionadas con GitHub Discussions
`experiments` | Características experimentales que aún no se consideran estables
`gists` | Herramientas relacionadas con GitHub Gist
`github_support_docs_search` | Buscar documentos para responder preguntas sobre productos y soporte de GitHub
`issues` | Herramientas relacionadas con GitHub Issues
`labels` | Herramientas relacionadas con etiquetas de GitHub
`notifications` | Herramientas relacionadas con notificaciones de GitHub
`orgs` | Herramientas relacionadas con organizaciones de GitHub
`projects` | Herramientas relacionadas con proyectos de GitHub
`pull_requests` | Herramientas relacionadas con Pull Requests de GitHub
`repos` | Herramientas relacionadas con repositorios de GitHub
`secret_protection` | Herramientas relacionadas con protección de secretos, como GitHub Secret Scanning
`security_advisories` | Herramientas relacionadas con avisos de seguridad
`stargazers` | Herramientas relacionadas con Stargazers de GitHub
`users` | Herramientas relacionadas con usuarios de GitHub

## Configuración

El servidor MCP remoto de GitHub tiene encabezados opcionales que se pueden usar para configurar
los conjuntos de herramientas disponibles y el modo de solo lectura:

- `X-MCP-Toolsets`: Lista separada por comas de conjuntos de herramientas a habilitar. (ej. "repos,issues")
    - Si la lista está vacía, se usarán los conjuntos de herramientas predeterminados. Si se proporciona
      un conjunto de herramientas incorrecto, el servidor fallará al iniciar y emitirá un estado de solicitud incorrecta 400.
      Los espacios en blanco se ignoran.

- `X-MCP-Readonly`: Habilita solo herramientas de "lectura".
    - Si este encabezado está vacío, es "false", "f", "no", "n", "0", o "off" (ignorando
      espacios en blanco y mayúsculas), se interpretará como falso. Todos los demás valores
      se interpretan como verdadero.


## Recursos adicionales

- [Repositorio del Servidor MCP de GitHub](https://github.com/github/github-mcp-server)
- [Documentación del Servidor MCP Remoto de GitHub](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md)
- [Políticas y Gobernanza para el Servidor MCP de GitHub](https://github.com/github/github-mcp-server/blob/main/docs/policies-and-governance.md)