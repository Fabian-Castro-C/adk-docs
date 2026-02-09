# Protocolo de Contexto de Modelo (MCP)

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span><span class="lst-typescript">TypeScript</span><span class="lst-go">Go</span><span class="lst-java">Java</span>
</div>

El
[Protocolo de Contexto de Modelo (MCP)](https://modelcontextprotocol.io/introduction) es
un estándar abierto diseñado para estandarizar cómo los Modelos de Lenguaje de Gran Escala (LLMs) como
Gemini y Claude se comunican con aplicaciones externas, fuentes de datos y
herramientas. Piénsalo como un mecanismo de conexión universal que simplifica cómo los LLMs
obtienen contexto, ejecutan acciones e interactúan con varios sistemas.

## ¿Cómo funciona MCP?

MCP sigue una arquitectura cliente-servidor, definiendo cómo los datos (recursos),
plantillas interactivas (prompts) y funciones accionables (herramientas) son
expuestos por un servidor MCP y consumidos por un cliente MCP (que podría ser
una aplicación anfitriona de LLM o un agente de IA).

## Herramientas MCP en ADK

ADK te ayuda tanto a usar como a consumir herramientas MCP en tus agentes, ya sea que estés
intentando construir una herramienta para llamar a un servicio MCP, o exponiendo un servidor MCP para
que otros desarrolladores o agentes interactúen con tus herramientas.

Consulta la [documentación de Herramientas MCP](/adk-docs/tools-custom/mcp-tools/) para ejemplos de código
y patrones de diseño que te ayudan a usar ADK junto con servidores MCP, incluyendo:

- **Usar Servidores MCP Existentes dentro de ADK**: Un agente ADK puede actuar como un cliente MCP
  y usar herramientas proporcionadas por servidores MCP externos.
- **Exponer Herramientas ADK a través de un Servidor MCP**: Cómo construir un servidor MCP que
  envuelve herramientas ADK, haciéndolas accesibles a cualquier cliente MCP.

## MCP Toolbox para Bases de Datos

[MCP Toolbox para Bases de Datos](https://github.com/googleapis/genai-toolbox) es un
servidor MCP de código abierto que expone de manera segura tus fuentes de datos backend como un
conjunto de herramientas preconstruidas y listas para producción para agentes de IA Gen. Funciona como una
capa de abstracción universal, permitiendo que tu agente ADK consulte, analice
y recupere información de manera segura desde una amplia variedad de bases de datos con soporte integrado.

El servidor MCP Toolbox incluye una biblioteca completa de conectores, asegurando que
los agentes puedan interactuar de manera segura con tu complejo patrimonio de datos.

### Fuentes de Datos Soportadas

MCP Toolbox proporciona conjuntos de herramientas listos para usar para las siguientes bases de datos y plataformas de datos:

#### Google Cloud

*   [BigQuery](https://googleapis.github.io/genai-toolbox/resources/sources/bigquery/) (incluyendo herramientas para ejecución SQL, descubrimiento de esquemas y pronóstico de series temporales impulsado por IA)
*   [AlloyDB](https://googleapis.github.io/genai-toolbox/resources/sources/alloydb-pg/) (compatible con PostgreSQL, con herramientas tanto para consultas estándar como consultas en lenguaje natural)
*   [AlloyDB Admin](https://googleapis.github.io/genai-toolbox/resources/sources/alloydb-admin/)
*   [Spanner](https://googleapis.github.io/genai-toolbox/resources/sources/spanner/) (soportando tanto dialectos GoogleSQL como PostgreSQL)
*   Cloud SQL (con soporte dedicado para [Cloud SQL para PostgreSQL](https://googleapis.github.io/genai-toolbox/resources/sources/cloud-sql-pg/), [Cloud SQL para MySQL](https://googleapis.github.io/genai-toolbox/resources/sources/cloud-sql-mysql/) y [Cloud SQL para SQL Server](https://googleapis.github.io/genai-toolbox/resources/sources/cloud-sql-mssql/))
*   [Cloud SQL Admin](https://googleapis.github.io/genai-toolbox/resources/sources/cloud-sql-admin/)
*   [Firestore](https://googleapis.github.io/genai-toolbox/resources/sources/firestore/)
*   [Bigtable](https://googleapis.github.io/genai-toolbox/resources/sources/bigtable/)
*   [Dataplex](https://googleapis.github.io/genai-toolbox/resources/sources/dataplex/) (para descubrimiento de datos y búsqueda de metadatos)
*   [Cloud Monitoring](https://googleapis.github.io/genai-toolbox/resources/sources/cloud-monitoring/)

#### Bases de Datos Relacionales y SQL

*   [PostgreSQL](https://googleapis.github.io/genai-toolbox/resources/sources/postgres/) (genérico)
*   [MySQL](https://googleapis.github.io/genai-toolbox/resources/sources/mysql/) (genérico)
*   [Microsoft SQL Server](https://googleapis.github.io/genai-toolbox/resources/sources/mssql/) (genérico)
*   [ClickHouse](https://googleapis.github.io/genai-toolbox/resources/sources/clickhouse/)
*   [TiDB](https://googleapis.github.io/genai-toolbox/resources/sources/tidb/)
*   [OceanBase](https://googleapis.github.io/genai-toolbox/resources/sources/oceanbase/)
*   [Firebird](https://googleapis.github.io/genai-toolbox/resources/sources/firebird/)
*   [SQLite](https://googleapis.github.io/genai-toolbox/resources/sources/sqlite/)
*   [YugabyteDB](https://googleapis.github.io/genai-toolbox/resources/sources/yugabytedb/)

#### Bases de Datos NoSQL y Almacenes Clave-Valor

*   [MongoDB](https://googleapis.github.io/genai-toolbox/resources/sources/mongodb/)
*   [Couchbase](https://googleapis.github.io/genai-toolbox/resources/sources/couchbase/)
*   [Redis](https://googleapis.github.io/genai-toolbox/resources/sources/redis/)
*   [Valkey](https://googleapis.github.io/genai-toolbox/resources/sources/valkey/)
*   [Cassandra](https://googleapis.github.io/genai-toolbox/resources/sources/cassandra/)

#### Bases de Datos de Grafos

*   [Neo4j](https://googleapis.github.io/genai-toolbox/resources/sources/neo4j/) (con herramientas para consultas Cypher e inspección de esquemas)
*   [Dgraph](https://googleapis.github.io/genai-toolbox/resources/sources/dgraph/)

#### Plataformas de Datos y Federación

*   [Looker](https://googleapis.github.io/genai-toolbox/resources/sources/looker/) (para ejecutar Looks, consultas y construir dashboards a través de la API de Looker)
*   [Trino](https://googleapis.github.io/genai-toolbox/resources/sources/trino/) (para ejecutar consultas federadas a través de múltiples fuentes)

#### Otros

*   [HTTP](https://googleapis.github.io/genai-toolbox/resources/sources/http/)

### Documentación

Consulta la
[documentación de MCP Toolbox para Bases de Datos](/adk-docs/tools/google-cloud/mcp-toolbox-for-databases/)
sobre cómo puedes usar ADK junto con MCP Toolbox para
Bases de Datos. Para comenzar con MCP Toolbox para Bases de Datos, también están disponibles una publicación de blog [Tutorial: MCP Toolbox para Bases de Datos - Exponiendo Conjuntos de Datos de BigQuery](https://medium.com/google-cloud/tutorial-mcp-toolbox-for-databases-exposing-big-query-datasets-9321f0064f4e) y un Codelab [MCP Toolbox para Bases de Datos: Haciendo disponibles conjuntos de datos de BigQuery para clientes MCP](https://codelabs.developers.google.com/mcp-toolbox-bigquery-dataset?hl=en#0).

![GenAI Toolbox](../assets/mcp_db_toolbox.png)

## Agente ADK y servidor FastMCP
[FastMCP](https://github.com/jlowin/fastmcp) maneja todos los detalles complejos del protocolo MCP y la gestión del servidor, para que puedas enfocarte en construir excelentes herramientas. Está diseñado para ser de alto nivel y Pythonic; en la mayoría de los casos, decorar una función es todo lo que necesitas.

Consulta la [documentación de Herramientas MCP](/adk-docs/tools-custom/mcp-tools/) sobre
cómo puedes usar ADK junto con el servidor FastMCP ejecutándose en Cloud Run.

## Servidores MCP para Google Cloud Genmedia

[Herramientas MCP para Servicios Genmedia](https://github.com/GoogleCloudPlatform/vertex-ai-creative-studio/tree/main/experiments/mcp-genmedia)
es un conjunto de servidores MCP de código abierto que te permiten integrar servicios de medios generativos de Google Cloud
—como Imagen, Veo, voces Chirp 3 HD y Lyria—en
tus aplicaciones de IA.

Agent Development Kit (ADK) y [Genkit](https://genkit.dev/) proporcionan soporte integrado
para estas herramientas MCP, permitiendo que tus agentes de IA orquesten efectivamente
flujos de trabajo de medios generativos. Para orientación de implementación, consulta el [agente de ejemplo de ADK](https://github.com/GoogleCloudPlatform/vertex-ai-creative-studio/tree/main/experiments/mcp-genmedia/sample-agents/adk)
y el
[ejemplo de Genkit](https://github.com/GoogleCloudPlatform/vertex-ai-creative-studio/tree/main/experiments/mcp-genmedia/sample-agents/genkit).