---
catalog_title: MCP Toolbox for Databases
catalog_description: Connect over 30 different data sources to your agents
catalog_icon: /adk-docs/assets/tools-mcp-toolbox-for-databases.png
---

# MCP Toolbox para Bases de Datos

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python</span><span class="lst-typescript">Typescript</span><span class="lst-go">Go</span>
</div>

[MCP Toolbox for Databases](https://github.com/googleapis/genai-toolbox) es un
servidor MCP de código abierto para bases de datos. Fue diseñado pensando en calidad
empresarial y lista para producción. Te permite desarrollar herramientas más fácil,
rápido y de forma más segura al manejar las complejidades como agrupación de
conexiones, autenticación y más.

El Kit de Desarrollo de Agentes (ADK) de Google tiene soporte integrado para Toolbox. Para más
información sobre
[cómo empezar](https://googleapis.github.io/genai-toolbox/getting-started/) o
[configurar](https://googleapis.github.io/genai-toolbox/getting-started/configure/)
Toolbox, consulta la
[documentación](https://googleapis.github.io/genai-toolbox/getting-started/introduction/).

![GenAI Toolbox](../../assets/mcp_db_toolbox.png)

## Fuentes de Datos Soportadas

MCP Toolbox proporciona conjuntos de herramientas listos para usar para las siguientes bases de datos y plataformas de datos:

### Google Cloud

*   [BigQuery](https://googleapis.github.io/genai-toolbox/resources/sources/bigquery/) (incluyendo herramientas para ejecución SQL, descubrimiento de esquemas y pronósticos de series temporales impulsados por IA)
*   [AlloyDB](https://googleapis.github.io/genai-toolbox/resources/sources/alloydb-pg/) (compatible con PostgreSQL, con herramientas tanto para consultas estándar como consultas en lenguaje natural)
*   [AlloyDB Admin](https://googleapis.github.io/genai-toolbox/resources/sources/alloydb-admin/)
*   [Spanner](https://googleapis.github.io/genai-toolbox/resources/sources/spanner/) (soporta dialectos tanto GoogleSQL como PostgreSQL)
*   Cloud SQL (con soporte dedicado para [Cloud SQL for PostgreSQL](https://googleapis.github.io/genai-toolbox/resources/sources/cloud-sql-pg/), [Cloud SQL for MySQL](https://googleapis.github.io/genai-toolbox/resources/sources/cloud-sql-mysql/), y [Cloud SQL for SQL Server](https://googleapis.github.io/genai-toolbox/resources/sources/cloud-sql-mssql/))
*   [Cloud SQL Admin](https://googleapis.github.io/genai-toolbox/resources/sources/cloud-sql-admin/)
*   [Firestore](https://googleapis.github.io/genai-toolbox/resources/sources/firestore/)
*   [Bigtable](https://googleapis.github.io/genai-toolbox/resources/sources/bigtable/)
*   [Dataplex](https://googleapis.github.io/genai-toolbox/resources/sources/dataplex/) (para descubrimiento de datos y búsqueda de metadatos)
*   [Cloud Monitoring](https://googleapis.github.io/genai-toolbox/resources/sources/cloud-monitoring/)

### Bases de Datos Relacionales y SQL

*   [PostgreSQL](https://googleapis.github.io/genai-toolbox/resources/sources/postgres/) (genérico)
*   [MySQL](https://googleapis.github.io/genai-toolbox/resources/sources/mysql/) (genérico)
*   [Microsoft SQL Server](https://googleapis.github.io/genai-toolbox/resources/sources/mssql/) (genérico)
*   [ClickHouse](https://googleapis.github.io/genai-toolbox/resources/sources/clickhouse/)
*   [TiDB](https://googleapis.github.io/genai-toolbox/resources/sources/tidb/)
*   [OceanBase](https://googleapis.github.io/genai-toolbox/resources/sources/oceanbase/)
*   [Firebird](https://googleapis.github.io/genai-toolbox/resources/sources/firebird/)
*   [SQLite](https://googleapis.github.io/genai-toolbox/resources/sources/sqlite/)
*   [YugabyteDB](https://googleapis.github.io/genai-toolbox/resources/sources/yugabytedb/)

### Bases de Datos NoSQL y Almacenes Clave-Valor

*   [MongoDB](https://googleapis.github.io/genai-toolbox/resources/sources/mongodb/)
*   [Couchbase](https://googleapis.github.io/genai-toolbox/resources/sources/couchbase/)
*   [Redis](https://googleapis.github.io/genai-toolbox/resources/sources/redis/)
*   [Valkey](https://googleapis.github.io/genai-toolbox/resources/sources/valkey/)
*   [Cassandra](https://googleapis.github.io/genai-toolbox/resources/sources/cassandra/)

### Bases de Datos de Grafos

*   [Neo4j](https://googleapis.github.io/genai-toolbox/resources/sources/neo4j/) (con herramientas para consultas Cypher e inspección de esquemas)
*   [Dgraph](https://googleapis.github.io/genai-toolbox/resources/sources/dgraph/)

### Plataformas de Datos y Federación

*   [Looker](https://googleapis.github.io/genai-toolbox/resources/sources/looker/) (para ejecutar Looks, consultas y construir paneles a través de la API de Looker)
*   [Trino](https://googleapis.github.io/genai-toolbox/resources/sources/trino/) (para ejecutar consultas federadas a través de múltiples fuentes)

### Otros

*   [HTTP](https://googleapis.github.io/genai-toolbox/resources/sources/http/)

## Configurar e implementar

Toolbox es un servidor de código abierto que tú despliegas y gestionas. Para más
instrucciones sobre cómo desplegar y configurar, consulta la documentación oficial de Toolbox:

* [Instalando el Servidor](https://googleapis.github.io/genai-toolbox/getting-started/introduction/#installing-the-server)
* [Configurando Toolbox](https://googleapis.github.io/genai-toolbox/getting-started/configure/)

## Instalar SDK del Cliente para ADK

=== "Python"

    ADK depende del paquete de python `toolbox-adk` para usar Toolbox. Instala el
    paquete antes de empezar:

    ```shell
    pip install google-adk[toolbox]
    ```

    ### Cargando Herramientas de Toolbox

    Una vez que tu servidor Toolbox esté configurado, funcionando, puedes cargar herramientas
    desde tu servidor usando ADK:

    ```python
    from google.adk.agents import Agent
    from google.adk.tools.toolbox_toolset import ToolboxToolset

    toolset = ToolboxToolset(
        server_url="http://127.0.0.1:5000"
    )

    root_agent = Agent(
        ...,
        tools=[toolset] # Proporciona el conjunto de herramientas al Agente
    )
    ```

    ### Autenticación

    El `ToolboxToolset` soporta varias estrategias de autenticación incluyendo Workload Identity (ADC), User Identity (OAuth2) y API Keys. Para documentación completa, consulta la [Guía de Autenticación de Toolbox ADK](https://github.com/googleapis/mcp-toolbox-sdk-python/tree/main/packages/toolbox-adk#authentication).

    **Ejemplo: Workload Identity (ADC)**

    Recomendado para Cloud Run, GKE o desarrollo local con `gcloud auth login`.

    ```python
    from google.adk.tools.toolbox_toolset import ToolboxToolset
    from toolbox_adk import CredentialStrategy

    # target_audience: La URL de tu servidor Toolbox
    creds = CredentialStrategy.workload_identity(target_audience="<TOOLBOX_URL>")

    toolset = ToolboxToolset(
        server_url="<TOOLBOX_URL>",
        credentials=creds
    )
    ```

    ### Configuración Avanzada

    Puedes configurar enlace de parámetros, hooks de solicitud y encabezados adicionales. Consulta la [documentación de Toolbox ADK](https://github.com/googleapis/mcp-toolbox-sdk-python/tree/main/packages/toolbox-adk) para más detalles.

    #### Enlace de Parámetros

    Enlaza valores a parámetros de herramientas globalmente. Estos valores están ocultos del modelo.

    ```python
    toolset = ToolboxToolset(
        server_url="...",
        bound_params={
            "region": "us-central1",
            "api_key": lambda: get_api_key() # Puede ser un callable
        }
    )
    ```

    #### Uso con Hooks

    Adjunta funciones `pre_hook` y `post_hook` para ejecutar lógica antes y después de la invocación de herramientas.

    ```python
    async def log_start(context, args):
        print(f"Starting tool with args: {args}")

    toolset = ToolboxToolset(
        server_url="...",
        pre_hook=log_start
    )
    ```

=== "Typescript"

    ADK depende del paquete TS `@toolbox-sdk/adk` para usar Toolbox. Instala el
    paquete antes de empezar:

    ```shell
    npm install @toolbox-sdk/adk
    ```

    ### Cargando Herramientas de Toolbox

    Una vez que tu servidor Toolbox esté configurado y funcionando, puedes cargar herramientas
    desde tu servidor usando ADK:

    ```typescript
    import {InMemoryRunner, LlmAgent} from '@google/adk';
    import {Content} from '@google/genai';
    import {ToolboxClient} from '@toolbox-sdk/adk'

    const toolboxClient = new ToolboxClient("http://127.0.0.1:5000");
    const loadedTools = await toolboxClient.loadToolset();

    export const rootAgent = new LlmAgent({
      name: 'weather_time_agent',
      model: 'gemini-2.5-flash',
      description:
        'Agente para responder preguntas sobre la hora y el clima en una ciudad.',
      instruction:
        'Eres un agente útil que puede responder preguntas de usuarios sobre la hora y el clima en una ciudad.',
      tools: loadedTools,
    });

    async function main() {
      const userId = 'test_user';
      const appName = rootAgent.name;
      const runner = new InMemoryRunner({agent: rootAgent, appName});
      const session = await runner.sessionService.createSession({
        appName,
        userId,
      });

      const prompt = 'What is the weather in New York? And the time?';
      const content: Content = {
        role: 'user',
        parts: [{text: prompt}],
      };
      console.log(content);
      for await (const e of runner.runAsync({
        userId,
        sessionId: session.id,
        newMessage: content,
      })) {
        if (e.content?.parts?.[0]?.text) {
          console.log(`${e.author}: ${JSON.stringify(e.content, null, 2)}`);
        }
      }
    }

    main().catch(console.error);
    ```

=== "Go"

    ADK depende del módulo go `mcp-toolbox-sdk-go` para usar Toolbox. Instala el
    módulo antes de empezar:

    ```shell
    go get github.com/googleapis/mcp-toolbox-sdk-go
    ```

    ### Cargando Herramientas de Toolbox

    Una vez que tu servidor Toolbox esté configurado y funcionando, puedes cargar herramientas
    desde tu servidor usando ADK:

    ```go
    package main

    import (
    	"context"
    	"fmt"

    	"github.com/googleapis/mcp-toolbox-sdk-go/tbadk"
    	"google.golang.org/adk/agent/llmagent"
    )

    func main() {

      toolboxClient, err := tbadk.NewToolboxClient("https://127.0.0.1:5000")
    	if err != nil {
    		log.Fatalf("Failed to create MCP Toolbox client: %v", err)
    	}

      // Carga un conjunto específico de herramientas
      toolboxtools, err := toolboxClient.LoadToolset("my-toolset-name", ctx)
      if err != nil {
        return fmt.Sprintln("Could not load Toolbox Toolset", err)
      }

      toolsList := make([]tool.Tool, len(toolboxtools))
        for i := range toolboxtools {
          toolsList[i] = &toolboxtools[i]
        }

      llmagent, err := llmagent.New(llmagent.Config{
        ...,
        Tools:       toolsList,
      })

      // Carga una sola herramienta
      tool, err := client.LoadTool("my-tool-name", ctx)
      if err != nil {
        return fmt.Sprintln("Could not load Toolbox Tool", err)
      }

      llmagent, err := llmagent.New(llmagent.Config{
        ...,
        Tools:       []tool.Tool{&toolboxtool},
      })
    }
    ```

## Características Avanzadas de Toolbox

Toolbox tiene una variedad de características para facilitar el desarrollo de herramientas Gen AI para bases de datos.
Para más información, lee más sobre las siguientes características:

* [Parámetros Autenticados](https://googleapis.github.io/genai-toolbox/resources/tools/#authenticated-parameters): enlaza entradas de herramientas a valores de tokens OIDC automáticamente, facilitando la ejecución de consultas sensibles sin potencialmente filtrar datos
* [Invocaciones Autorizadas:](https://googleapis.github.io/genai-toolbox/resources/tools/#authorized-invocations) restringe el acceso para usar una herramienta basándose en el token de autenticación del usuario
* [OpenTelemetry](https://googleapis.github.io/genai-toolbox/how-to/export_telemetry/): obtén métricas y rastreo de Toolbox con OpenTelemetry