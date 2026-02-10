---
catalog_title: Postman
catalog_description: Administra colecciones de API, espacios de trabajo y genera código de cliente
catalog_icon: /assets/tools-postman.png
---

# Postman

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de Postman](https://github.com/postmanlabs/postman-mcp-server)
conecta tu agente ADK al ecosistema de [Postman](https://www.postman.com/).
Esta integración le da a tu agente la capacidad de acceder a espacios de trabajo, administrar
colecciones y entornos, evaluar APIs y automatizar flujos de trabajo a través
de interacciones en lenguaje natural.

## Casos de uso

- **Pruebas de API**: Prueba continuamente tus APIs usando tus colecciones de Postman.

- **Administración de colecciones**: Crea y etiqueta colecciones, actualiza documentación,
  agrega comentarios o realiza acciones en múltiples colecciones sin salir
  de tu editor.

- **Administración de espacios de trabajo y entornos**: Crea espacios de trabajo y entornos,
  y administra tus variables de entorno.

- **Generación de código de cliente**: Genera código de cliente listo para producción que
  consume APIs siguiendo las mejores prácticas y convenciones del proyecto.

## Requisitos previos

- Crea una [cuenta de Postman](https://identity.getpostman.com/signup)
- Genera una [clave API de Postman](https://postman.postman.co/settings/me/api-keys)

## Uso con agente

=== "Python"

    === "Servidor MCP Local"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        POSTMAN_API_KEY = "YOUR_POSTMAN_API_KEY"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="postman_agent",
            instruction="Help users manage their Postman workspaces and collections",  # Ayuda a los usuarios a administrar sus espacios de trabajo y colecciones de Postman
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="npx",
                            args=[
                                "-y",
                                "@postman/postman-mcp-server",
                                # "--full",  # Usa todas las 100+ herramientas
                                # "--code",  # Usa herramientas de generación de código
                                # "--region", "eu",  # Usa región EU
                            ],
                            env={
                                "POSTMAN_API_KEY": POSTMAN_API_KEY,
                            },
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

        POSTMAN_API_KEY = "YOUR_POSTMAN_API_KEY"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="postman_agent",
            instruction="Help users manage their Postman workspaces and collections",  # Ayuda a los usuarios a administrar sus espacios de trabajo y colecciones de Postman
            tools=[
                McpToolset(
                    connection_params=StreamableHTTPServerParams(
                        url="https://mcp.postman.com/mcp",
                        # (Optional) Use "/minimal" for essential tools only
                        # (Optional) Use "/code" for code generation tools
                        # (Optional) Use "https://mcp.eu.postman.com" for EU region
                        # (Opcional) Usa "/minimal" solo para herramientas esenciales
                        # (Opcional) Usa "/code" para herramientas de generación de código
                        # (Opcional) Usa "https://mcp.eu.postman.com" para la región EU
                        headers={
                            "Authorization": f"Bearer {POSTMAN_API_KEY}",
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

        const POSTMAN_API_KEY = "YOUR_POSTMAN_API_KEY";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "postman_agent",
            instruction: "Help users manage their Postman workspaces and collections",  // Ayuda a los usuarios a administrar sus espacios de trabajo y colecciones de Postman
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "npx",
                        args: [
                            "-y",
                            "@postman/postman-mcp-server",
                            // "--full",  // Usa todas las 100+ herramientas
                            // "--code",  // Usa herramientas de generación de código
                            // "--region", "eu",  // Usa región EU
                        ],
                        env: {
                            POSTMAN_API_KEY: POSTMAN_API_KEY,
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

        const POSTMAN_API_KEY = "YOUR_POSTMAN_API_KEY";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "postman_agent",
            instruction: "Help users manage their Postman workspaces and collections",  // Ayuda a los usuarios a administrar sus espacios de trabajo y colecciones de Postman
            tools: [
                new MCPToolset({
                    type: "StreamableHTTPConnectionParams",
                    url: "https://mcp.postman.com/mcp",
                    // (Optional) Use "/minimal" for essential tools only
                    // (Optional) Use "/code" for code generation tools
                    // (Optional) Use "https://mcp.eu.postman.com" for EU region
                    // (Opcional) Usa "/minimal" solo para herramientas esenciales
                    // (Opcional) Usa "/code" para herramientas de generación de código
                    // (Opcional) Usa "https://mcp.eu.postman.com" para la región EU
                    header: {
                        Authorization: `Bearer ${POSTMAN_API_KEY}`,
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

## Configuración

Postman ofrece tres configuraciones de herramientas:

- **Mínima** (predeterminada): Herramientas esenciales para operaciones básicas de Postman. Mejor para
  modificaciones simples a colecciones, espacios de trabajo o entornos.
- **Completa**: Todas las herramientas API de Postman disponibles (más de 100 herramientas). Ideal para
  colaboración avanzada y características empresariales.
- **Código**: Herramientas para buscar definiciones de API y generar código de cliente.
  Perfecta para desarrolladores que necesitan consumir APIs.

Para seleccionar una configuración:

- **Servidor local**: Agrega `--full` o `--code` a la lista de `args`.
- **Servidor remoto**: Cambia la ruta de URL a `/minimal`, `/mcp` (completa) o `/code`.

Para la región EU, usa `--region eu` (local) o `https://mcp.eu.postman.com` (remoto).

## Recursos adicionales

- [Servidor MCP de Postman en GitHub](https://github.com/postmanlabs/postman-mcp-server)
- [Configuración de claves API de Postman](https://postman.postman.co/settings/me/api-keys)
- [Centro de Aprendizaje de Postman](https://learning.postman.com/)