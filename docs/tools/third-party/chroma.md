---
catalog_title: Chroma
catalog_description: Store and retrieve information using semantic vector search
catalog_icon: /assets/tools-chroma.png
---

# Chroma

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de Chroma](https://github.com/chroma-core/chroma-mcp) conecta tu
agente ADK a [Chroma](https://www.trychroma.com/), una base de datos de embeddings
de código abierto. Esta integración le da a tu agente la capacidad de crear colecciones,
almacenar documentos y recuperar información usando búsqueda semántica, búsqueda de texto
completo y filtrado por metadatos.

## Casos de uso

- **Memoria Semántica para Agentes**: Almacena contexto de conversación, hechos o información
  aprendida que los agentes pueden recuperar más tarde usando consultas en lenguaje natural.

- **Recuperación de Base de Conocimiento**: Construye un sistema de generación aumentada por
  recuperación (RAG) almacenando documentos y recuperando contexto relevante para las respuestas.

- **Contexto Persistente entre Sesiones**: Mantén memoria a largo plazo entre
  conversaciones, permitiendo a los agentes referenciar interacciones pasadas y conocimiento
  acumulado.

## Prerrequisitos

- **Para almacenamiento local**: Una ruta de directorio para persistir datos
- **Para Chroma Cloud**: Una cuenta de [Chroma Cloud](https://www.trychroma.com/) 
  con ID de tenant, nombre de base de datos y clave API

## Uso con agente

=== "Python"

    === "Servidor MCP Local"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        # Para almacenamiento local, usa:
        DATA_DIR = "/path/to/your/data/directory"

        # Para Chroma Cloud, usa:
        # CHROMA_TENANT = "your-tenant-id"
        # CHROMA_DATABASE = "your-database-name"
        # CHROMA_API_KEY = "your-api-key"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="chroma_agent",
            instruction="Help users store and retrieve information using semantic search",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="uvx",
                            args=[
                                "chroma-mcp",
                                # Para almacenamiento local, usa:
                                "--client-type",
                                "persistent",
                                "--data-dir",
                                DATA_DIR,
                                # Para Chroma Cloud, usa:
                                # "--client-type",
                                # "cloud",
                                # "--tenant",
                                # CHROMA_TENANT,
                                # "--database",
                                # CHROMA_DATABASE,
                                # "--api-key",
                                # CHROMA_API_KEY,
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

        // Para almacenamiento local, usa:
        const DATA_DIR = "/path/to/your/data/directory";

        // Para Chroma Cloud, usa:
        // const CHROMA_TENANT = "your-tenant-id";
        // const CHROMA_DATABASE = "your-database-name";
        // const CHROMA_API_KEY = "your-api-key";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "chroma_agent",
            instruction: "Help users store and retrieve information using semantic search",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "uvx",
                        args: [
                            "chroma-mcp",
                            // Para almacenamiento local, usa:
                            "--client-type",
                            "persistent",
                            "--data-dir",
                            DATA_DIR,
                            // Para Chroma Cloud, usa:
                            // "--client-type",
                            // "cloud",
                            // "--tenant",
                            // CHROMA_TENANT,
                            // "--database",
                            // CHROMA_DATABASE,
                            // "--api-key",
                            // CHROMA_API_KEY,
                        ],
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

## Herramientas disponibles

### Gestión de colecciones

Herramienta | Descripción
---- | -----------
`chroma_list_collections` | Lista todas las colecciones con soporte de paginación
`chroma_create_collection` | Crea una nueva colección con configuración HNSW opcional
`chroma_get_collection_info` | Obtiene información detallada sobre una colección
`chroma_get_collection_count` | Obtiene el número de documentos en una colección
`chroma_modify_collection` | Actualiza el nombre o metadatos de una colección
`chroma_delete_collection` | Elimina una colección
`chroma_peek_collection` | Visualiza una muestra de documentos en una colección

### Operaciones de documentos

Herramienta | Descripción
---- | -----------
`chroma_add_documents` | Agrega documentos con metadatos opcionales e IDs personalizados
`chroma_query_documents` | Consulta documentos usando búsqueda semántica con filtrado avanzado
`chroma_get_documents` | Recupera documentos por IDs o filtros con paginación
`chroma_update_documents` | Actualiza el contenido, metadatos o embeddings de documentos existentes
`chroma_delete_documents` | Elimina documentos específicos de una colección

## Configuración

El servidor MCP de Chroma soporta múltiples tipos de cliente para adaptarse a diferentes necesidades:

### Tipos de cliente

Tipo de Cliente | Descripción | Argumentos Clave
----------- | ----------- | -------------
`ephemeral` | Almacenamiento en memoria, se borra al reiniciar. Útil para pruebas. | Ninguno (predeterminado)
`persistent` | Almacenamiento basado en archivos en tu máquina local | `--data-dir`
`http` | Conecta a un servidor Chroma auto-hospedado | `--host`, `--port`, `--ssl`, `--custom-auth-credentials`
`cloud` | Conecta a Chroma Cloud (api.trychroma.com) | `--tenant`, `--database`, `--api-key`

### Variables de entorno

También puedes configurar el cliente usando variables de entorno. Los argumentos de línea de
comandos tienen precedencia sobre las variables de entorno.

Variable | Descripción
-------- | -----------
`CHROMA_CLIENT_TYPE` | Tipo de cliente: `ephemeral`, `persistent`, `http`, o `cloud`
`CHROMA_DATA_DIR` | Ruta para almacenamiento local persistente
`CHROMA_TENANT` | ID de tenant para Chroma Cloud
`CHROMA_DATABASE` | Nombre de base de datos para Chroma Cloud
`CHROMA_API_KEY` | Clave API para Chroma Cloud
`CHROMA_HOST` | Host para cliente HTTP auto-hospedado
`CHROMA_PORT` | Puerto para cliente HTTP auto-hospedado
`CHROMA_SSL` | Habilitar SSL para cliente HTTP (`true` o `false`)
`CHROMA_DOTENV_PATH` | Ruta al archivo `.env` (predeterminado `.chroma_env`)

## Recursos adicionales

- [Repositorio del Servidor MCP de Chroma](https://github.com/chroma-core/chroma-mcp)
- [Documentación de Chroma](https://docs.trychroma.com/)
- [Chroma Cloud](https://www.trychroma.com/)