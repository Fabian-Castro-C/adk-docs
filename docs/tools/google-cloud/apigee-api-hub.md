---
catalog_title: Apigee API Hub
catalog_description: Convierte cualquier API documentada desde Apigee API hub en una herramienta
catalog_icon: /adk-docs/assets/tools-apigee.png
---

# Herramientas de Apigee API Hub para ADK

<div class="language-support-tag">
  <span class="lst-supported">Compatible con ADK</span><span class="lst-python">Python v0.1.0</span>
</div>

**ApiHubToolset** te permite convertir cualquier API documentada desde Apigee API hub en una
herramienta con pocas líneas de código. Esta sección te muestra las instrucciones paso a paso
incluyendo la configuración de autenticación para una conexión segura a tus
APIs.

**Requisitos previos**

1. [Instalar ADK](/adk-docs/get-started/installation/)
2. Instalar el
   [CLI de Google Cloud](https://cloud.google.com/sdk/docs/install?db=bigtable-docs#installation_instructions).
3. Instancia de [Apigee API hub](https://cloud.google.com/apigee/docs/apihub/what-is-api-hub)
    con APIs documentadas (es decir, especificación OpenAPI)
4. Configurar la estructura de tu proyecto y crear los archivos requeridos

```console
project_root_folder
 |
 `-- my_agent
     |-- .env
     |-- __init__.py
     |-- agent.py
     `__ tool.py
```

## Crear un API Hub Toolset

Nota: Este tutorial incluye la creación de un agente. Si ya tienes un agente,
solo necesitas seguir un subconjunto de estos pasos.

1. Obtén tu token de acceso, para que APIHubToolset pueda obtener la especificación desde la API de API Hub.
   En tu terminal ejecuta el siguiente comando

    ```shell
    gcloud auth print-access-token
    # Imprime tu token de acceso como 'ya29....'
    ```

2. Asegúrate de que la cuenta utilizada tenga los permisos requeridos. Puedes usar el
   rol predefinido `roles/apihub.viewer` o asignar los siguientes permisos:

    1. **apihub.specs.get (requerido)**
    2. apihub.apis.get (opcional)
    3. apihub.apis.list (opcional)
    4. apihub.versions.get (opcional)
    5. apihub.versions.list (opcional)
    6. apihub.specs.list (opcional)

3. Crea una herramienta con `APIHubToolset`. Agrega lo siguiente a `tools.py`

    Si tu API requiere autenticación, debes configurar la autenticación para
    la herramienta. El siguiente ejemplo de código demuestra cómo configurar una
    clave API. ADK soporta autenticación basada en tokens (clave API, token Bearer), cuenta de servicio,
    y OpenID Connect. Pronto añadiremos soporte para varios flujos OAuth2.

    ```py
    from google.adk.tools.openapi_tool.auth.auth_helpers import token_to_scheme_credential
    from google.adk.tools.apihub_tool.apihub_toolset import APIHubToolset

    # Proporciona autenticación para tus APIs. No es requerido si tus APIs no requieren autenticación.
    auth_scheme, auth_credential = token_to_scheme_credential(
        "apikey", "query", "apikey", apikey_credential_str
    )

    sample_toolset = APIHubToolset(
        name="apihub-sample-tool",
        description="Sample Tool",
        access_token="...",  # Copia tu token de acceso generado en el paso 1
        apihub_resource_name="...", # Nombre del recurso de API Hub
        auth_scheme=auth_scheme,
        auth_credential=auth_credential,
    )
    ```

    Para despliegue en producción recomendamos usar una cuenta de servicio en lugar de un
    token de acceso. En el fragmento de código anterior, usa
    `service_account_json=service_account_cred_json_str` y proporciona tus
    credenciales de cuenta de seguridad en lugar del token.

    Para apihub\_resource\_name, si conoces el ID específico de la especificación OpenAPI
    que se está utilizando para tu API, usa
    `` `projects/my-project-id/locations/us-west1/apis/my-api-id/versions/version-id/specs/spec-id` ``.
    Si deseas que el Toolset obtenga automáticamente la primera especificación disponible
    de la API, usa
    `` `projects/my-project-id/locations/us-west1/apis/my-api-id` ``

4. Crea tu archivo de agente Agent.py y agrega las herramientas creadas a la definición de tu agente:

    ```py
    from google.adk.agents.llm_agent import LlmAgent
    from .tools import sample_toolset

    root_agent = LlmAgent(
        model='gemini-2.0-flash',
        name='enterprise_assistant',
        instruction='Help user, leverage the tools you have access to',
        tools=sample_toolset.get_tools(),
    )
    ```

5. Configura tu `__init__.py` para exponer tu agente

    ```py
    from . import agent
    ```

6. Inicia la interfaz web de Google ADK y prueba tu agente:

    ```shell
    # asegúrate de ejecutar `adk web` desde tu project_root_folder
    adk web
    ```

   Luego ve a [http://localhost:8000](http://localhost:8000) para probar tu agente desde la interfaz web.