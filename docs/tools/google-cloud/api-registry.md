---
catalog_title: API Registry
catalog_description: Dynamically connect with Google Cloud services as MCP tools
catalog_icon: /assets/developer-tools-color.svg
---

# Conectar herramientas MCP desde Cloud API Registry

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v1.20.0</span><span class="lst-preview">Preview</span>
</div>

La herramienta de conector de Google Cloud API Registry para Agent Development Kit (ADK)
te permite acceder a una amplia gama de servicios de Google Cloud para tus agentes como
servidores de Model Context Protocol (MCP) a través del
[Google Cloud API Registry](https://docs.cloud.google.com/api-registry/docs/overview).
Puedes configurar esta herramienta para conectar tu agente a tus proyectos de Google Cloud
y acceder dinámicamente a los servicios de Cloud habilitados para ese proyecto.

!!! example "Preview release"
    La función de Google Cloud API Registry es una versión Preview. Para
    más información, consulta las
    [descripciones de etapas de lanzamiento](https://cloud.google.com/products#product-launch-stages).

## Prerrequisitos

Antes de usar el API Registry con tu agente, necesitas asegurar lo siguiente:

-   **Proyecto de Google Cloud:** Configura tu agente para acceder a modelos de IA usando un
    proyecto existente de Google Cloud.

-   **Acceso a API Registry:** El entorno donde se ejecuta tu agente necesita
    [Application Default Credentials](https://docs.cloud.google.com/docs/authentication/provide-credentials-adc)
    de Google Cloud con el rol `apiregistry.viewer` para listar los servidores MCP disponibles.

-   **APIs de Cloud:** En tu proyecto de Google Cloud, habilita las
    APIs de Google Cloud *cloudapiregistry.googleapis.com* y *apihub.googleapis.com*.

-   **Acceso a MCP Server y herramientas:** Asegúrate de habilitar los MCP Servers en el
    API Registry para los servicios de Google Cloud en tu proyecto de Cloud a los que
    quieres acceder con tu agente. Puedes habilitar esto en Cloud Console o
    usar un comando gcloud como:
    `gcloud beta api-registry mcp enable bigquery.googleapis.com --project={PROJECT_ID}`.
    Las credenciales usadas por el agente deben tener permisos para acceder al servidor MCP
    y a los servicios subyacentes usados por las herramientas. Por ejemplo, para usar
    herramientas de BigQuery, la cuenta de servicio necesita roles de IAM de BigQuery como
    `bigquery.dataViewer` y `bigquery.jobUser`. Para más información sobre
    permisos requeridos, consulta [Autenticación y acceso](#auth).

Puedes verificar qué servidores MCP están habilitados con API Registry usando el siguiente
comando gcloud:

```console
gcloud beta api-registry mcp servers list --project={PROJECT_ID}.
```

## Usar con agente

Al configurar la herramienta de conector de API Registry con un agente, primero
inicializas la clase ***ApiRegistry*** para establecer una conexión con los servicios de Cloud,
y luego usas la función `get_toolset()` para recuperar un conjunto de herramientas para un
servidor MCP específico registrado en el API Registry. El siguiente ejemplo de código
demuestra cómo crear un agente que usa herramientas de un servidor MCP listado en
API Registry. Este agente está diseñado para interactuar con BigQuery:

```python
import os
from google.adk.agents.llm_agent import LlmAgent
from google.adk.tools.api_registry import ApiRegistry

# Configura con tu ID de proyecto de Google Cloud y nombre del servidor MCP registrado
PROJECT_ID = "your-google-cloud-project-id"
MCP_SERVER_NAME = "projects/your-google-cloud-project-id/locations/global/mcpServers/your-mcp-server-name"

# Proveedor de encabezado de ejemplo para BigQuery, se requiere un encabezado de proyecto.
def header_provider(context):
    return {"x-goog-user-project": PROJECT_ID}

# Inicializar ApiRegistry
api_registry = ApiRegistry(
    api_registry_project_id=PROJECT_ID,
    header_provider=header_provider
)

# Obtener el conjunto de herramientas para el servidor MCP específico
registry_tools = api_registry.get_toolset(
    mcp_server_name=MCP_SERVER_NAME,
    # Filtrar herramientas opcionalmente:
    #tool_filter=["list_datasets", "run_query"]
)

# Crear un agente con las herramientas
root_agent = LlmAgent(
    model="gemini-1.5-flash", # O tu modelo preferido
    name="bigquery_assistant",
    instruction="""
Help user access their BigQuery data using the available tools.
    """,
    tools=[registry_tools],
)
```

Para el código completo de este ejemplo, consulta la
muestra [api_registry_agent](https://github.com/google/adk-python/tree/main/contributing/samples/api_registry_agent/).
Para información sobre las opciones de configuración, consulta
[Configuración](#configuration).
Para información sobre la autenticación para esta herramienta, consulta
[Autenticación y acceso](#auth).

## Autenticación y acceso {#auth}

Usar el API Registry con tu agente requiere autenticación para los servicios
a los que el agente accede. Por defecto, la herramienta usa
[Application Default Credentials](https://docs.cloud.google.com/docs/authentication/provide-credentials-adc)
de Google Cloud para autenticación. Al usar esta herramienta, asegúrate de que tu agente tenga los siguientes
permisos y acceso:

-   **Acceso a API Registry:** La clase `ApiRegistry` usa Application Default
    Credentials (`google.auth.default()`) para autenticar solicitudes al
    API Registry de Google Cloud para listar los servidores MCP disponibles. Asegúrate de que el entorno
    donde se ejecuta el agente tenga credenciales con los permisos necesarios para ver
    los recursos del API Registry, como `apiregistry.viewer`.

-   **Acceso a MCP Server y herramientas:** El `McpToolset` devuelto por `get_toolset`
    también usa las Application Default Credentials de Google Cloud por defecto para
    autenticar llamadas al endpoint real del servidor MCP. Las credenciales usadas
    deben tener los permisos necesarios para ambos:
    1.  Acceder al servidor MCP en sí.
    1.  Utilizar los servicios y recursos subyacentes con los que las herramientas interactúan.

-   **Rol de usuario de herramientas MCP:** Permite que la cuenta usada por tu agente llame a
    herramientas MCP a través del API Registry otorgando el rol de usuario de herramientas MCP:
    `gcloud projects add-iam-policy-binding {PROJECT_ID} --member={member}
    --role="roles/mcp.toolUser"`

Por ejemplo, al usar herramientas de servidor MCP que interactúan con BigQuery, la
cuenta asociada con las credenciales, como una cuenta de servicio, debe tener
roles de IAM de BigQuery apropiados, como `bigquery.dataViewer` o
`bigquery.jobUser`, dentro de tu proyecto de Google Cloud para acceder a conjuntos de datos y ejecutar
consultas. En el caso del servidor MCP de bigquery, se requiere un encabezado
`"x-goog-user-project": PROJECT_ID` para usar sus herramientas. Los encabezados adicionales para
autenticación o contexto de proyecto pueden inyectarse a través del argumento `header_provider`
en el constructor de `ApiRegistry`.

## Configuración {#configuration}

El objeto ***APIRegistry*** tiene las siguientes opciones de configuración:

-   **`api_registry_project_id`** (str): El ID de proyecto de Google Cloud donde se
    encuentra el API Registry.

-   **`location`** (str, opcional): La ubicación de los recursos del API Registry.
    Por defecto es `"global"`.

-   **`header_provider`** (Callable, opcional): Una función que toma el contexto de
    llamada y devuelve un diccionario de encabezados HTTP adicionales para enviar con
    solicitudes al servidor MCP. Esto a menudo se usa para autenticación dinámica o
    encabezados específicos del proyecto.

La función `get_toolset()` tiene las siguientes opciones de configuración:

-   **`mcp_server_name`** (str): El nombre completo del servidor MCP registrado desde
    el cual cargar herramientas, por ejemplo:
    `projects/my-project/locations/global/mcpServers/my-server`.

-   **`tool_filter`** (Union[ToolPredicate, List[str]], opcional): Especifica
    qué herramientas incluir en el conjunto de herramientas.
    -   Si es una lista de strings, solo se incluyen las herramientas con nombres en la lista.
    -   Si es una función `ToolPredicate`, la función se llama para cada herramienta, y
        solo se incluyen las herramientas para las cuales devuelve `True`.
    -   Si es `None`, se incluyen todas las herramientas del servidor MCP.

-   **`tool_name_prefix`** (str, opcional): Un prefijo para agregar al nombre de cada
    herramienta en el conjunto de herramientas resultante.

## Recursos adicionales

-   Muestra de código ADK [api_registry_agent](https://github.com/google/adk-python/tree/main/contributing/samples/api_registry_agent/)
-   Documentación de [Google Cloud API Registry](https://docs.cloud.google.com/api-registry/docs/overview)