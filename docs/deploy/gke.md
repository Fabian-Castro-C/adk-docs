# Desplegar en Google Kubernetes Engine (GKE)

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python</span>
</div>

[GKE](https://cloud.google.com/gke) es el servicio de Kubernetes administrado de Google Cloud. Te permite desplegar y gestionar aplicaciones en contenedores usando Kubernetes.

Para desplegar tu agente necesitarás tener un clúster de Kubernetes ejecutándose en GKE. Puedes crear un clúster usando la consola de Google Cloud o la herramienta de línea de comandos `gcloud`.

En este ejemplo desplegaremos un agente simple a GKE. El agente será una aplicación FastAPI que usa `Gemini 2.0 Flash` como LLM. Podemos usar Vertex AI o AI Studio como proveedor de LLM usando la variable de entorno `GOOGLE_GENAI_USE_VERTEXAI`.

## Variables de entorno

Configura tus variables de entorno como se describe en la guía de [Configuración e Instalación](../get-started/installation.md). También necesitas instalar la herramienta de línea de comandos `kubectl`. Puedes encontrar instrucciones para hacerlo en la [Documentación de Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl).

```bash
export GOOGLE_CLOUD_PROJECT=your-project-id # Tu ID de proyecto de GCP
export GOOGLE_CLOUD_LOCATION=us-central1 # O tu ubicación preferida
export GOOGLE_GENAI_USE_VERTEXAI=true # Establece a true si usas Vertex AI
export GOOGLE_CLOUD_PROJECT_NUMBER=$(gcloud projects describe --format json $GOOGLE_CLOUD_PROJECT | jq -r ".projectNumber")
```

Si no tienes `jq` instalado, puedes usar el siguiente comando para obtener el número de proyecto:

```bash
gcloud projects describe $GOOGLE_CLOUD_PROJECT
```

Y copiar el número de proyecto de la salida.

```bash
export GOOGLE_CLOUD_PROJECT_NUMBER=YOUR_PROJECT_NUMBER
```



## Habilitar APIs y Permisos

Asegúrate de haberte autenticado con Google Cloud (`gcloud auth login` y `gcloud config set project <your-project-id>`).

Habilita las APIs necesarias para tu proyecto. Puedes hacer esto usando la herramienta de línea de comandos `gcloud`.

```bash
gcloud services enable \
    container.googleapis.com \
    artifactregistry.googleapis.com \
    cloudbuild.googleapis.com \
    aiplatform.googleapis.com
```

Otorga los roles necesarios a la cuenta de servicio predeterminada de compute engine requerida por el comando `gcloud builds submit`.



```bash
ROLES_TO_ASSIGN=(
    "roles/artifactregistry.writer"
    "roles/storage.objectViewer"
    "roles/logging.viewer"
    "roles/logging.logWriter"
)

for ROLE in "${ROLES_TO_ASSIGN[@]}"; do
    gcloud projects add-iam-policy-binding "${GOOGLE_CLOUD_PROJECT}" \
        --member="serviceAccount:${GOOGLE_CLOUD_PROJECT_NUMBER}-compute@developer.gserviceaccount.com" \
        --role="${ROLE}"
done
```

## Contenido del despliegue {#payload}

Cuando despliegas tu flujo de trabajo de agente ADK en Google Cloud GKE,
el siguiente contenido se sube al servicio:

- Tu código de agente ADK
- Cualquier dependencia declarada en tu código de agente ADK
- Versión del código del servidor API de ADK usado por tu agente

El despliegue predeterminado *no* incluye las bibliotecas de la interfaz de usuario web de ADK,
a menos que lo especifiques como configuración de despliegue, como la opción `--with_ui` para
el comando `adk deploy gke`.

## Opciones de despliegue

Puedes desplegar tu agente a GKE ya sea **manualmente usando manifiestos de Kubernetes** o **automáticamente usando el comando `adk deploy gke`**. Elige el enfoque que mejor se adapte a tu flujo de trabajo.


## Opción 1: Despliegue Manual usando gcloud y kubectl

### Crear un clúster GKE

Puedes crear un clúster GKE usando la herramienta de línea de comandos `gcloud`. Este ejemplo crea un clúster Autopilot llamado `adk-cluster` en la región `us-central1`.

> Si creas un clúster GKE Standard, asegúrate de que [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity) esté habilitado. Workload Identity está habilitado por defecto en un clúster AutoPilot.

```bash
gcloud container clusters create-auto adk-cluster \
    --location=$GOOGLE_CLOUD_LOCATION \
    --project=$GOOGLE_CLOUD_PROJECT
```

Después de crear el clúster, necesitas conectarte a él usando `kubectl`. Este comando configura `kubectl` para usar las credenciales de tu nuevo clúster.

```bash
gcloud container clusters get-credentials adk-cluster \
    --location=$GOOGLE_CLOUD_LOCATION \
    --project=$GOOGLE_CLOUD_PROJECT
```

### Crear Tu Agente

Haremos referencia al ejemplo de `capital_agent` definido en la página de [Agentes LLM](../agents/llm-agents.md).

Para proceder, organiza los archivos de tu proyecto de la siguiente manera:

```txt
your-project-directory/
├── capital_agent/
│   ├── __init__.py
│   └── agent.py       # Tu código de agente (ver "Ejemplo de Capital Agent" abajo)
├── main.py            # Punto de entrada de la aplicación FastAPI
├── requirements.txt   # Dependencias de Python
└── Dockerfile         # Instrucciones de construcción del contenedor
```



### Archivos de código

Crea los siguientes archivos (`main.py`, `requirements.txt`, `Dockerfile`, `capital_agent/agent.py`, `capital_agent/__init__.py`) en la raíz de `your-project-directory/`.

1. Este es el ejemplo de Capital Agent dentro del directorio `capital_agent`

    ```python title="capital_agent/agent.py"
    from google.adk.agents import LlmAgent 

    # Define una función de herramienta
    def get_capital_city(country: str) -> str:
      """Recupera la ciudad capital para un país dado."""
      # Reemplaza con lógica real (por ejemplo, llamada a API, búsqueda en base de datos)
      capitals = {"france": "Paris", "japan": "Tokyo", "canada": "Ottawa"}
      return capitals.get(country.lower(), f"Sorry, I don't know the capital of {country}.")

    # Agrega la herramienta al agente
    capital_agent = LlmAgent(
        model="gemini-2.0-flash",
        name="capital_agent", #nombre de tu agente
        description="Answers user questions about the capital city of a given country.",
        instruction="""You are an agent that provides the capital city of a country... (previous instruction text)""",
        tools=[get_capital_city] # Proporciona la función directamente
    )

    # ADK descubrirá la instancia root_agent
    root_agent = capital_agent
    ```
    
    Marca tu directorio como un paquete de python

    ```python title="capital_agent/__init__.py"

    from . import agent
    ```

2. Este archivo configura la aplicación FastAPI usando `get_fast_api_app()` de ADK:

    ```python title="main.py"
    import os

    import uvicorn
    from fastapi import FastAPI
    from google.adk.cli.fast_api import get_fast_api_app

    # Obtiene el directorio donde se encuentra main.py
    AGENT_DIR = os.path.dirname(os.path.abspath(__file__))
    # Ejemplo de URI del servicio de sesión (por ejemplo, SQLite)
    # Nota: Usa 'sqlite+aiosqlite' en lugar de 'sqlite' porque DatabaseSessionService requiere un controlador async
    SESSION_SERVICE_URI = "sqlite+aiosqlite:///./sessions.db"
    # Ejemplo de orígenes permitidos para CORS
    ALLOWED_ORIGINS = ["http://localhost", "http://localhost:8080", "*"]
    # Establece web=True si pretendes servir una interfaz web, False de lo contrario
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
        # Usa la variable de entorno PORT proporcionada por Cloud Run, con valor predeterminado 8080
        uvicorn.run(app, host="0.0.0.0", port=int(os.environ.get("PORT", 8080)))
    ```

    *Nota: Especificamos `agent_dir` al directorio en el que está `main.py` y usamos `os.environ.get("PORT", 8080)` para compatibilidad con Cloud Run.*

3. Lista los paquetes de Python necesarios:

    ```txt title="requirements.txt"
    google-adk
    # Agrega cualquier otra dependencia que tu agente necesite
    ```

4. Define la imagen del contenedor:

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

### Construir la imagen del contenedor

Necesitas crear un repositorio de Google Artifact Registry para almacenar tus imágenes de contenedor. Puedes hacer esto usando la herramienta de línea de comandos `gcloud`.

```bash
gcloud artifacts repositories create adk-repo \
    --repository-format=docker \
    --location=$GOOGLE_CLOUD_LOCATION \
    --description="ADK repository"
```

Construye la imagen del contenedor usando la herramienta de línea de comandos `gcloud`. Este ejemplo construye la imagen y la etiqueta como `adk-repo/adk-agent:latest`.

```bash
gcloud builds submit \
    --tag $GOOGLE_CLOUD_LOCATION-docker.pkg.dev/$GOOGLE_CLOUD_PROJECT/adk-repo/adk-agent:latest \
    --project=$GOOGLE_CLOUD_PROJECT \
    .
```

Verifica que la imagen esté construida y subida al Artifact Registry:

```bash
gcloud artifacts docker images list \
  $GOOGLE_CLOUD_LOCATION-docker.pkg.dev/$GOOGLE_CLOUD_PROJECT/adk-repo \
  --project=$GOOGLE_CLOUD_PROJECT
```

### Configurar la Cuenta de Servicio de Kubernetes para Vertex AI

Si tu agente usa Vertex AI, necesitas crear una cuenta de servicio de Kubernetes con los permisos necesarios. Este ejemplo crea una cuenta de servicio llamada `adk-agent-sa` y la vincula al rol `Vertex AI User`.

> Si estás usando AI Studio y accediendo al modelo con una clave API puedes omitir este paso.

```bash
kubectl create serviceaccount adk-agent-sa
```

```bash
gcloud projects add-iam-policy-binding projects/${GOOGLE_CLOUD_PROJECT} \
    --role=roles/aiplatform.user \
    --member=principal://iam.googleapis.com/projects/${GOOGLE_CLOUD_PROJECT_NUMBER}/locations/global/workloadIdentityPools/${GOOGLE_CLOUD_PROJECT}.svc.id.goog/subject/ns/default/sa/adk-agent-sa \
    --condition=None
```

### Crear los archivos de manifiesto de Kubernetes

Crea un archivo de manifiesto de despliegue de Kubernetes llamado `deployment.yaml` en el directorio de tu proyecto. Este archivo define cómo desplegar tu aplicación en GKE.

```yaml title="deployment.yaml"
cat <<  EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: adk-agent
spec:
  replicas: 1
  selector:
    matchLabels:
      app: adk-agent
  template:
    metadata:
      labels:
        app: adk-agent
    spec:
      serviceAccount: adk-agent-sa
      containers:
      - name: adk-agent
        imagePullPolicy: Always
        image: $GOOGLE_CLOUD_LOCATION-docker.pkg.dev/$GOOGLE_CLOUD_PROJECT/adk-repo/adk-agent:latest
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
            ephemeral-storage: "128Mi"
          requests:
            memory: "128Mi"
            cpu: "500m"
            ephemeral-storage: "128Mi"
        ports:
        - containerPort: 8080
        env:
          - name: PORT
            value: "8080"
          - name: GOOGLE_CLOUD_PROJECT
            value: $GOOGLE_CLOUD_PROJECT
          - name: GOOGLE_CLOUD_LOCATION
            value: $GOOGLE_CLOUD_LOCATION
          - name: GOOGLE_GENAI_USE_VERTEXAI
            value: "$GOOGLE_GENAI_USE_VERTEXAI"
          # Si usas AI Studio, establece GOOGLE_GENAI_USE_VERTEXAI a false y configura lo siguiente:
          # - name: GOOGLE_API_KEY
          #   value: $GOOGLE_API_KEY
          # Agrega cualquier otra variable de entorno necesaria que tu agente pueda necesitar
---
apiVersion: v1
kind: Service
metadata:
  name: adk-agent
spec:       
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: adk-agent
EOF
```

### Desplegar la Aplicación

Despliega la aplicación usando la herramienta de línea de comandos `kubectl`. Este comando aplica los archivos de manifiesto de despliegue y servicio a tu clúster GKE.

```bash
kubectl apply -f deployment.yaml
```

Después de unos momentos, puedes verificar el estado de tu despliegue usando:

```bash
kubectl get pods -l=app=adk-agent
```

Este comando lista los pods asociados con tu despliegue. Deberías ver un pod con un estado de `Running`.

Una vez que el pod esté ejecutándose, puedes verificar el estado del servicio usando:

```bash
kubectl get service adk-agent
```

Si la salida muestra una `External IP`, significa que tu servicio es accesible desde internet. Puede tomar unos minutos para que se asigne la IP externa.

Puedes obtener la dirección IP externa de tu servicio usando:

```bash
kubectl get svc adk-agent -o=jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

## Opción 2: Despliegue Automatizado usando `adk deploy gke`

ADK proporciona un comando CLI para optimizar el despliegue en GKE. Esto evita la necesidad de construir imágenes manualmente, escribir manifiestos de Kubernetes o subir a Artifact Registry.

#### Prerrequisitos

Antes de comenzar, asegúrate de tener lo siguiente configurado:

1. **Un clúster GKE en ejecución:** Necesitas un clúster de Kubernetes activo en Google Cloud.

2. **CLIs Requeridos:** 
    * **CLI `gcloud`:** La CLI de Google Cloud debe estar instalada, autenticada y configurada para usar tu proyecto objetivo. Ejecuta `gcloud auth login` y `gcloud config set project [YOUR_PROJECT_ID]`.
    * **kubectl:** La CLI de Kubernetes debe estar instalada para desplegar la aplicación en tu clúster.

3. **APIs de Google Cloud Habilitadas:** Asegúrate de que las siguientes APIs estén habilitadas en tu proyecto de Google Cloud:
    * API de Kubernetes Engine (`container.googleapis.com`)
    * API de Cloud Build (`cloudbuild.googleapis.com`)
    * API de Container Registry (`containerregistry.googleapis.com`)

4. **Permisos IAM Requeridos:** El usuario o la cuenta de servicio predeterminada de Compute Engine que ejecuta el comando necesita, como mínimo, los siguientes roles:

   * **Desarrollador de Kubernetes Engine** (`roles/container.developer`): Para interactuar con el clúster GKE.

   * **Visualizador de Objetos de Storage** (`roles/storage.objectViewer`): Para permitir que Cloud Build descargue el código fuente del bucket de Cloud Storage donde gcloud builds submit lo sube.

   * **Escritor Create on Push de Artifact Registry** (`roles/artifactregistry.createOnPushWriter`): Para permitir que Cloud Build suba la imagen del contenedor construida a Artifact Registry. Este rol también permite la creación sobre la marcha del repositorio especial gcr.io dentro de Artifact Registry si es necesario en el primer push.

   * **Escritor de Logs** (`roles/logging.logWriter`): Para permitir que Cloud Build escriba logs de construcción en Cloud Logging.

### El Comando `deploy gke`

El comando toma la ruta a tu agente y parámetros que especifican el clúster GKE objetivo.

#### Sintaxis

```bash
adk deploy gke [OPTIONS] AGENT_PATH
```

### Argumentos y Opciones

| Argumento    | Descripción | Requerido |
| -------- | ------- | ------  |
| AGENT_PATH  | La ruta del archivo local al directorio raíz de tu agente.    |Sí |
| --project | El ID del Proyecto de Google Cloud donde se encuentra tu clúster GKE.     | Sí | 
| --cluster_name   | El nombre de tu clúster GKE.    | Sí |
| --region    | La región de Google Cloud de tu clúster (por ejemplo, us-central1).    | Sí |
| --with_ui   | Despliega tanto la API del back-end del agente como una interfaz de usuario de front-end complementaria.    | No |
| --log_level   | Establece el nivel de registro para el proceso de despliegue. Opciones: debug, info, warning, error.     | No |


### Cómo Funciona
Cuando ejecutas el comando `adk deploy gke`, el ADK realiza los siguientes pasos automáticamente:

- Contenerización: Construye una imagen de contenedor Docker desde el código fuente de tu agente.

- Subida de Imagen: Etiqueta la imagen del contenedor y la sube al Artifact Registry de tu proyecto.

- Generación de Manifiesto: Genera dinámicamente los archivos de manifiesto de Kubernetes necesarios (un `Deployment` y un `Service`).

- Despliegue en el Clúster: Aplica estos manifiestos a tu clúster GKE especificado, lo que desencadena lo siguiente:

El `Deployment` instruye a GKE para extraer la imagen del contenedor del Artifact Registry y ejecutarla en uno o más Pods.

El `Service` crea un punto final de red estable para tu agente. Por defecto, este es un servicio LoadBalancer, que proporciona una dirección IP pública para exponer tu agente a internet.


### Ejemplo de Uso
Aquí hay un ejemplo práctico de desplegar un agente ubicado en `~/agents/multi_tool_agent/` a un clúster GKE llamado test.

```bash
adk deploy gke \
    --project myproject \
    --cluster_name test \
    --region us-central1 \
    --with_ui \
    --log_level info \
    ~/agents/multi_tool_agent/
```

### Verificando Tu Despliegue
Si usaste `adk deploy gke`, verifica el despliegue usando `kubectl`:

1. Verifica los Pods: Asegúrate de que los pods de tu agente estén en el estado Running.

```bash
kubectl get pods
```
Deberías ver una salida como `adk-default-service-name-xxxx-xxxx ... 1/1 Running` en el namespace predeterminado.

2. Encuentra la IP Externa: Obtén la dirección IP pública para el servicio de tu agente.

```bash
kubectl get service
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
adk-default-service-name   LoadBalancer   34.118.228.70   34.63.153.253   80:32581/TCP   5d20h
```

Podemos navegar a la IP externa e interactuar con el agente a través de la UI
![alt text](../assets/agent-gke-deployment.png)

## Probando tu agente

Una vez que tu agente esté desplegado en GKE, puedes interactuar con él a través de la UI desplegada (si está habilitada) o directamente con sus endpoints de API usando herramientas como `curl`. Necesitarás la URL del servicio proporcionada después del despliegue.

=== "Pruebas con UI"

    ### Pruebas con UI

    Si desplegaste tu agente con la UI habilitada:

    Puedes probar tu agente simplemente navegando a la URL del servicio de kubernetes en tu navegador web.

    La UI de desarrollo de ADK te permite interactuar con tu agente, gestionar sesiones y ver detalles de ejecución directamente en el navegador.

    Para verificar que tu agente está funcionando según lo previsto, puedes:

    1. Seleccionar tu agente del menú desplegable.
    2. Escribir un mensaje y verificar que recibes una respuesta esperada de tu agente.

    Si experimentas algún comportamiento inesperado, verifica los logs del pod de tu agente usando:

    ```bash
    kubectl logs -l app=adk-agent
    ```

=== "Pruebas con API (curl)"

    ### Pruebas con API (curl)

    Puedes interactuar con los endpoints de la API del agente usando herramientas como `curl`. Esto es útil para interacción programática o si desplegaste sin la UI.

    #### Establecer la URL de la aplicación

    Reemplaza la URL de ejemplo con la URL real de tu servicio Cloud Run desplegado.

    ```bash
    export APP_URL=$(kubectl get service adk-agent -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    ```

    #### Listar aplicaciones disponibles

    Verifica el nombre de la aplicación desplegada.

    ```bash
    curl -X GET $APP_URL/list-apps
    ```

    *(Ajusta el `app_name` en los siguientes comandos según esta salida si es necesario. El predeterminado suele ser el nombre del directorio del agente, por ejemplo, `capital_agent`)*.

    #### Crear o Actualizar una Sesión

    Inicializa o actualiza el estado para un usuario y sesión específicos. Reemplaza `capital_agent` con el nombre real de tu aplicación si es diferente. Los valores `user_123` y `session_abc` son identificadores de ejemplo; puedes reemplazarlos con tus IDs de usuario y sesión deseados.

    ```bash
    curl -X POST \
        $APP_URL/apps/capital_agent/users/user_123/sessions/session_abc \
        -H "Content-Type: application/json" \
        -d '{"preferred_language": "English", "visit_count": 5}'
    ```

    #### Ejecutar el Agente

    Envía un prompt a tu agente. Reemplaza `capital_agent` con el nombre de tu aplicación y ajusta los IDs de usuario/sesión y el prompt según sea necesario.

    ```bash
    curl -X POST $APP_URL/run_sse \
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

    * Establece `"streaming": true` si quieres recibir Eventos Enviados por el Servidor (SSE).
    * La respuesta contendrá los eventos de ejecución del agente, incluida la respuesta final.

## Solución de Problemas

Estos son algunos problemas comunes que podrías encontrar al desplegar tu agente a GKE:

### 403 Permiso Denegado para `Gemini 2.0 Flash`

Esto generalmente significa que la cuenta de servicio de Kubernetes no tiene el permiso necesario para acceder a la API de Vertex AI. Asegúrate de haber creado la cuenta de servicio y la hayas vinculado al rol `Vertex AI User` como se describe en la sección [Configurar la Cuenta de Servicio de Kubernetes para Vertex AI](#configure-kubernetes-service-account-for-vertex-ai). Si estás usando AI Studio, asegúrate de haber establecido la variable de entorno `GOOGLE_API_KEY` en el manifiesto de despliegue y que sea válida.

### Respuesta 404 o No Encontrado

Esto generalmente significa que hay un error en tu solicitud. Verifica los logs de la aplicación para diagnosticar el problema.

```bash

export POD_NAME=$(kubectl get pod -l app=adk-agent -o jsonpath='{.items[0].metadata.name}')
kubectl logs $POD_NAME
```

### Intento de escribir en una base de datos de solo lectura

Podrías ver que no se crea un ID de sesión en la UI y el agente no responde a ningún mensaje. Esto generalmente es causado por que la base de datos SQLite sea de solo lectura. Esto puede suceder si ejecutas el agente localmente y luego creas la imagen del contenedor que copia la base de datos SQLite en el contenedor. La base de datos entonces es de solo lectura en el contenedor.

```bash
sqlalchemy.exc.OperationalError: (sqlite3.OperationalError) attempt to write a readonly database
[SQL: UPDATE app_states SET state=?, update_time=CURRENT_TIMESTAMP WHERE app_states.app_name = ?]
```

Para solucionar este problema, puedes:

Eliminar el archivo de base de datos SQLite de tu máquina local antes de construir la imagen del contenedor. Esto creará una nueva base de datos SQLite cuando se inicie el contenedor.

```bash
rm -f sessions.db
```

o (recomendado) puedes agregar un archivo `.dockerignore` al directorio de tu proyecto para excluir la base de datos SQLite de ser copiada en la imagen del contenedor.

```txt title=".dockerignore"
sessions.db
```

Construye la imagen del contenedor y despliega la aplicación nuevamente.

### Permiso Insuficiente para Transmitir Logs `ERROR: (gcloud.builds.submit)`

Este error puede ocurrir cuando no tienes permisos suficientes para transmitir logs de construcción, o tu política de seguridad VPC-SC restringe el acceso al bucket de logs predeterminado.

Para verificar el progreso de la construcción, sigue el enlace proporcionado en el mensaje de error o navega a la página de Cloud Build en la consola de Google Cloud.

También puedes verificar que la imagen fue construida y subida al Artifact Registry usando el comando bajo la sección [Construir la imagen del contenedor](#build-the-container-image).

### Gemini-2.0-Flash No Soportado en Live Api

Cuando usas la UI de Desarrollo de ADK para tu agente desplegado, el chat basado en texto funciona, pero la voz (por ejemplo, hacer clic en el botón del micrófono) falla. Podrías ver un `websockets.exceptions.ConnectionClosedError` en los logs del pod indicando que tu modelo "no está soportado en la live api".

Este error ocurre porque el agente está configurado con un modelo (como `gemini-2.0-flash` en el ejemplo) que no soporta la API Live de Gemini. La API Live es requerida para transmisión bidireccional en tiempo real de audio y video.

## Limpieza

Para eliminar el clúster GKE y todos los recursos asociados, ejecuta:

```bash
gcloud container clusters delete adk-cluster \
    --location=$GOOGLE_CLOUD_LOCATION \
    --project=$GOOGLE_CLOUD_PROJECT
```

Para eliminar el repositorio de Artifact Registry, ejecuta:

```bash
gcloud artifacts repositories delete adk-repo \
    --location=$GOOGLE_CLOUD_LOCATION \
    --project=$GOOGLE_CLOUD_PROJECT
```

También puedes eliminar el proyecto si ya no lo necesitas. Esto eliminará todos los recursos asociados con el proyecto, incluido el clúster GKE, el repositorio de Artifact Registry y cualquier otro recurso que hayas creado.

```bash
gcloud projects delete $GOOGLE_CLOUD_PROJECT
```