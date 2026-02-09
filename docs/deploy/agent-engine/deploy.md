# Desplegar en Vertex AI Agent Engine

<div class="language-support-tag" title="Vertex AI Agent Engine currently supports only Python.">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span>
</div>

Este procedimiento de despliegue describe cómo realizar un despliegue estándar de
código de agente ADK en Google Cloud
[Agent Engine](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/overview).
Debes seguir esta ruta de despliegue si tienes un proyecto de Google Cloud
existente y si deseas gestionar cuidadosamente el despliegue de un agente ADK en el
entorno de ejecución de Agent Engine. Estas instrucciones utilizan Cloud Console, la interfaz de
línea de comandos gcloud y la interfaz de línea de comandos ADK (ADK CLI). Esta ruta
se recomienda para usuarios que ya están familiarizados con la configuración de proyectos de Google Cloud
y usuarios que se preparan para despliegues de producción.

Estas instrucciones describen cómo desplegar un proyecto ADK en el entorno de ejecución de Google Cloud Agent
Engine, que incluye las siguientes etapas:

*   [Configurar proyecto de Google Cloud](#setup-cloud-project)
*   [Preparar carpeta del proyecto del agente](#define-your-agent)
*   [Desplegar el agente](#deploy-agent)

## Configurar proyecto de Google Cloud {#setup-cloud-project}

Para desplegar tu agente en Agent Engine, necesitas un proyecto de Google Cloud:

1. **Iniciar sesión en Google Cloud**:
    * Si eres un **usuario existente** de Google Cloud:
        * Inicia sesión a través de
          [https://console.cloud.google.com](https://console.cloud.google.com)
        * Si anteriormente usaste una Prueba Gratuita que ha expirado, es posible que necesites
          actualizar a una
          [cuenta de facturación de Pago](https://docs.cloud.google.com/free/docs/free-cloud-features#how-to-upgrade).
    * Si eres un **nuevo usuario** de Google Cloud:
        * Puedes registrarte en el
          [programa de Prueba Gratuita](https://docs.cloud.google.com/free/docs/free-cloud-features).
          La Prueba Gratuita te otorga un crédito de bienvenida de $300 para gastar durante 91 días en varios
          [productos de Google Cloud](https://docs.cloud.google.com/free/docs/free-cloud-features#during-free-trial)
          y no se te facturará. Durante la Prueba Gratuita, también obtienes acceso al
          [Nivel Gratuito de Google Cloud](https://docs.cloud.google.com/free/docs/free-cloud-features#free-tier),
          que te brinda uso gratuito de productos seleccionados hasta límites mensuales
          especificados, y a pruebas gratuitas específicas de productos.

2. **Crear un proyecto de Google Cloud**
    * Si ya tienes un proyecto de Google Cloud existente, puedes usarlo, pero
      ten en cuenta que este proceso probablemente agregará nuevos servicios al proyecto.
    * Si deseas crear un nuevo proyecto de Google Cloud, puedes crear uno nuevo
      en la página [Crear Proyecto](https://console.cloud.google.com/projectcreate).

3. **Obtener tu ID de Proyecto de Google Cloud**
    * Necesitas tu ID de Proyecto de Google Cloud, que puedes encontrar en tu página de inicio de GCP.
      Asegúrate de anotar el ID del Proyecto (alfanumérico con guiones),
      _no_ el número de proyecto (numérico).

    <img src="/adk-docs/assets/project-id.png" alt="Google Cloud Project ID">

4. **Habilitar Vertex AI en tu proyecto**
    * Para usar Agent Engine, necesitas [habilitar la API de Vertex AI](https://console.cloud.google.com/apis/library/aiplatform.googleapis.com). Haz clic en el botón "Enable" para habilitar la API. Una vez habilitada,
    debería decir "API Enabled".

5. **Habilitar la API de Cloud Resource Manager en tu proyecto**
    * Para usar Agent Engine, necesitas [habilitar la API de Cloud Resource Manager](https://console.developers.google.com/apis/api/cloudresourcemanager.googleapis.com/overview). Haz clic en el botón "Enable" para habilitar la API. Una vez habilitada, debería decir "API Enabled".

## Configurar tu entorno de codificación {#prerequisites-coding-env}

Ahora que has preparado tu proyecto de Google Cloud, puedes regresar a tu entorno de
codificación. Estos pasos requieren acceso a una terminal dentro de tu entorno de
codificación para ejecutar instrucciones de línea de comandos.

### Autenticar tu entorno de codificación con Google Cloud

*   Necesitas autenticar tu entorno de codificación para que tú y tu
    código puedan interactuar con Google Cloud. Para hacerlo, necesitas la CLI de gcloud.
    Si nunca has usado la CLI de gcloud, primero necesitas
    [descargarla e instalarla](https://docs.cloud.google.com/sdk/docs/install-sdk)
    antes de continuar con los pasos a continuación:

*   Ejecuta el siguiente comando en tu terminal para acceder a tu proyecto de Google Cloud
    como usuario:

    ```shell
    gcloud auth login
    ```

    Después de autenticarte, deberías ver el mensaje
    `You are now authenticated with the gcloud CLI!`.

*   Ejecuta el siguiente comando para autenticar tu código para que pueda trabajar con
    Google Cloud:

    ```shell
    gcloud auth application-default login
    ```

    Después de autenticarte, deberías ver el mensaje
    `You are now authenticated with the gcloud CLI!`.

*   (Opcional) Si necesitas establecer o cambiar tu proyecto predeterminado en gcloud, puedes
    usar:

    ```shell
    gcloud config set project MY-PROJECT-ID
    ```

### Definir tu agente {#define-your-agent}

Con tu proyecto de Google Cloud y entorno de codificación preparados, estás listo para desplegar
tu agente. Las instrucciones asumen que tienes una carpeta de proyecto de agente,
tal como:

```shell
multi_tool_agent/
├── .env
├── __init__.py
└── agent.py
```

Para más detalles sobre los archivos y formato del proyecto, consulta el
ejemplo de código
[multi_tool_agent](https://github.com/google/adk-docs/tree/main/examples/python/snippets/get-started/multi_tool_agent).

## Desplegar el agente {#deploy-agent}

Puedes desplegar desde tu terminal usando la herramienta de línea de comandos `adk deploy`. Este
proceso empaqueta tu código, lo construye en un contenedor y lo despliega en el
servicio administrado de Agent Engine. Este proceso puede tomar varios minutos.

El siguiente comando de despliegue de ejemplo usa el código de muestra `multi_tool_agent` como
el proyecto a desplegar:

```shell
PROJECT_ID=my-project-id
LOCATION_ID=us-central1

adk deploy agent_engine \
        --project=$PROJECT_ID \
        --region=$LOCATION_ID \
        --display_name="My First Agent" \
        multi_tool_agent
```

Para `region`, puedes encontrar una lista de las regiones compatibles en la
[página de ubicaciones de Vertex AI Agent Builder](https://docs.cloud.google.com/agent-builder/locations#supported-regions-agent-engine).
Para obtener información sobre las opciones de CLI para el comando `adk deploy agent_engine`, consulta la
[Referencia de CLI de ADK](https://google.github.io/adk-docs/api-reference/cli/cli.html#adk-deploy-agent-engine).

### Salida del comando de despliegue

Una vez desplegado exitosamente, deberías ver la siguiente salida:

```shell
Creating AgentEngine
Create AgentEngine backing LRO: projects/123456789/locations/us-central1/reasoningEngines/751619551677906944/operations/2356952072064073728
View progress and logs at https://console.cloud.google.com/logs/query?project=hopeful-sunset-478017-q0
AgentEngine created. Resource name: projects/123456789/locations/us-central1/reasoningEngines/751619551677906944
To use this AgentEngine in another session:
agent_engine = vertexai.agent_engines.get('projects/123456789/locations/us-central1/reasoningEngines/751619551677906944')
Cleaning up the temp folder: /var/folders/k5/pv70z5m92s30k0n7hfkxszfr00mz24/T/agent_engine_deploy_src/20251219_134245
```

Ten en cuenta que ahora tienes un `RESOURCE_ID` donde tu agente ha sido desplegado (que
en el ejemplo anterior es `751619551677906944`). Necesitas este número de ID junto
con los otros valores para usar tu agente en Agent Engine.

## Usar un agente en Agent Engine

Una vez que hayas completado el despliegue de tu proyecto ADK, puedes consultar el agente
usando el SDK de Vertex AI, la biblioteca de solicitudes de Python o un cliente de API REST. Esta
sección proporciona información sobre lo que necesitas para interactuar con tu agente
y cómo construir URLs para interactuar con la API REST de tu agente.

Para interactuar con tu agente en Agent Engine, necesitas lo siguiente:

*   **PROJECT_ID** (ejemplo: "my-project-id") que puedes encontrar en tu
    [página de detalles del proyecto](https://console.cloud.google.com/iam-admin/settings)
*   **LOCATION_ID** (ejemplo: "us-central1"), que usaste para desplegar tu agente
*   **RESOURCE_ID** (ejemplo: "751619551677906944"), que puedes encontrar en la
    [IU de Agent Engine](https://console.cloud.google.com/vertex-ai/agents/agent-engines)

La estructura de URL de consulta es la siguiente:

```shell
https://$(LOCATION_ID)-aiplatform.googleapis.com/v1/projects/$(PROJECT_ID)/locations/$(LOCATION_ID)/reasoningEngines/$(RESOURCE_ID):query
```

Puedes hacer solicitudes desde tu agente usando esta estructura de URL. Para más información
sobre cómo hacer solicitudes, consulta las instrucciones en la documentación de Agent Engine
[Usar un agente del Kit de Desarrollo de Agentes](https://docs.cloud.google.com/agent-builder/agent-engine/use/adk#rest-api).
También puedes consultar la documentación de Agent Engine para aprender sobre cómo gestionar tu
[agente desplegado](https://docs.cloud.google.com/agent-builder/agent-engine/manage/overview).
Para más información sobre pruebas e interacción con un agente desplegado, consulta
[Probar agentes desplegados en Agent Engine](/adk-docs/deploy/agent-engine/test/).

### Monitoreo y verificación

*   Puedes monitorear el estado del despliegue en la
    [IU de Agent Engine](https://console.cloud.google.com/vertex-ai/agents/agent-engines)
    en la Consola de Google Cloud.
*   Para detalles adicionales, puedes visitar la documentación de Agent Engine sobre
    [desplegar un agente](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/deploy)
    y
    [gestionar agentes desplegados](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/manage/overview).

## Probar agentes desplegados

Después de completar el despliegue de tu agente ADK, debes probar el flujo de trabajo en
su nuevo entorno alojado. Para más información sobre cómo probar un agente ADK
desplegado en Agent Engine, consulta
[Probar agentes desplegados en Agent Engine](/adk-docs/deploy/agent-engine/test/).