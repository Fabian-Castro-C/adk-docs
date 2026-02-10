---
catalog_title: MongoDB
catalog_description: Query collections, manage databases, and analyze schemas
catalog_icon: /assets/tools-mongodb.png
---

# MongoDB

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [servidor MCP de MongoDB](https://github.com/mongodb-js/mongodb-mcp-server)
conecta tu agente ADK a bases de datos [MongoDB](https://www.mongodb.com/) y
clústeres de MongoDB Atlas. Esta integración le da a tu agente la capacidad de consultar
colecciones, administrar bases de datos e interactuar con la infraestructura de MongoDB Atlas
usando lenguaje natural.

## Casos de uso

- **Exploración y análisis de datos**: Consulta colecciones de MongoDB usando lenguaje
  natural, ejecuta agregaciones y analiza esquemas de documentos sin escribir
  consultas complejas manualmente.

- **Administración de bases de datos**: Lista bases de datos y colecciones, crea índices,
  administra usuarios y monitorea estadísticas de bases de datos a través de comandos conversacionales.

- **Gestión de infraestructura Atlas**: Crea y administra clústeres de MongoDB Atlas,
  configura listas de acceso y visualiza recomendaciones de rendimiento directamente desde
  tu agente.

## Prerrequisitos

- **Para acceso a bases de datos**: Una cadena de conexión de MongoDB (local, auto-hospedada o
  clúster de Atlas)
- **Para gestión de Atlas**: Una cuenta de servicio de [MongoDB Atlas](https://www.mongodb.com/atlas)
  con credenciales de API (ID de cliente y secreto)

## Uso con el agente

=== "Python"

    === "Local MCP Server"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        # Para acceso a bases de datos, usa una cadena de conexión:
        CONNECTION_STRING = "mongodb://localhost:27017/myDatabase"

        # Para gestión de Atlas, usa credenciales de API:
        # ATLAS_CLIENT_ID = "YOUR_ATLAS_CLIENT_ID"
        # ATLAS_CLIENT_SECRET = "YOUR_ATLAS_CLIENT_SECRET"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="mongodb_agent",
            instruction="Help users query and manage MongoDB databases",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="npx",
                            args=[
                                "-y",
                                "mongodb-mcp-server",
                                "--readOnly",  # Eliminar para operaciones de escritura
                            ],
                            env={
                                # Para acceso a bases de datos, usa:
                                "MDB_MCP_CONNECTION_STRING": CONNECTION_STRING,
                                # Para gestión de Atlas, usa:
                                # "MDB_MCP_API_CLIENT_ID": ATLAS_CLIENT_ID,
                                # "MDB_MCP_API_CLIENT_SECRET": ATLAS_CLIENT_SECRET,
                            },
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

        // Para acceso a bases de datos, usa una cadena de conexión:
        const CONNECTION_STRING = "mongodb://localhost:27017/myDatabase";

        // Para gestión de Atlas, usa credenciales de API:
        // const ATLAS_CLIENT_ID = "YOUR_ATLAS_CLIENT_ID";
        // const ATLAS_CLIENT_SECRET = "YOUR_ATLAS_CLIENT_SECRET";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "mongodb_agent",
            instruction: "Help users query and manage MongoDB databases",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "npx",
                        args: [
                            "-y",
                            "mongodb-mcp-server",
                            "--readOnly", // Eliminar para operaciones de escritura
                        ],
                        env: {
                            // Para acceso a bases de datos, usa:
                            MDB_MCP_CONNECTION_STRING: CONNECTION_STRING,
                            // Para gestión de Atlas, usa:
                            // MDB_MCP_API_CLIENT_ID: ATLAS_CLIENT_ID,
                            // MDB_MCP_API_CLIENT_SECRET: ATLAS_CLIENT_SECRET,
                        },
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

## Herramientas disponibles

### Herramientas de bases de datos MongoDB

Herramienta | Descripción
----------- | -----------
`find` | Ejecuta una consulta find contra una colección de MongoDB
`aggregate` | Ejecuta una agregación contra una colección de MongoDB
`count` | Obtiene el número de documentos en una colección
`list-databases` | Lista todas las bases de datos para una conexión de MongoDB
`list-collections` | Lista todas las colecciones para una base de datos dada
`collection-schema` | Describe el esquema para una colección
`collection-indexes` | Describe los índices para una colección
`insert-many` | Inserta documentos en una colección
`update-many` | Actualiza documentos que coinciden con un filtro
`delete-many` | Elimina documentos que coinciden con un filtro
`create-collection` | Crea una nueva colección
`drop-collection` | Elimina una colección de la base de datos
`drop-database` | Elimina una base de datos
`create-index` | Crea un índice para una colección
`drop-index` | Elimina un índice de una colección
`rename-collection` | Renombra una colección
`db-stats` | Obtiene estadísticas para una base de datos
`explain` | Obtiene estadísticas de ejecución de consultas
`export` | Exporta resultados de consultas en formato EJSON

### Herramientas de MongoDB Atlas

!!! note

    Las herramientas de Atlas requieren credenciales de API. Establece las variables de entorno `MDB_MCP_API_CLIENT_ID` y
    `MDB_MCP_API_CLIENT_SECRET` para habilitarlas.

Herramienta | Descripción
----------- | -----------
`atlas-list-orgs` | Lista organizaciones de MongoDB Atlas
`atlas-list-projects` | Lista proyectos de MongoDB Atlas
`atlas-list-clusters` | Lista clústeres de MongoDB Atlas
`atlas-inspect-cluster` | Inspecciona metadatos de un clúster
`atlas-list-db-users` | Lista usuarios de base de datos
`atlas-create-free-cluster` | Crea un clúster gratuito de Atlas
`atlas-create-project` | Crea un proyecto de Atlas
`atlas-create-db-user` | Crea un usuario de base de datos
`atlas-create-access-list` | Configura lista de acceso IP
`atlas-inspect-access-list` | Visualiza entradas de lista de acceso IP
`atlas-list-alerts` | Lista alertas de Atlas
`atlas-get-performance-advisor` | Obtiene recomendaciones de rendimiento

## Configuración

### Variables de entorno

Variable | Descripción
-------- | -----------
`MDB_MCP_CONNECTION_STRING` | Cadena de conexión de MongoDB para acceso a bases de datos
`MDB_MCP_API_CLIENT_ID` | ID de cliente de API de Atlas para herramientas de Atlas
`MDB_MCP_API_CLIENT_SECRET` | Secreto de cliente de API de Atlas para herramientas de Atlas
`MDB_MCP_READ_ONLY` | Habilita modo de solo lectura (`true` o `false`)
`MDB_MCP_DISABLED_TOOLS` | Lista separada por comas de herramientas a deshabilitar
`MDB_MCP_LOG_PATH` | Directorio para archivos de registro

### Modo de solo lectura

La bandera `--readOnly` restringe el servidor a operaciones de lectura, conexión y metadatos
solamente. Esto previene cualquier operación de creación, actualización o eliminación,
haciéndolo seguro para exploración de datos sin riesgo de modificaciones accidentales.

### Deshabilitar herramientas

Puedes deshabilitar herramientas específicas o categorías usando `MDB_MCP_DISABLED_TOOLS`:

- Nombres de herramientas: `find`, `aggregate`, `insert-many`, etc.
- Categorías: `atlas` (todas las herramientas de Atlas), `mongodb` (todas las herramientas de base de datos)
- Tipos de operación: `create`, `update`, `delete`, `read`, `metadata`

## Recursos adicionales

- [Repositorio del servidor MCP de MongoDB](https://github.com/mongodb-js/mongodb-mcp-server)
- [Documentación de MongoDB](https://www.mongodb.com/docs/)
- [MongoDB Atlas](https://www.mongodb.com/atlas)