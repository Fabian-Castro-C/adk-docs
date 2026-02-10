---
catalog_title: Qdrant
catalog_description: Store and retrieve information using semantic vector search
catalog_icon: /assets/tools-qdrant.png
---

# Qdrant

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de Qdrant](https://github.com/qdrant/mcp-server-qdrant) conecta
tu agente ADK a [Qdrant](https://qdrant.tech/), un motor de búsqueda vectorial de código abierto. Esta integración le brinda a tu agente la capacidad de almacenar y
recuperar información usando búsqueda semántica.

## Casos de uso

- **Memoria Semántica para Agentes**: Almacena contexto de conversaciones, hechos o información
  aprendida que los agentes pueden recuperar más tarde usando consultas en lenguaje natural.

- **Búsqueda en Repositorios de Código**: Construye un índice buscable de fragmentos de código,
  documentación y patrones de implementación que se pueden consultar semánticamente.

- **Recuperación de Base de Conocimientos**: Crea un sistema de generación aumentada por recuperación (RAG)
  almacenando documentos y recuperando contexto relevante para las respuestas.

## Requisitos previos

- Una instancia de Qdrant en ejecución. Puedes:
    - Usar [Qdrant Cloud](https://cloud.qdrant.io/) (servicio administrado)
    - Ejecutar localmente con Docker: `docker run -p 6333:6333 qdrant/qdrant`
- (Opcional) Una clave API de Qdrant para autenticación

## Uso con el agente

=== "Python"

    === "Local MCP Server"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        QDRANT_URL = "http://localhost:6333"  # O tu URL de Qdrant Cloud
        COLLECTION_NAME = "my_collection"
        # QDRANT_API_KEY = "YOUR_QDRANT_API_KEY"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="qdrant_agent",
            instruction="Help users store and retrieve information using semantic search",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="uvx",
                            args=["mcp-server-qdrant"],
                            env={
                                "QDRANT_URL": QDRANT_URL,
                                "COLLECTION_NAME": COLLECTION_NAME,
                                # "QDRANT_API_KEY": QDRANT_API_KEY,
                            }
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

        const QDRANT_URL = "http://localhost:6333"; // O tu URL de Qdrant Cloud
        const COLLECTION_NAME = "my_collection";
        // const QDRANT_API_KEY = "YOUR_QDRANT_API_KEY";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "qdrant_agent",
            instruction: "Help users store and retrieve information using semantic search",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "uvx",
                        args: ["mcp-server-qdrant"],
                        env: {
                            QDRANT_URL: QDRANT_URL,
                            COLLECTION_NAME: COLLECTION_NAME,
                            // QDRANT_API_KEY: QDRANT_API_KEY,
                        },
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

## Herramientas disponibles

Herramienta | Descripción
---- | -----------
`qdrant-store` | Almacena información en Qdrant con metadatos opcionales
`qdrant-find` | Busca información relevante usando consultas en lenguaje natural

## Configuración

El servidor MCP de Qdrant se puede configurar usando variables de entorno:

Variable | Descripción | Por defecto
-------- | ----------- | -------
`QDRANT_URL` | URL del servidor de Qdrant | `None` (requerido)
`QDRANT_API_KEY` | Clave API para autenticación en Qdrant Cloud | `None`
`COLLECTION_NAME` | Nombre de la colección a usar | `None`
`QDRANT_LOCAL_PATH` | Ruta para almacenamiento persistente local (alternativa a URL) | `None`
`EMBEDDING_MODEL` | Modelo de embedding a usar | `sentence-transformers/all-MiniLM-L6-v2`
`EMBEDDING_PROVIDER` | Proveedor para embeddings (`fastembed` o `ollama`) | `fastembed`
`TOOL_STORE_DESCRIPTION` | Descripción personalizada para la herramienta store | Descripción por defecto
`TOOL_FIND_DESCRIPTION` | Descripción personalizada para la herramienta find | Descripción por defecto

### Descripciones de herramientas personalizadas

Puedes personalizar las descripciones de las herramientas para guiar el comportamiento del agente:

```python
env={
    "QDRANT_URL": "http://localhost:6333",
    "COLLECTION_NAME": "code-snippets",
    "TOOL_STORE_DESCRIPTION": "Store code snippets with descriptions. The 'information' parameter should contain a description of what the code does, while the actual code should be in 'metadata.code'.",
    "TOOL_FIND_DESCRIPTION": "Search for relevant code snippets using natural language. Describe the functionality you're looking for.",
}
```

## Recursos adicionales

- [Repositorio del Servidor MCP de Qdrant](https://github.com/qdrant/mcp-server-qdrant)
- [Documentación de Qdrant](https://qdrant.tech/documentation/)
- [Qdrant Cloud](https://cloud.qdrant.io/)