# Desplegar en Cloud Run

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span><span class="lst-go">Go</span><span class="lst-java">Java</span>
</div>

[Cloud Run](https://cloud.google.com/run)
es una plataforma totalmente administrada que te permite ejecutar tu código directamente sobre la infraestructura escalable de Google.

Para desplegar tu agente, puedes usar el comando `adk deploy cloud_run` _(recomendado para Python)_, o con el comando `gcloud run deploy` a través de Cloud Run.

## Ejemplo de agente

Para cada uno de los comandos, haremos referencia al ejemplo `Capital Agent` definido en la página [Agente LLM](../agents/llm-agents.md). Asumiremos que está en un directorio (ej: `capital_agent`).

Para continuar, confirma que el código de tu agente esté configurado de la siguiente manera:

=== "Python"

    1. El código del agente está en un archivo llamado `agent.py` dentro del directorio de tu agente.
    2. Tu variable de agente se llama `root_agent`.
    3. `__init__.py` está dentro del directorio de tu agente y contiene `from . import agent`.
    4. Tu archivo `requirements.txt` está presente en el directorio del agente.

=== "Go"

    1. El punto de entrada de tu aplicación (el paquete main y la función main()) está en un
       único archivo Go. Usar main.go es una convención fuerte.
    2. Tu instancia del agente se pasa a una configuración de lanzador, típicamente usando
       agent.NewSingleLoader(yourAgent). La herramienta adkgo usa este lanzador para iniciar
       tu agente con los servicios correctos.
    3. Tus archivos go.mod y go.sum están presentes en el directorio de tu proyecto para gestionar
       las dependencias.

    Consulta la siguiente sección para más detalles. También puedes encontrar una [aplicación de ejemplo](https://github.com/google/tree/main/examples/go/cloud-run) en el repositorio de Github.

=== "Java"

    1. El código del agente está en un archivo llamado `CapitalAgent.java` dentro del directorio de tu agente.
    2. Tu variable de agente es global y sigue el formato `public static final BaseAgent ROOT_AGENT`.
    3. La definición de tu agente está presente en un método de clase estático.

    Consulta la siguiente sección para más detalles. También puedes encontrar una [aplicación de ejemplo](https://github.com/google/tree/main/examples/java/cloud-run) en el repositorio de Github.


## Variables de entorno

Configura tus variables de entorno como se describe en la guía [Configuración e Instalación](../get-started/installation.md).

```bash
export GOOGLE_CLOUD_PROJECT=your-project-id
export GOOGLE_CLOUD_LOCATION=us-central1 # O tu ubicación preferida
export GOOGLE_GENAI_USE_VERTEXAI=True
```

_(Reemplaza `your-project-id` con tu ID de proyecto GCP real)_

Alternativamente, también puedes usar una clave API de AI Studio

```bash
export GOOGLE_CLOUD_PROJECT=your-project-id
export GOOGLE_CLOUD_LOCATION=us-central1 # O tu ubicación preferida
export GOOGLE_GENAI_USE_VERTEXAI=FALSE
export GOOGLE_API_KEY=your-api-key
```
*(Reemplaza `your-project-id` con tu ID de proyecto GCP real y `your-api-key` con tu clave API real de AI Studio)*

## Prerrequisitos

1. Debes tener un proyecto de Google Cloud. Necesitas conocer tu:
    1. Nombre del proyecto (ej. "my-project")
    1. Ubicación del proyecto (ej. "us-central1")
    1. Cuenta de servicio (ej. "1234567890-compute@developer.gserviceaccount.com")
    1. GOOGLE_API_KEY 

## Secreto

Por favor, asegúrate de haber creado un secreto que pueda ser leído por tu cuenta de servicio.

### Entrada para el secreto GOOGLE_API_KEY

Puedes crear tu secreto manualmente o usar CLI:
```bash
echo "<<pon tu GOOGLE_API_KEY aquí>>" | gcloud secrets create GOOGLE_API_KEY --project=my-project --data-file=-
```

### Permisos para leer
Debes otorgar el permiso apropiado para que tu cuenta de servicio lea este secreto.
```bash
gcloud secrets add-iam-policy-binding GOOGLE_API_KEY --member="serviceAccount:1234567890-compute@developer.gserviceaccount.com" --role="roles/secretmanager.secretAccessor" --project=my-project
```

## Carga útil de despliegue {#payload}

Cuando despliegas tu flujo de trabajo del agente ADK en Google Cloud Run,
el siguiente contenido se carga al servicio:

- Tu código del agente ADK
- Cualquier dependencia declarada en tu código del agente ADK
- Versión del código del servidor API de ADK usado por tu agente

El despliegue predeterminado *no* incluye las bibliotecas de la interfaz de usuario web de ADK,
a menos que lo especifiques como una configuración de despliegue, como la opción `--with_ui` para
el comando `adk deploy cloud_run`.

## Comandos de despliegue

=== "Python - adk CLI"

    ###  adk CLI

    El comando `adk deploy cloud_run` despliega tu código de agente en Google Cloud Run.

    Asegúrate de haberte autenticado con Google Cloud (`gcloud auth login` y `gcloud config set project <your-project-id>`).

    #### Configurar variables de entorno

    Opcional pero recomendado: Configurar variables de entorno puede hacer que los comandos de despliegue sean más limpios.

    ```bash
    # Configura tu ID de Proyecto de Google Cloud
    export GOOGLE_CLOUD_PROJECT="your-gcp-project-id"

    # Configura tu Ubicación de Google Cloud deseada
    export GOOGLE_CLOUD_LOCATION="us-central1" # Ubicación de ejemplo

    # Configura la ruta al directorio del código de tu agente
    export AGENT_PATH="./capital_agent" # Asumiendo que capital_agent está en el directorio actual

    # Configura un nombre para tu servicio de Cloud Run (opcional)
    export SERVICE_NAME="capital-agent-service"

    # Configura un nombre de aplicación (opcional)
    export APP_NAME="capital_agent_app"
    ```

    #### Uso del comando

    ##### Comando mínimo

    ```bash
    adk deploy cloud_run \
    --project=$GOOGLE_CLOUD_PROJECT \
    --region=$GOOGLE_CLOUD_LOCATION \
    $AGENT_PATH
    ```

    ##### Comando completo con banderas opcionales

    ```bash
    adk deploy cloud_run \
    --project=$GOOGLE_CLOUD_PROJECT \
    --region=$GOOGLE_CLOUD_LOCATION \
    --service_name=$SERVICE_NAME \
    --app_name=$APP_NAME \
    --with_ui \
    $AGENT_PATH
    ```

    ##### Argumentos

    * `AGENT_PATH`: (Requerido) Argumento posicional que especifica la ruta al directorio que contiene el código fuente de tu agente (ej., `$AGENT_PATH` en los ejemplos, o `capital_agent/`). Este directorio debe contener al menos un `__init__.py` y tu archivo principal del agente (ej., `agent.py`).

    ##### Opciones

    * `--project TEXT`: (Requerido) Tu ID de proyecto de Google Cloud (ej., `$GOOGLE_CLOUD_PROJECT`).
    * `--region TEXT`: (Requerido) La ubicación de Google Cloud para el despliegue (ej., `$GOOGLE_CLOUD_LOCATION`, `us-central1`).
    * `--service_name TEXT`: (Opcional) El nombre para el servicio de Cloud Run (ej., `$SERVICE_NAME`). Por defecto es `adk-default-service-name`.
    * `--app_name TEXT`: (Opcional) El nombre de la aplicación para el servidor API de ADK (ej., `$APP_NAME`). Por defecto es el nombre del directorio especificado por `AGENT_PATH` (ej., `capital_agent` si `AGENT_PATH` es `./capital_agent`).
    * `--agent_engine_id TEXT`: (Opcional) Si estás usando un servicio de sesión administrado a través de Vertex AI Agent Engine, proporciona su ID de recurso aquí.
    * `--port INTEGER`: (Opcional) El número de puerto en el que el servidor API de ADK escuchará dentro del contenedor. Por defecto es 8000.
    * `--with_ui`: (Opcional) Si se incluye, despliega la interfaz de usuario de desarrollo de ADK junto con el servidor API del agente. Por defecto, solo se despliega el servidor API.
    * `--temp_folder TEXT`: (Opcional) Especifica un directorio para almacenar archivos intermedios generados durante el proceso de despliegue. Por defecto es una carpeta con marca de tiempo en el directorio temporal del sistema. *(Nota: Esta opción generalmente no se necesita a menos que estés solucionando problemas).*
    * `--help`: Muestra el mensaje de ayuda y sale.

    ##### Acceso autenticado
    Durante el proceso de despliegue, se te podría preguntar: `Allow unauthenticated invocations to [your-service-name] (y/N)?`.

    * Ingresa `y` para permitir acceso público al endpoint API de tu agente sin autenticación.
    * Ingresa `N` (o presiona Enter para el predeterminado) para requerir autenticación (ej., usando un token de identidad como se muestra en la sección "Probando tu agente").

    Tras una ejecución exitosa, el comando despliega tu agente en Cloud Run y proporciona la URL del servicio desplegado.

=== "Python - gcloud CLI"

    ### gcloud CLI para Python

    Alternativamente, puedes desplegar usando el comando estándar `gcloud run deploy` con un `Dockerfile`. Este método requiere más configuración manual en comparación con el comando `adk` pero ofrece flexibilidad, particularmente si deseas integrar tu agente dentro de una aplicación personalizada de [FastAPI](https://fastapi.tiangolo.com/).

    Asegúrate de haberte autenticado con Google Cloud (`gcloud auth login` y `gcloud config set project <your-project-id>`).

    #### Estructura del proyecto

    Organiza los archivos de tu proyecto de la siguiente manera:

    ```txt
    your-project-directory/
    ├── capital_agent/
    │   ├── __init__.py
    │   └── agent.py       # Tu código del agente (ver pestaña "Agent sample")
    ├── main.py            # Punto de entrada de la aplicación FastAPI
    ├── requirements.txt   # Dependencias de Python
    └── Dockerfile         # Instrucciones de construcción del contenedor
    ```

    Crea los siguientes archivos (`main.py`, `requirements.txt`, `Dockerfile`) en la raíz de `your-project-directory/`.

    #### Archivos de código

    1. Este archivo configura la aplicación FastAPI usando `get_fast_api_app()` de ADK:

        ```python title="main.py"
        import os

        import uvicorn
        from fastapi import FastAPI
        from google.adk.cli.fast_api import get_fast_api_app

        # Obtiene el directorio donde se encuentra main.py
        AGENT_DIR = os.path.dirname(os.path.abspath(__file__))
        # URI del servicio de sesión de ejemplo (ej., SQLite)
        # Nota: Usa 'sqlite+aiosqlite' en lugar de 'sqlite' porque DatabaseSessionService requiere un controlador asíncrono
        SESSION_SERVICE_URI = "sqlite+aiosqlite:///./sessions.db"
        # Orígenes permitidos de ejemplo para CORS
        ALLOWED_ORIGINS = ["http://localhost", "http://localhost:8080", "*"]
        # Configura web=True si pretendes servir una interfaz web, False de lo contrario
        SERVE_WEB_INTERFACE = True

        # Llama a la función para obtener la instancia de la aplicación FastAPI
        # Asegúrate de que el nombre del directorio del agente ('capital_agent') coincida con tu carpeta de agente
        app: FastAPI = get_fast_api_app(
            agents_dir=AGENT_DIR,
            session_service_uri=SESSION_SERVICE_URI,
            allow_origins=ALLOWED_ORIGINS,
            web=SERVE_WEB_INTERFACE,
        )

        # Puedes agregar más rutas o configuraciones de FastAPI a continuación si es necesario
        # Ejemplo:
        # @app.get("/hello")
        # async def read_root():
        #     return {"Hello": "World"}

        if __name__ == "__main__":
            # Usa la variable de entorno PORT proporcionada por Cloud Run, por defecto 8080
            uvicorn.run(app, host="0.0.0.0", port=int(os.environ.get("PORT", 8080)))
        ```

        *Nota: Especificamos `agent_dir` al directorio en el que se encuentra `main.py` y usamos `os.environ.get("PORT", 8080)` para compatibilidad con Cloud Run.*

    2. Lista los paquetes de Python necesarios:

        ```txt title="requirements.txt"
        google-adk
        # Agrega cualquier otra dependencia que tu agente necesite
        ```

    3. Define la imagen del contenedor:

        ```dockerfile title="Dockerfile"
        FROM python:3.13-slim
        WORKDIR /app

        COPY requirements.txt .
        RUN pip install --no-cache-dir -r requirements.txt

        RUN adduser --disabled-password --gecos "" myuser && \
            chown -R myuser:myuser /app

        COPY . .

        USER myuser

        ENV PATH="/home/myuser/.local/bin:$PATH"

        CMD ["sh", "-c", "uvicorn main:app --host 0.0.0.0 --port $PORT"]
        ```

    #### Definiendo múltiples agentes

    Puedes definir y desplegar múltiples agentes dentro de la misma instancia de Cloud Run creando carpetas separadas en la raíz de `your-project-directory/`. Cada carpeta representa un agente y debe definir un `root_agent` en su configuración.

    Estructura de ejemplo:

    ```txt
    your-project-directory/
    ├── capital_agent/
    │   ├── __init__.py
    │   └── agent.py       # contiene la definición de `root_agent`
    ├── population_agent/
    │   ├── __init__.py
    │   └── agent.py       # contiene la definición de `root_agent`
    └── ...
    ```

    #### Desplegar usando `gcloud`

    Navega a `your-project-directory` en tu terminal.

    ```bash
    gcloud run deploy capital-agent-service \
    --source . \
    --region $GOOGLE_CLOUD_LOCATION \
    --project $GOOGLE_CLOUD_PROJECT \
    --allow-unauthenticated \
    --set-env-vars="GOOGLE_CLOUD_PROJECT=$GOOGLE_CLOUD_PROJECT,GOOGLE_CLOUD_LOCATION=$GOOGLE_CLOUD_LOCATION,GOOGLE_GENAI_USE_VERTEXAI=$GOOGLE_GENAI_USE_VERTEXAI"
    # Agrega cualquier otra variable de entorno necesaria que tu agente pueda necesitar
    ```

    * `capital-agent-service`: El nombre que deseas darle a tu servicio de Cloud Run.
    * `--source .`: Le dice a gcloud que construya la imagen del contenedor desde el Dockerfile en el directorio actual.
    * `--region`: Especifica la región de despliegue.
    * `--project`: Especifica el proyecto GCP.
    * `--allow-unauthenticated`: Permite acceso público al servicio. Elimina esta bandera para servicios privados.
    * `--set-env-vars`: Pasa las variables de entorno necesarias al contenedor en ejecución. Asegúrate de incluir todas las variables requeridas por ADK y tu agente (como claves API si no estás usando Credenciales Predeterminadas de la Aplicación).

    `gcloud` construirá la imagen de Docker, la enviará a Google Artifact Registry y la desplegará en Cloud Run. Al completarse, generará la URL de tu servicio desplegado.

    Para una lista completa de opciones de despliegue, consulta la [documentación de referencia de `gcloud run deploy`](https://cloud.google.com/sdk/gcloud/reference/run/deploy).

=== "Go - adkgo CLI"

    ### adk CLI

    El comando adkgo se encuentra en el repositorio google/adk-go bajo cmd/adkgo. Antes de usarlo, necesitas construirlo desde la raíz del repositorio adk-go:

    `go build ./cmd/adkgo`

    El comando adkgo deploy cloudrun automatiza el despliegue de tu aplicación. No necesitas proporcionar tu propio Dockerfile.

    #### Estructura del código del agente

    Al usar la herramienta adkgo, tu archivo main.go debe usar el marco de lanzador. Esto se debe a que la herramienta compila tu código y luego ejecuta el ejecutable resultante con argumentos de línea de comandos específicos (como web, api, a2a) para iniciar los servicios requeridos. El lanzador está diseñado para analizar estos argumentos correctamente.

    Tu main.go debería verse así:

    ```go title="main.go"
    --8<-- "examples/go/cloud-run/main.go"
    ```

    #### Cómo funciona
    1. La herramienta adkgo compila tu main.go en un binario enlazado estáticamente para Linux.
    2. Genera un Dockerfile que copia este binario en un contenedor mínimo.
    3. Usa gcloud para construir y desplegar este contenedor en Cloud Run.
    4. Después del despliegue, inicia un proxy local que se conecta de forma segura a tu nuevo
        servicio.

    Asegúrate de haberte autenticado con Google Cloud (`gcloud auth login` y `gcloud config set project <your-project-id>`).

    #### Configurar variables de entorno

    Opcional pero recomendado: Configurar variables de entorno puede hacer que los comandos de despliegue sean más limpios.

    ```bash
    # Configura tu ID de Proyecto de Google Cloud
    export GOOGLE_CLOUD_PROJECT="your-gcp-project-id"

    # Configura tu Ubicación de Google Cloud deseada
    export GOOGLE_CLOUD_LOCATION="us-central1"

    # Configura la ruta al archivo Go principal de tu agente
    export AGENT_PATH="./examples/go/cloud-run/main.go"

    # Configura un nombre para tu servicio de Cloud Run
    export SERVICE_NAME="capital-agent-service"
    ```

    #### Uso del comando

    ```bash
    ./adkgo deploy cloudrun \
        -p $GOOGLE_CLOUD_PROJECT \
        -r $GOOGLE_CLOUD_LOCATION \
        -s $SERVICE_NAME \
        --proxy_port=8081 \
        --server_port=8080 \
        -e $AGENT_PATH \
        --a2a --api --webui
    ```

    ##### Requerido

    * `-p, --project_name`: Tu ID de proyecto de Google Cloud (ej., $GOOGLE_CLOUD_PROJECT).
    * `-r, --region`: La ubicación de Google Cloud para el despliegue (ej., $GOOGLE_CLOUD_LOCATION, us-central1).
    * `-s, --service_name`: El nombre para el servicio de Cloud Run (ej., $SERVICE_NAME).
    * `-e, --entry_point_path`: Ruta al archivo Go principal que contiene el código fuente de tu agente (ej., $AGENT_PATH).

    ##### Opcional

    * `--proxy_port`: El puerto local para que el proxy de autenticación escuche. Por defecto es 8081.
    * `--server_port`: El número de puerto en el que el servidor escuchará dentro del contenedor de Cloud Run. Por defecto es 8080.
    * `--a2a`: Si se incluye, habilita la comunicación Agent2Agent. Habilitado por defecto.
    * `--a2a_agent_url`: URL de la tarjeta del agente A2A como se anuncia en la tarjeta pública del agente. Esta bandera solo es válida cuando se usa con la bandera --a2a.
    * `--api`: Si se incluye, despliega el servidor API de ADK. Habilitado por defecto.
    * `--webui`: Si se incluye, despliega la interfaz de usuario de desarrollo de ADK junto con el servidor API del agente. Habilitado por defecto.
    * `--temp_dir`: Directorio temporal para artefactos de construcción. Por defecto es os.TempDir().
    * `--help`: Muestra el mensaje de ayuda y sale.

    ##### Acceso autenticado
    El servicio se despliega con --no-allow-unauthenticated por defecto.

    Tras una ejecución exitosa, el comando despliega tu agente en Cloud Run y proporciona una URL local para acceder al servicio a través del proxy.

=== "Java - gcloud CLI"

    ### gcloud CLI para Java

    Puedes desplegar agentes Java usando el comando estándar `gcloud run deploy` y un `Dockerfile`. Esta es la forma recomendada actualmente para desplegar agentes Java en Google Cloud Run.

    Asegúrate de estar [autenticado](https://cloud.google.com/docs/authentication/gcloud) con Google Cloud.
    Específicamente, ejecuta los comandos `gcloud auth login` y `gcloud config set project <your-project-id>` desde tu terminal.

    #### Estructura del proyecto

    Organiza los archivos de tu proyecto de la siguiente manera:

    ```txt
    your-project-directory/
    ├── src/
    │   └── main/
    │       └── java/
    │             └── agents/
    │                 ├── capitalagent/
    │                     └── CapitalAgent.java    # Tu código del agente
    ├── pom.xml                                    # Dependencias de adk y adk-dev de Java
    └── Dockerfile                                 # Instrucciones de construcción del contenedor
    ```

    Crea el `pom.xml` y el `Dockerfile` en la raíz del directorio de tu proyecto. Tu archivo de código del agente (`CapitalAgent.java`) dentro de un directorio como se muestra arriba.

    #### Archivos de código

    1. Esta es nuestra definición de agente. Este es el mismo código que está presente en [Agente LLM](../agents/llm-agents.md) con dos salvedades:

           * El agente ahora se inicializa como una **variable global public static final**.

           * La definición del agente puede exponerse en un método estático o en línea durante la declaración.

        Consulta el código para el ejemplo `CapitalAgent` en el 
        repositorio de [ejemplos](https://github.com/google/blob/main/examples/java/cloud-run/src/main/java/agents/capitalagent/CapitalAgent.java).

    2. Agrega las siguientes dependencias y plugin al archivo pom.xml.

        ```xml title="pom.xml"
        <dependencies>
          <dependency>
             <groupId>com.google.adk</groupId>
             <artifactId>google-adk</artifactId>
             <version>0.5.0</version>
          </dependency>
          <dependency>
             <groupId>com.google.adk</groupId>
             <artifactId>google-adk-dev</artifactId>
             <version>0.5.0</version>
          </dependency>
        </dependencies>

        <plugin>
          <groupId>org.codehaus.mojo</groupId>
          <artifactId>exec-maven-plugin</artifactId>
          <version>3.2.0</version>
          <configuration>
            <mainClass>com.google.adk.web.AdkWebServer</mainClass>
            <classpathScope>compile</classpathScope>
          </configuration>
        </plugin>
        ```

    3.  Define la imagen del contenedor:

        ```dockerfile title="Dockerfile"
        --8<-- "examples/java/cloud-run/Dockerfile"
        ```

    #### Desplegar usando `gcloud`

    Navega a `your-project-directory` en tu terminal.

    ```bash
    gcloud run deploy capital-agent-service \
    --source . \
    --region $GOOGLE_CLOUD_LOCATION \
    --project $GOOGLE_CLOUD_PROJECT \
    --allow-unauthenticated \
    --set-env-vars="GOOGLE_CLOUD_PROJECT=$GOOGLE_CLOUD_PROJECT,GOOGLE_CLOUD_LOCATION=$GOOGLE_CLOUD_LOCATION,GOOGLE_GENAI_USE_VERTEXAI=$GOOGLE_GENAI_USE_VERTEXAI"
    # Agrega cualquier otra variable de entorno necesaria que tu agente pueda necesitar
    ```

    * `capital-agent-service`: El nombre que deseas darle a tu servicio de Cloud Run.
    * `--source .`: Le dice a gcloud que construya la imagen del contenedor desde el Dockerfile en el directorio actual.
    * `--region`: Especifica la región de despliegue.
    * `--project`: Especifica el proyecto GCP.
    * `--allow-unauthenticated`: Permite acceso público al servicio. Elimina esta bandera para servicios privados.
    * `--set-env-vars`: Pasa las variables de entorno necesarias al contenedor en ejecución. Asegúrate de incluir todas las variables requeridas por ADK y tu agente (como claves API si no estás usando Credenciales Predeterminadas de la Aplicación).

    `gcloud` construirá la imagen de Docker, la enviará a Google Artifact Registry y la desplegará en Cloud Run. Al completarse, generará la URL de tu servicio desplegado.

    Para una lista completa de opciones de despliegue, consulta la [documentación de referencia de `gcloud run deploy`](https://cloud.google.com/sdk/gcloud/reference/run/deploy).

## Probando tu agente

Una vez que tu agente esté desplegado en Cloud Run, puedes interactuar con él a través de la interfaz de usuario desplegada (si está habilitada) o directamente con sus endpoints API usando herramientas como `curl`. Necesitarás la URL del servicio proporcionada después del despliegue.

=== "UI Testing"

    ### Pruebas de interfaz de usuario

    Si desplegaste tu agente con la interfaz de usuario habilitada:

    *   **adk CLI:** Incluiste la bandera `--webui` durante el despliegue.
    *   **gcloud CLI:** Configuraste `SERVE_WEB_INTERFACE = True` en tu `main.py`.

    Puedes probar tu agente simplemente navegando a la URL del servicio de Cloud Run proporcionada después del despliegue en tu navegador web.

    ```bash
    # Formato de URL de ejemplo
    # https://your-service-name-abc123xyz.a.run.app
    ```

    La interfaz de usuario de desarrollo de ADK te permite interactuar con tu agente, gestionar sesiones y ver detalles de ejecución directamente en el navegador.

    Para verificar que tu agente está funcionando como se esperaba, puedes:

    1. Seleccionar tu agente del menú desplegable.
    2. Escribir un mensaje y verificar que recibes una respuesta esperada de tu agente.

    Si experimentas algún comportamiento inesperado, revisa los registros de la consola de [Cloud Run](https://console.cloud.google.com/run).

=== "API Testing (curl)"

    ### Pruebas de API (curl)

    Puedes interactuar con los endpoints API del agente usando herramientas como `curl`. Esto es útil para interacción programática o si desplegaste sin la interfaz de usuario.

    Necesitarás la URL del servicio proporcionada después del despliegue y potencialmente un token de identidad para autenticación si tu servicio no está configurado para permitir acceso no autenticado.

    #### Configurar la URL de la aplicación

    Reemplaza la URL de ejemplo con la URL real de tu servicio de Cloud Run desplegado.

    ```bash
    export APP_URL="YOUR_CLOUD_RUN_SERVICE_URL"
    # Ejemplo: export APP_URL="https://adk-default-service-name-abc123xyz.a.run.app"
    ```

    #### Obtener un token de identidad (si es necesario)

    Si tu servicio requiere autenticación (es decir, no usaste `--allow-unauthenticated` con `gcloud` o respondiste 'N' a la pregunta con `adk`), obtén un token de identidad.

    ```bash
    export TOKEN=$(gcloud auth print-identity-token)
    ```

    *Si tu servicio permite acceso no autenticado, puedes omitir el encabezado `-H "Authorization: Bearer $TOKEN"` de los comandos `curl` a continuación.*

    #### Listar aplicaciones disponibles

    Verifica el nombre de la aplicación desplegada.

    ```bash
    curl -X GET -H "Authorization: Bearer $TOKEN" $APP_URL/list-apps
    ```

    *(Ajusta el `app_name` en los siguientes comandos según esta salida si es necesario. El predeterminado es a menudo el nombre del directorio del agente, ej., `capital_agent`)*.

    #### Crear o actualizar una sesión

    Inicializa o actualiza el estado para un usuario y sesión específicos. Reemplaza `capital_agent` con el nombre real de tu aplicación si es diferente. Los valores `user_123` y `session_abc` son identificadores de ejemplo; puedes reemplazarlos con tus IDs de usuario y sesión deseados.

    ```bash
    curl -X POST -H "Authorization: Bearer $TOKEN" \
        $APP_URL/apps/capital_agent/users/user_123/sessions/session_abc \
        -H "Content-Type: application/json" \
        -d '{"preferred_language": "English", "visit_count": 5}'
    ```

    #### Ejecutar el agente

    Envía un prompt a tu agente. Reemplaza `capital_agent` con el nombre de tu aplicación y ajusta los IDs de usuario/sesión y el prompt según sea necesario.

    ```bash
    curl -X POST -H "Authorization: Bearer $TOKEN" \
        $APP_URL/run_sse \
        -H "Content-Type: application/json" \
        -d '{
        "app_name": "capital_agent",
        "user_id": "user_123",
        "session_id": "session_abc",
        "new_message": {
            "role": "user",
            "parts": [{
            "text": "What is the capital of Canada?"
            }]
        },
        "streaming": false
        }'
    ```

    * Configura `"streaming": true` si deseas recibir Eventos Enviados por el Servidor (SSE).
    * La respuesta contendrá los eventos de ejecución del agente, incluyendo la respuesta final.