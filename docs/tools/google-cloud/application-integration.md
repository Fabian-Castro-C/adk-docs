---
catalog_title: Application Integration
catalog_description: Link your agents to enterprise apps using Integration Connectors
catalog_icon: /adk-docs/assets/tools-apigee-integration.png
---

# Herramientas de Application Integration para ADK

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-java">Java v0.3.0</span>
</div>

Con **ApplicationIntegrationToolset**, puedes otorgar de manera transparente a tus agentes
acceso seguro y gobernado a aplicaciones empresariales utilizando los más de 100 conectores
preconstruidos de Integration Connectors para sistemas como Salesforce, ServiceNow,
JIRA, SAP y más.

Es compatible tanto con aplicaciones locales como SaaS. Además, puedes convertir
tus automatizaciones de procesos existentes de Application Integration en flujos de trabajo
agénticos proporcionando flujos de trabajo de integración de aplicaciones como herramientas para tus agentes ADK.

La búsqueda federada dentro de Application Integration te permite usar agentes ADK para consultar
múltiples aplicaciones empresariales y fuentes de datos simultáneamente.

[:fontawesome-brands-youtube:{.youtube-red-icon} Mira cómo funciona ADK Federated Search en Application Integration en este video tutorial](https://www.youtube.com/watch?v=JdlWOQe5RgU){: target="_blank" rel="noopener noreferrer"}

<iframe width="560" height="315" src="https://www.youtube.com/embed/JdlWOQe5RgU?si=bFY_-jJ6Oliy5UMG" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Prerrequisitos

### 1. Instalar ADK

Instala Agent Development Kit siguiendo los pasos en la
[guía de instalación](/adk-docs/get-started/installation/).

### 2. Instalar CLI

Instala la
[Google Cloud CLI](https://cloud.google.com/sdk/docs/install#installation_instructions).
Para usar la herramienta con credenciales predeterminadas, ejecuta los siguientes comandos:

```shell
gcloud config set project <project-id>
gcloud auth application-default login
gcloud auth application-default set-quota-project <project-id>
```

Reemplaza `<project-id>` con el ID único de tu proyecto de Google Cloud.

### 3. Aprovisionar el flujo de trabajo de Application Integration y publicar Connection Tool

Usa un
[flujo de trabajo de Application Integration](https://cloud.google.com/application-integration/docs/overview)
existente o una conexión de
[Integrations Connector](https://cloud.google.com/integration-connectors/docs/overview)
que desees usar con tu agente. También puedes crear un nuevo
[flujo de trabajo de Application Integration](https://cloud.google.com/application-integration/docs/setup-application-integration)
o una
[conexión](https://cloud.google.com/integration-connectors/docs/connectors/neo4j/configure#configure-the-connector).

Importa y publica la
[Connection Tool](https://console.cloud.google.com/integrations/templates/connection-tool/locations/global)
desde la biblioteca de plantillas.

**Nota**: Para usar un conector de Integration Connectors, necesitas aprovisionar
Application Integration en la misma región que tu conexión.

### 4. Crear estructura del proyecto

=== "Python"

    Configura la estructura de tu proyecto y crea los archivos requeridos:

      ```console
      project_root_folder
      ├── .env
      └── my_agent
          ├── __init__.py
          ├── agent.py
          └── tools.py
      ```

    Al ejecutar el agente, asegúrate de ejecutar `adk web` desde la `project_root_folder`.

=== "Java"

    Configura la estructura de tu proyecto y crea los archivos requeridos:

      ```console
        project_root_folder
        └── my_agent
            ├── agent.java
            └── pom.xml
      ```

     Al ejecutar el agente, asegúrate de ejecutar los comandos desde la `project_root_folder`.

### 5. Establecer roles y permisos

Para obtener los permisos que necesitas configurar para
**ApplicationIntegrationToolset**, debes tener los siguientes roles de IAM en el
proyecto (comunes tanto para Integration Connectors como para flujos de trabajo de Application Integration):

    - roles/integrations.integrationEditor
    - roles/connectors.invoker
    - roles/secretmanager.secretAccessor

**Nota:** Al usar Agent Engine (AE) para el despliegue, no uses
`roles/integrations.integrationInvoker`, ya que puede resultar en errores 403. Usa
`roles/integrations.integrationEditor` en su lugar.

## Usar Integration Connectors

Conecta tu agente a aplicaciones empresariales usando
[Integration Connectors](https://cloud.google.com/integration-connectors/docs/overview).

### Antes de comenzar

**Nota:** La integración *ExecuteConnection* generalmente se crea automáticamente cuando aprovisionas Application Integration en una región determinada. Si *ExecuteConnection* no existe en la [lista de integraciones](https://console.cloud.google.com/integrations/list), debes seguir estos pasos para crearla:

1. Para usar un conector de Integration Connectors, haz clic en **QUICK SETUP** y [aprovisiona](https://console.cloud.google.com/integrations)
   Application Integration en la misma región que tu conexión.

   ![Google Cloud Tools](/adk-docs/assets/application-integration-overview.png)



2. Ve a la plantilla de [Connection Tool](https://console.cloud.google.com/integrations/templates/connection-tool/locations/us-central1)
   en la biblioteca de plantillas y haz clic en **USE TEMPLATE**.


    ![Google Cloud Tools](/adk-docs/assets/use-connection-tool-template.png)

3. Ingresa el nombre de integración como *ExecuteConnection* (es obligatorio usar exactamente este nombre de integración).
   Luego, selecciona la región para que coincida con tu región de conexión y haz clic en **CREATE**.

4. Haz clic en **PUBLISH** para publicar la integración en el editor de <i>Application Integration</i>.


    ![Google Cloud Tools](/adk-docs/assets/publish-integration.png)


### Crear un Toolset de Application Integration

Para crear un Toolset de Application Integration para Integration Connectors, sigue estos pasos:

1.  Crea una herramienta con `ApplicationIntegrationToolset` en el archivo `tools.py`:

    ```py
    from google.adk.tools.application_integration_tool.application_integration_toolset import ApplicationIntegrationToolset

    connector_tool = ApplicationIntegrationToolset(
        project="test-project", # TODO: reemplazar con el proyecto GCP de la conexión
        location="us-central1", #TODO: reemplazar con la ubicación de la conexión
        connection="test-connection", #TODO: reemplazar con el nombre de la conexión
        entity_operations={"Entity_One": ["LIST","CREATE"], "Entity_Two": []},#lista vacía para acciones significa que todas las operaciones en la entidad están soportadas.
        actions=["action1"], #TODO: reemplazar con las acciones
        service_account_json='{...}', # opcional. Json convertido en cadena para la clave de la cuenta de servicio
        tool_name_prefix="tool_prefix2",
        tool_instructions="..."
    )
    ```

    **Nota:**

    * Puedes proporcionar una cuenta de servicio para usar en lugar de las credenciales predeterminadas generando una [Clave de Cuenta de Servicio](https://cloud.google.com/iam/docs/keys-create-delete#creating), y proporcionando los [roles de IAM de Application Integration e Integration Connector](#prerequisites) correctos a la cuenta de servicio.
    * Para encontrar la lista de entidades y acciones soportadas para una conexión, usa las APIs de Connectors: [listActions](https://cloud.google.com/integration-connectors/docs/reference/rest/v1/projects.locations.connections.connectionSchemaMetadata/listActions) o [listEntityTypes](https://cloud.google.com/integration-connectors/docs/reference/rest/v1/projects.locations.connections.connectionSchemaMetadata/listEntityTypes).


    `ApplicationIntegrationToolset` soporta `auth_scheme` y `auth_credential` para **autenticación OAuth2 dinámica** para Integration Connectors. Para usarlo, crea una herramienta similar a esta en el archivo `tools.py`:

    ```py
    from google.adk.tools.application_integration_tool.application_integration_toolset import ApplicationIntegrationToolset
    from google.adk.tools.openapi_tool.auth.auth_helpers import dict_to_auth_scheme
    from google.adk.auth import AuthCredential
    from google.adk.auth import AuthCredentialTypes
    from google.adk.auth import OAuth2Auth

    oauth2_data_google_cloud = {
      "type": "oauth2",
      "flows": {
          "authorizationCode": {
              "authorizationUrl": "https://accounts.google.com/o/oauth2/auth",
              "tokenUrl": "https://oauth2.googleapis.com/token",
              "scopes": {
                  "https://www.googleapis.com/auth/cloud-platform": (
                      "View and manage your data across Google Cloud Platform"
                      " services"
                  ),
                  "https://www.googleapis.com/auth/calendar.readonly": "View your calendars"
              },
          }
      },
    }

    oauth_scheme = dict_to_auth_scheme(oauth2_data_google_cloud)

    auth_credential = AuthCredential(
      auth_type=AuthCredentialTypes.OAUTH2,
      oauth2=OAuth2Auth(
          client_id="...", #TODO: reemplazar con client_id
          client_secret="...", #TODO: reemplazar con client_secret
      ),
    )

    connector_tool = ApplicationIntegrationToolset(
        project="test-project", # TODO: reemplazar con el proyecto GCP de la conexión
        location="us-central1", #TODO: reemplazar con la ubicación de la conexión
        connection="test-connection", #TODO: reemplazar con el nombre de la conexión
        entity_operations={"Entity_One": ["LIST","CREATE"], "Entity_Two": []},#lista vacía para acciones significa que todas las operaciones en la entidad están soportadas.
        actions=["GET_calendars/%7BcalendarId%7D/events"], #TODO: reemplazar con acciones. este es para listar eventos
        service_account_json='{...}', # opcional. Json convertido en cadena para la clave de la cuenta de servicio
        tool_name_prefix="tool_prefix2",
        tool_instructions="...",
        auth_scheme=oauth_scheme,
        auth_credential=auth_credential
    )
    ```


2. Actualiza el archivo `agent.py` y agrega la herramienta a tu agente:

    ```py
    from google.adk.agents.llm_agent import LlmAgent
    from .tools import connector_tool

    root_agent = LlmAgent(
        model='gemini-2.0-flash',
        name='connector_agent',
        instruction="Help user, leverage the tools you have access to",
        tools=[connector_tool],
    )
    ```

3. Configura `__init__.py` para exponer tu agente:

    ```py
    from . import agent
    ```

4. Inicia la interfaz web de Google ADK y usa tu agente:

    ```shell
    # asegúrate de ejecutar `adk web` desde tu project_root_folder
    adk web
    ```

Después de completar los pasos anteriores, ve a [http://localhost:8000](http://localhost:8000), y elige
   el agente `my\_agent` (que es el mismo que el nombre de la carpeta del agente).


## Usar flujos de trabajo de Application Integration

Usa un
[flujo de trabajo de Application Integration](https://cloud.google.com/application-integration/docs/overview)
existente como una herramienta para tu agente o crea uno nuevo.


### 1. Crear una herramienta

=== "Python"

    Para crear una herramienta con `ApplicationIntegrationToolset` en el archivo `tools.py`, usa el siguiente código:

      ```py
          integration_tool = ApplicationIntegrationToolset(
              project="test-project", # TODO: reemplazar con el proyecto GCP de la conexión
              location="us-central1", #TODO: reemplazar con la ubicación de la conexión
              integration="test-integration", #TODO: reemplazar con el nombre de la integración
              triggers=["api_trigger/test_trigger"],#TODO: reemplazar con el/los ID(s) de trigger. Lista vacía significaría que todos los api triggers en la integración se consideran.
              service_account_json='{...}', #opcional. Json convertido en cadena para la clave de la cuenta de servicio
              tool_name_prefix="tool_prefix1",
              tool_instructions="..."
          )
      ```

      **Nota:** Puedes proporcionar una cuenta de servicio para usar en lugar de las credenciales predeterminadas. Para hacer esto, genera una [Clave de Cuenta de Servicio](https://cloud.google.com/iam/docs/keys-create-delete#creating) y proporciona los
         [roles de IAM de Application Integration e Integration Connector](#prerequisites) correctos a la cuenta de servicio. Para más detalles sobre los roles de IAM, consulta la sección [Prerrequisitos](#prerequisites).

=== "Java"

    Para crear una herramienta con `ApplicationIntegrationToolset` en el archivo `tools.java`, usa el siguiente código:

      ```java
          import com.google.adk.tools.applicationintegrationtoolset.ApplicationIntegrationToolset;
          import com.google.common.collect.ImmutableList;
          import com.google.common.collect.ImmutableMap;

          public class Tools {
              private static ApplicationIntegrationToolset integrationTool;
              private static ApplicationIntegrationToolset connectionsTool;

              static {
                  integrationTool = new ApplicationIntegrationToolset(
                          "test-project",
                          "us-central1",
                          "test-integration",
                          ImmutableList.of("api_trigger/test-api"),
                          null,
                          null,
                          null,
                          "{...}",
                          "tool_prefix1",
                          "...");

                  connectionsTool = new ApplicationIntegrationToolset(
                          "test-project",
                          "us-central1",
                          null,
                          null,
                          "test-connection",
                          ImmutableMap.of("Issue", ImmutableList.of("GET")),
                          ImmutableList.of("ExecuteCustomQuery"),
                          "{...}",
                          "tool_prefix",
                          "...");
              }
          }
      ```

      **Nota:** Puedes proporcionar una cuenta de servicio para usar en lugar de las credenciales predeterminadas. Para hacer esto, genera una [Clave de Cuenta de Servicio](https://cloud.google.com/iam/docs/keys-create-delete#creating) y proporciona los [roles de IAM de Application Integration e Integration Connector](#prerequisites) correctos a la cuenta de servicio. Para más detalles sobre los roles de IAM, consulta la sección [Prerrequisitos](#prerequisites).

### 2. Agregar la herramienta a tu agente

=== "Python"

    Para actualizar el archivo `agent.py` y agregar la herramienta a tu agente, usa el siguiente código:

      ```py
          from google.adk.agents.llm_agent import LlmAgent
          from .tools import integration_tool, connector_tool

          root_agent = LlmAgent(
              model='gemini-2.0-flash',
              name='integration_agent',
              instruction="Help user, leverage the tools you have access to",
              tools=[integration_tool],
          )
      ```

=== "Java"

    Para actualizar el archivo `agent.java` y agregar la herramienta a tu agente, usa el siguiente código:

      ```java
          import com.google.adk.agent.LlmAgent;
          import com.google.adk.tools.BaseTool;
          import com.google.common.collect.ImmutableList;

            public class MyAgent {
                public static void main(String[] args) {
                    // Asumiendo que la clase Tools está definida como en el paso anterior
                    ImmutableList<BaseTool> tools = ImmutableList.<BaseTool>builder()
                            .add(Tools.integrationTool)
                            .add(Tools.connectionsTool)
                            .build();

                    // Finalmente, crea tu agente con las herramientas generadas automáticamente.
                    LlmAgent rootAgent = LlmAgent.builder()
                            .name("science-teacher")
                            .description("Science teacher agent")
                            .model("gemini-2.0-flash")
                            .instruction(
                                    "Help user, leverage the tools you have access to."
                            )
                            .tools(tools)
                            .build();

                    // Ahora puedes usar rootAgent para interactuar con el LLM
                    // Por ejemplo, puedes iniciar una conversación con el agente.
                }
            }
        ```

**Nota:** Para encontrar la lista de entidades y acciones soportadas para una
        conexión, usa estas APIs de Connector: `listActions`, `listEntityTypes`.

### 3. Exponer tu agente

=== "Python"

    Para configurar `__init__.py` para exponer tu agente, usa el siguiente código:

      ```py
          from . import agent
      ```

### 4. Usar tu agente

=== "Python"

    Para iniciar la interfaz web de Google ADK y usar tu agente, usa los siguientes comandos:

      ```shell
          # asegúrate de ejecutar `adk web` desde tu project_root_folder
          adk web
      ```
    Después de completar los pasos anteriores, ve a [http://localhost:8000](http://localhost:8000), y elige el agente `my_agent` (que es el mismo que el nombre de la carpeta del agente).

=== "Java"

    Para iniciar la interfaz web de Google ADK y usar tu agente, usa los siguientes comandos:

      ```bash
          mvn install

          mvn exec:java \
              -Dexec.mainClass="com.google.adk.web.AdkWebServer" \
              -Dexec.args="--adk.agents.source-dir=src/main/java" \
              -Dexec.classpathScope="compile"
      ```

    Después de completar los pasos anteriores, ve a [http://localhost:8000](http://localhost:8000), y elige el agente `my_agent` (que es el mismo que el nombre de la carpeta del agente).