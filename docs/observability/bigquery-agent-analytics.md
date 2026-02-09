# Plugin de Analíticas de Agentes BigQuery

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v1.21.0</span><span class="lst-preview">Preview</span>
</div>

!!! important "Requisito de Versión"

   Utiliza la ***última versión*** del ADK (versión 1.21.0 o superior) para aprovechar al máximo las funcionalidades descritas en este documento.

El Plugin de Analíticas de Agentes BigQuery mejora significativamente el Kit de Desarrollo de Agentes (ADK) al proporcionar una solución robusta para el análisis en profundidad del comportamiento de agentes. Utilizando la arquitectura de Plugins del ADK y la **API de Escritura de Almacenamiento de BigQuery**, captura y registra eventos operacionales críticos directamente en una tabla de Google BigQuery, dotándote de capacidades avanzadas para depuración, monitoreo en tiempo real y evaluación integral del rendimiento offline.

La versión 1.21.0 introduce **Registro Multimodal Híbrido**, permitiéndote registrar cargas grandes (imágenes, audio, blobs) descargándolas a Google Cloud Storage (GCS) mientras mantienes una referencia estructurada (`ObjectRef`) en BigQuery.

!!! example "Versión Preview"

    El Plugin de Analíticas de Agentes BigQuery está en versión Preview. Para más
    información, consulta las
    [descripciones de etapas de lanzamiento](https://cloud.google.com/products#product-launch-stages).

!!! warning "API de Escritura de Almacenamiento de BigQuery"

    Esta funcionalidad utiliza la **API de Escritura de Almacenamiento de BigQuery**, que es un servicio de pago.
    Para información sobre costos, consulta la
    [documentación de BigQuery](https://cloud.google.com/bigquery/pricing?e=48754805&hl=en#data-ingestion-pricing).

## Casos de uso

-   **Depuración y análisis de flujos de trabajo de agentes:** Captura una amplia gama de
    *eventos del ciclo de vida de plugins* (llamadas a LLM, uso de herramientas) y *eventos
    generados por agentes* (entrada de usuario, respuestas del modelo), en un esquema bien definido.
-   **Análisis y depuración de alto volumen:** Las operaciones de registro se realizan
    de forma asíncrona utilizando la API de Escritura de Almacenamiento para permitir alto rendimiento y baja latencia.
-   **Análisis Multimodal**: Registra y analiza texto, imágenes y otras modalidades. Los archivos grandes se descargan a GCS, haciéndolos accesibles a BigQuery ML a través de Tablas de Objetos.
-   **Rastreo Distribuido**: Soporte integrado para rastreo estilo OpenTelemetry (`trace_id`, `span_id`) para visualizar flujos de ejecución de agentes.

Los datos de eventos del agente registrados varían según el tipo de evento del ADK. Para más
información, consulta [Tipos de eventos y cargas](#event-types).

## Requisitos previos

-   **Proyecto de Google Cloud** con la **API de BigQuery** habilitada.
-   **Dataset de BigQuery:** Crea un dataset para almacenar tablas de registro antes
    de usar el plugin. El plugin crea automáticamente la tabla de eventos necesaria dentro del dataset si la tabla no existe.
-   **Bucket de Google Cloud Storage (Opcional):** Si planeas registrar contenido multimodal (imágenes, audio, etc.), se recomienda crear un bucket de GCS para descargar archivos grandes.
-   **Autenticación:**
    -   **Local:** Ejecuta `gcloud auth application-default login`.
    -   **Nube:** Asegúrate de que tu cuenta de servicio tenga los permisos requeridos.

### Permisos IAM

Para que el agente funcione correctamente, el principal (por ejemplo, cuenta de servicio, cuenta de usuario) bajo el cual se ejecuta el agente necesita estos roles de Google Cloud:
* `roles/bigquery.jobUser` a Nivel de Proyecto para ejecutar consultas BigQuery.
* `roles/bigquery.dataEditor` a Nivel de Tabla para escribir datos de registro/evento.
* **Si usas descarga a GCS:** `roles/storage.objectCreator` y `roles/storage.objectViewer` en el bucket de destino.

## Uso con agente

Utilizas el Plugin de Analíticas de Agentes BigQuery configurándolo y registrándolo con
el objeto App del agente de tu ADK. El siguiente ejemplo muestra una implementación de un
agente con este plugin, incluyendo descarga a GCS:

```python title="my_bq_agent/agent.py"
# my_bq_agent/agent.py
import os
import google.auth
from google.adk.apps import App
from google.adk.plugins.bigquery_agent_analytics_plugin import BigQueryAgentAnalyticsPlugin, BigQueryLoggerConfig
from google.adk.agents import Agent
from google.adk.models.google_llm import Gemini
from google.adk.tools.bigquery import BigQueryToolset, BigQueryCredentialsConfig


# --- Inicialización de OpenTelemetry (Opcional) ---
# Recomendado para habilitar rastreo distribuido (puebla trace_id, span_id).
# Si no está configurado, el plugin usa UUIDs internos para correlación de spans.
try:
    from opentelemetry import trace
    from opentelemetry.sdk.trace import TracerProvider
    trace.set_tracer_provider(TracerProvider())
except ImportError:
    pass # OpenTelemetry es opcional

# --- Configuración ---
PROJECT_ID = os.environ.get("GOOGLE_CLOUD_PROJECT", "your-gcp-project-id")
DATASET_ID = os.environ.get("BIG_QUERY_DATASET_ID", "your-big-query-dataset-id")
LOCATION = os.environ.get("GOOGLE_CLOUD_LOCATION", "US") # la ubicación predeterminada es US en el plugin
GCS_BUCKET = os.environ.get("GCS_BUCKET_NAME", "your-gcs-bucket-name") # Opcional

if PROJECT_ID == "your-gcp-project-id":
    raise ValueError("Please set GOOGLE_CLOUD_PROJECT or update the code.")

# --- CRÍTICO: Establece variables de entorno ANTES de la instanciación de Gemini ---
os.environ['GOOGLE_CLOUD_PROJECT'] = PROJECT_ID
os.environ['GOOGLE_CLOUD_LOCATION'] = LOCATION
os.environ['GOOGLE_GENAI_USE_VERTEXAI'] = 'True'

# --- Inicializa el Plugin con Configuración ---
bq_config = BigQueryLoggerConfig(
    enabled=True,
    gcs_bucket_name=GCS_BUCKET, # Habilita descarga a GCS para contenido multimodal
    log_multi_modal_content=True,
    max_content_length=500 * 1024, # Límite de 500 KB para texto inline
    batch_size=1, # El predeterminado es 1 para baja latencia, aumenta para alto rendimiento
    shutdown_timeout=10.0
)

bq_logging_plugin = BigQueryAgentAnalyticsPlugin(
    project_id=PROJECT_ID,
    dataset_id=DATASET_ID,
    table_id="agent_events_v2", # el nombre de tabla predeterminado es agent_events_v2
    config=bq_config,
    location=LOCATION
)

# --- Inicializa Herramientas y Modelo ---
credentials, _ = google.auth.default(scopes=["https://www.googleapis.com/auth/cloud-platform"])
bigquery_toolset = BigQueryToolset(
    credentials_config=BigQueryCredentialsConfig(credentials=credentials)
)

llm = Gemini(model="gemini-2.5-flash")

root_agent = Agent(
    model=llm,
    name='my_bq_agent',
    instruction="You are a helpful assistant with access to BigQuery tools.",
    tools=[bigquery_toolset]
)

# --- Crea la App ---
app = App(
    name="my_bq_agent",
    root_agent=root_agent,
    plugins=[bq_logging_plugin],
)
```

### Ejecutar y probar agente

Prueba el plugin ejecutando el agente y realizando algunas solicitudes a través de la
interfaz de chat, como "tell me what you can do" o "List datasets in my cloud project <your-gcp-project-id>". Estas acciones crean eventos que se
registran en tu instancia de BigQuery del proyecto de Google Cloud. Una vez que estos eventos han
sido procesados, puedes ver los datos en la [Consola de BigQuery](https://console.cloud.google.com/bigquery), usando esta consulta

```sql
SELECT timestamp, event_type, content 
FROM `your-gcp-project-id.your-big-query-dataset-id.agent_events_v2`
ORDER BY timestamp DESC
LIMIT 20;
```

```

## Rastreo y Observabilidad

El plugin soporta **OpenTelemetry** para rastreo distribuido.

- **Gestión Automática de Spans**: El plugin genera automáticamente spans para la ejecución del Agente, llamadas a LLM y ejecuciones de Herramientas.
- **Integración con OpenTelemetry**: Si un `TracerProvider` de OpenTelemetry está configurado (como se muestra en el ejemplo anterior), el plugin usará spans válidos de OTel, poblando `trace_id`, `span_id` y `parent_span_id` con identificadores estándar de OTel. Esto te permite correlacionar registros de agentes con otros servicios en tu sistema distribuido.
- **Mecanismo de Respaldo**: Si OpenTelemetry no está instalado o configurado, el plugin automáticamente recurre a generar UUIDs internos para spans y usa el `invocation_id` como ID de rastreo. Esto asegura que la jerarquía padre-hijo (Agente -> Span -> Herramienta/LLM) esté *siempre* preservada en los registros de BigQuery, incluso sin una configuración completa de OTel.

## Opciones de configuración

Puedes personalizar el plugin usando `BigQueryLoggerConfig`.

-   **`enabled`** (`bool`, predeterminado: `True`): Para deshabilitar el registro de datos del agente del plugin a la tabla de BigQuery, establece este parámetro en False.
-   **`clustering_fields`** (`List[str]`, predeterminado: `["event_type", "agent", "user_id"]`): Los campos utilizados para agrupar la tabla de BigQuery cuando se crea automáticamente.
-   **`gcs_bucket_name`** (`Optional[str]`, predeterminado: `None`): El nombre del bucket de GCS para descargar contenido grande (imágenes, blobs, texto grande). Si no se proporciona, el contenido grande puede ser truncado o reemplazado con marcadores de posición.
-   **`connection_id`** (`Optional[str]`, predeterminado: `None`): El ID de conexión de BigQuery (por ejemplo, `us.my-connection`) a usar como autorizador para columnas `ObjectRef`. Requerido para usar `ObjectRef` con BigQuery ML.
-   **`max_content_length`** (`int`, predeterminado: `500 * 1024`): La longitud máxima (en caracteres) de contenido de texto para almacenar **inline** en BigQuery antes de descargar a GCS (si está configurado) o truncar. El predeterminado es 500 KB.
-   **`batch_size`** (`int`, predeterminado: `1`): El número de eventos a agrupar antes de escribir a BigQuery.
-   **`batch_flush_interval`** (`float`, predeterminado: `1.0`): El tiempo máximo (en segundos) a esperar antes de vaciar un lote parcial.
-   **`shutdown_timeout`** (`float`, predeterminado: `10.0`): Segundos a esperar para que los registros se vacíen durante el apagado.
-   **`event_allowlist`** (`Optional[List[str]]`, predeterminado: `None`): Una lista
    de tipos de eventos a registrar. Si es `None`, todos los eventos se registran excepto aquellos en
    `event_denylist`. Para una lista completa de tipos de eventos soportados, consulta
    la sección [Tipos de eventos y cargas](#event-types).
-   **`event_denylist`** (`Optional[List[str]]`, predeterminado: `None`): Una lista de
    tipos de eventos a omitir en el registro. Para una lista completa de tipos de eventos soportados,
    consulta la sección [Tipos de eventos y cargas](#event-types).
-   **`content_formatter`** (`Optional[Callable[[Any, str], Any]]`, predeterminado: `None`): Una función opcional para formatear el contenido del evento antes del registro.
-   **`log_multi_modal_content`** (`bool`, predeterminado: `True`): Si registrar partes de contenido detalladas (incluyendo referencias a GCS).
-   **`queue_max_size`** (`int`, predeterminado: `10000`): El número máximo de eventos a mantener en la cola en memoria antes de descartar nuevos eventos.
-   **`retry_config`** (`RetryConfig`, predeterminado: `RetryConfig()`): Configuración para reintentar escrituras fallidas a BigQuery (atributos: `max_retries`, `initial_delay`, `multiplier`, `max_delay`).


El siguiente ejemplo de código muestra cómo definir una configuración para el
plugin de Analíticas de Agentes BigQuery:

```python
import json
import re

from google.adk.plugins.bigquery_agent_analytics_plugin import BigQueryLoggerConfig

def redact_dollar_amounts(event_content: Any) -> str:
    """
    Formateador personalizado para redactar cantidades en dólares (por ejemplo, $600, $12.50)
    y asegurar salida JSON si la entrada es un dict.
    """
    text_content = ""
    if isinstance(event_content, dict):
        text_content = json.dumps(event_content)
    else:
        text_content = str(event_content)

    # Regex para encontrar cantidades en dólares: $ seguido de dígitos, opcionalmente con comas o decimales.
    # Ejemplos: $600, $1,200.50, $0.99
    redacted_content = re.sub(r'\$\d+(?:,\d{3})*(?:\.\d+)?', 'xxx', text_content)

    return redacted_content

config = BigQueryLoggerConfig(
    enabled=True,
    event_allowlist=["LLM_REQUEST", "LLM_RESPONSE"], # Solo registra estos eventos
    # event_denylist=["TOOL_STARTING"], # Omite estos eventos
    shutdown_timeout=10.0, # Espera hasta 10s para que los registros se vacíen al salir
    client_close_timeout=2.0, # Espera hasta 2s para que el cliente BQ se cierre
    max_content_length=500, # Trunca contenido a 500 caracteres
    content_formatter=redact_dollar_amounts, # Redacta las cantidades en dólares en el contenido de registro
    queue_max_size=10000, # Máximo de eventos a mantener en memoria
    # retry_config=RetryConfig(max_retries=3), # Opcional: Configura reintentos
)

plugin = BigQueryAgentAnalyticsPlugin(..., config=config)
```


## Esquema y configuración de producción

El plugin crea automáticamente la tabla si no existe. Sin embargo, para
producción, recomendamos crear la tabla manualmente usando el siguiente DDL, que utiliza el tipo **JSON** para flexibilidad y **REPEATED RECORD**s para contenido multimodal.

**DDL Recomendado:**

```sql
CREATE TABLE `your-gcp-project-id.adk_agent_logs.agent_events_v2`
(
  timestamp TIMESTAMP NOT NULL OPTIONS(description="The UTC time at which the event was logged."),
  event_type STRING OPTIONS(description="Indicates the type of event being logged (e.g., 'LLM_REQUEST', 'TOOL_COMPLETED')."),
  agent STRING OPTIONS(description="The name of the ADK agent or author associated with the event."),
  session_id STRING OPTIONS(description="A unique identifier to group events within a single conversation or user session."),
  invocation_id STRING OPTIONS(description="A unique identifier for each individual agent execution or turn within a session."),
  user_id STRING OPTIONS(description="The identifier of the user associated with the current session."),
  trace_id STRING OPTIONS(description="OpenTelemetry trace ID for distributed tracing."),
  span_id STRING OPTIONS(description="OpenTelemetry span ID for this specific operation."),
  parent_span_id STRING OPTIONS(description="OpenTelemetry parent span ID to reconstruct hierarchy."),
  content JSON OPTIONS(description="The event-specific data (payload) stored as JSON."),
  content_parts ARRAY<STRUCT<
    mime_type STRING,
    uri STRING,
    object_ref STRUCT<
      uri STRING,
      version STRING,
      authorizer STRING,
      details JSON
    >,
    text STRING,
    part_index INT64,
    part_attributes STRING,
    storage_mode STRING
  >> OPTIONS(description="Detailed content parts for multi-modal data."),
  attributes JSON OPTIONS(description="Arbitrary key-value pairs for additional metadata (e.g., 'root_agent_name', 'model_version', 'usage_metadata')."),
  latency_ms JSON OPTIONS(description="Latency measurements (e.g., total_ms)."),
  status STRING OPTIONS(description="The outcome of the event, typically 'OK' or 'ERROR'."),
  error_message STRING OPTIONS(description="Populated if an error occurs."),
  is_truncated BOOLEAN OPTIONS(description="Flag indicates if content was truncated.")
)
PARTITION BY DATE(timestamp)
CLUSTER BY event_type, agent, user_id;
```

### Tipos de eventos y cargas {#event-types}

La columna `content` ahora contiene un objeto **JSON** específico para el `event_type`.
La columna `content_parts` proporciona una vista estructurada del contenido, especialmente útil para imágenes o datos descargados.

!!! note "Truncamiento de Contenido"

    - Los campos de contenido variable se truncan a `max_content_length` (configurado en `BigQueryLoggerConfig`, predeterminado 500KB).
    - Si `gcs_bucket_name` está configurado, el contenido grande se descarga a GCS en lugar de truncarse, y se almacena una referencia en `content_parts.object_ref`.

#### Interacciones con LLM (ciclo de vida del plugin)

Estos eventos rastrean las solicitudes sin procesar enviadas a y las respuestas recibidas del
LLM.

<table>
  <thead>
    <tr>
      <th><strong>Tipo de Evento</strong></th>
      <th><strong>Estructura de Content (JSON)</strong></th>
      <th><strong>Attributes (JSON)</strong></th>
      <th><strong>Ejemplo de Content (Simplificado)</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><pre>LLM_REQUEST</pre></p></td>
      <td><p><pre>
{
  "prompt": [
    {"role": "user", "content": "..."}
  ],
  "system_prompt": "..."
}
</pre></p></td>
      <td><p><pre>
{
  "tools": ["tool_a", "tool_b"],
  "llm_config": {"temperature": 0.5},
  "root_agent_name": "my_root_agent"
}
</pre></p></td>
      <td><p><pre>
{
  "prompt": [
    {"role": "user", "content": "What is the capital of France?"}
  ],
  "system_prompt": "You are a helpful geography assistant."
}
</pre></p></td>
    </tr>
    <tr>
      <td><p><pre>LLM_RESPONSE</pre></p></td>
      <td><p><pre>
{
  "response": "...",
  "usage": {...}
}
</pre></p></td>
      <td><p><pre>
{
  "model_version": "gemini-2.5-pro-001",
  "usage_metadata": {
    "prompt_token_count": 15,
    "candidates_token_count": 7,
    "total_token_count": 22
  }
}
</pre></p></td>
      <td><p><pre>
{
  "response": "The capital of France is Paris.",
  "usage": {
    "prompt": 15,
    "completion": 7,
    "total": 22
  }
}
</pre></p></td>
    </tr>
    <tr>
      <td><p><pre>LLM_ERROR</pre></p></td>
      <td><p><pre>null</pre></p></td>
      <td><p><pre>{}</pre></p></td>
      <td><p><pre>null (Ver columna error_message)</pre></p></td>
    </tr>
  </tbody>
</table>

#### Uso de herramientas (ciclo de vida del plugin)

Estos eventos rastrean la ejecución de herramientas por el agente.

<table>
  <thead>
    <tr>
      <th><strong>Tipo de Evento</strong></th>
      <th><strong>Estructura de Content (JSON)</strong></th>
      <th><strong>Attributes (JSON)</strong></th>
      <th><strong>Ejemplo de Content</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><pre>TOOL_STARTING</pre></p></td>
      <td><p><pre>
{
  "tool": "...",
  "args": {...}
}
</pre></p></td>
      <td><p><pre>{}</pre></p></td>
      <td><p><pre>
{"tool": "list_datasets", "args": {"project_id": "my-project"}}
</pre></p></td>
    </tr>
    <tr>
      <td><p><pre>TOOL_COMPLETED</pre></p></td>
      <td><p><pre>
{
  "tool": "...",
  "result": "..."
}
</pre></p></td>
      <td><p><pre>{}</pre></p></td>
      <td><p><pre>
{"tool": "list_datasets", "result": ["ds1", "ds2"]}
</pre></p></td>
    </tr>
    <tr>
      <td><p><pre>TOOL_ERROR</pre></p></td>
      <td><p><pre>
{
  "tool": "...",
  "args": {...}
}
</pre></p></td>
      <td><p><pre>{}</pre></p></td>
      <td><p><pre>
{"tool": "list_datasets", "args": {}}
</pre></p></td>
    </tr>
  </tbody>
</table>

#### Ciclo de vida del Agente y Eventos Genéricos

<table>
  <thead>
    <tr>
      <th><strong>Tipo de Evento</strong></th>
      <th><strong>Estructura de Content (JSON)</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><pre>INVOCATION_STARTING</pre></p></td>
      <td><p><pre>{}</pre></p></td>
    </tr>
    <tr>
      <td><p><pre>INVOCATION_COMPLETED</pre></p></td>
      <td><p><pre>{}</pre></p></td>
    </tr>
    <tr>
      <td><p><pre>AGENT_STARTING</pre></p></td>
      <td><p><pre>"You are a helpful agent..."</pre></p></td>
    </tr>
    <tr>
      <td><p><pre>AGENT_COMPLETED</pre></p></td>
      <td><p><pre>{}</pre></p></td>
    </tr>
    <tr>
      <td><p><pre>USER_MESSAGE_RECEIVED</pre></p></td>
      <td><p><pre>{"text_summary": "Help me book a flight."}</pre></p></td>
    </tr>

  </tbody>
</table>

#### Ejemplos de Descarga a GCS (Multimodal y Texto Grande)

Cuando `gcs_bucket_name` está configurado, el texto grande y el contenido multimodal (imágenes, audio, etc.) se descargan automáticamente a GCS. La columna `content` contendrá un resumen o marcador de posición, mientras que `content_parts` contiene el `object_ref` que apunta al URI de GCS.

**Ejemplo de Texto Descargado**

```json
{
  "event_type": "LLM_REQUEST",
  "content_parts": [
    {
      "part_index": 1,
      "mime_type": "text/plain",
      "storage_mode": "GCS_REFERENCE",
      "text": "AAAA... [OFFLOADED]",
      "object_ref": {
        "uri": "gs://haiyuan-adk-debug-verification-1765319132/2025-12-10/e-f9545d6d/ae5235e6_p1.txt",
        "authorizer": "us.bqml_connection",
        "details": {"gcs_metadata": {"content_type": "text/plain"}}
      }
    }
  ]
}
```

**Ejemplo de Imagen Descargada**

```json
{
  "event_type": "LLM_REQUEST",
  "content_parts": [
    {
      "part_index": 2,
      "mime_type": "image/png",
      "storage_mode": "GCS_REFERENCE",
      "text": "[MEDIA OFFLOADED]",
      "object_ref": {
        "uri": "gs://haiyuan-adk-debug-verification-1765319132/2025-12-10/e-f9545d6d/ae5235e6_p2.png",
        "authorizer": "us.bqml_connection",
        "details": {"gcs_metadata": {"content_type": "image/png"}}
      }
    }
  ]
}
```

**Consultando Contenido Descargado (Obtener URLs Firmadas)**

```sql
SELECT
  timestamp,
  event_type,
  part.mime_type,
  part.storage_mode,
  part.object_ref.uri AS gcs_uri,
  -- Genera una URL firmada para leer el contenido directamente (requiere configuración de connection_id)
  STRING(OBJ.GET_ACCESS_URL(part.object_ref, 'r').access_urls.read_url) AS signed_url
FROM `your-gcp-project-id.your-dataset-id.agent_events_v2`,
UNNEST(content_parts) AS part
WHERE part.storage_mode = 'GCS_REFERENCE'
ORDER BY timestamp DESC
LIMIT 10;
```

## Consultas de análisis avanzadas

**Rastrear un turno específico de conversación usando trace_id**

```sql
SELECT timestamp, event_type, agent, JSON_VALUE(content, '$.response') as summary
FROM `your-gcp-project-id.your-dataset-id.agent_events_v2`
WHERE trace_id = 'your-trace-id'
ORDER BY timestamp ASC;
```

**Análisis de uso de tokens (accediendo a campos JSON)**

```sql
SELECT
  AVG(CAST(JSON_VALUE(content, '$.usage.total') AS INT64)) as avg_tokens
FROM `your-gcp-project-id.your-dataset-id.agent_events_v2`
WHERE event_type = 'LLM_RESPONSE';
```

**Consultando Contenido Multimodal (usando content_parts y ObjectRef)**

```sql
SELECT
  timestamp,
  part.mime_type,
  part.object_ref.uri as gcs_uri
FROM `your-gcp-project-id.your-dataset-id.agent_events_v2`,
UNNEST(content_parts) as part
WHERE part.mime_type LIKE 'image/%'
ORDER BY timestamp DESC;
```

**Analizar Contenido Multimodal con Modelo Remoto de BigQuery (Gemini)**

```sql
SELECT
  logs.session_id,
  -- Obtiene una URL firmada para la imagen
  STRING(OBJ.GET_ACCESS_URL(parts.object_ref, "r").access_urls.read_url) as signed_url,
  -- Analiza la imagen usando un modelo remoto (por ejemplo, gemini-pro-vision)
  AI.GENERATE(
    ('Describe this image briefly. What company logo?', parts.object_ref)
  ) AS generated_result
FROM
  `your-gcp-project-id.your-dataset-id.agent_events_v2` logs,
  UNNEST(logs.content_parts) AS parts
WHERE
  parts.mime_type LIKE 'image/%'
ORDER BY logs.timestamp DESC
LIMIT 1;
```

**Análisis de Latencia (LLM y Herramientas)**

```sql
SELECT
  event_type,
  AVG(CAST(JSON_VALUE(latency_ms, '$.total_ms') AS INT64)) as avg_latency_ms
FROM `your-gcp-project-id.your-dataset-id.agent_events_v2`
WHERE event_type IN ('LLM_RESPONSE', 'TOOL_COMPLETED')
GROUP BY event_type;
```

**Análisis de Jerarquía y Duración de Spans**

```sql
SELECT
  span_id,
  parent_span_id,
  event_type,
  timestamp,
  -- Extrae duración de latency_ms para operaciones completadas
  CAST(JSON_VALUE(latency_ms, '$.total_ms') AS INT64) as duration_ms,
  -- Identifica la herramienta u operación específica
  COALESCE(
    JSON_VALUE(content, '$.tool'), 
    'LLM_CALL'
  ) as operation
FROM `your-gcp-project-id.your-dataset-id.agent_events_v2`
WHERE trace_id = 'your-trace-id'
  AND event_type IN ('LLM_RESPONSE', 'TOOL_COMPLETED')
ORDER BY timestamp ASC;
```


### 7. Análisis de Causa Raíz Potenciado por IA (Agent Ops)

Analiza automáticamente sesiones fallidas para determinar la causa raíz de errores usando BigQuery ML y Gemini.

```sql
DECLARE failed_session_id STRING;
-- Encuentra una sesión fallida reciente
SET failed_session_id = (
    SELECT session_id
    FROM `your-gcp-project-id.your-dataset-id.agent_events_v2`
    WHERE error_message IS NOT NULL
    ORDER BY timestamp DESC
    LIMIT 1
);

-- Reconstruye el contexto completo de la conversación
WITH SessionContext AS (
    SELECT
        session_id,
        STRING_AGG(CONCAT(event_type, ': ', COALESCE(TO_JSON_STRING(content), '')), '\n' ORDER BY timestamp) as full_history
    FROM `your-gcp-project-id.your-dataset-id.agent_events_v2`
    WHERE session_id = failed_session_id
    GROUP BY session_id
)
-- Pide a Gemini que diagnostique el problema
SELECT
    session_id,
    AI.GENERATE(
        ('Analyze this conversation log and explain the root cause of the failure. Log: ', full_history),
        connection_id => 'your-gcp-project-id.us.my-connection',
        endpoint => 'gemini-2.5-flash'
    ).result AS root_cause_explanation
FROM SessionContext;
```


## Analíticas Conversacionales en BigQuery

También puedes usar 
[Analíticas Conversacionales de BigQuery](https://cloud.google.com/bigquery/docs/conversational-analytics)
para analizar tus registros de agente usando lenguaje natural. Usa esta herramienta para responder preguntas como:

*   "Muéstrame la tasa de error a lo largo del tiempo"
*   "¿Cuáles son las llamadas a herramientas más comunes?"
*   "Identifica sesiones con alto uso de tokens"

## Panel de Looker Studio

Puedes visualizar el rendimiento de tu agente usando nuestra [plantilla de Panel de Looker Studio](https://lookerstudio.google.com/c/reporting/f1c5b513-3095-44f8-90a2-54953d41b125/page/8YdhF) predefinida.

Para conectar este panel a tu propia tabla de BigQuery, usa el siguiente formato de enlace, reemplazando los marcadores de posición con tu proyecto, dataset e IDs de tabla específicos:

```text
https://lookerstudio.google.com/reporting/create?c.reportId=f1c5b513-3095-44f8-90a2-54953d41b125&ds.ds3.connector=bigQuery&ds.ds3.type=TABLE&ds.ds3.projectId=<your-project-id>&ds.ds3.datasetId=<your-dataset-id>&ds.ds3.tableId=<your-table-id>
```

## Recursos adicionales

-   [API de Escritura de Almacenamiento de BigQuery](https://cloud.google.com/bigquery/docs/write-api)
-   [Introducción a Tablas de Objetos](https://cloud.google.com/bigquery/docs/object-tables-intro)
-   [Notebook de Demostración Interactivo](https://github.com/haiyuan-eng-google/demo_BQ_agent_analytics_plugin_notebook)