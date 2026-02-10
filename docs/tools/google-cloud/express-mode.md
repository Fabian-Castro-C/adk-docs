---
catalog_title: Vertex AI express mode
catalog_description: Try development with Vertex AI services at no cost
catalog_icon: /assets/tools-vertex-ai.png
---

# Modo express de Vertex AI

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-java">Java v0.1.0</span><span class="lst-preview">Vista previa</span>
</div>

El modo express de Vertex AI de Google Cloud proporciona un nivel de acceso sin costo para
prototipado y desarrollo, permitiéndote usar los servicios de Vertex AI sin
crear un Proyecto de Google Cloud completo. Este servicio incluye acceso a muchos
servicios potentes de Vertex AI, incluyendo:

- [Vertex AI SessionService](#vertex-ai-session-service)
- [Vertex AI MemoryBankService](#vertex-ai-memory-bank)

Puedes registrarte para obtener una cuenta de modo express usando una cuenta de Gmail y recibir una
clave de API para usar con el ADK. Obtén una clave de API a través de la
[Consola de Google Cloud](https://console.cloud.google.com/expressmode).
Para más información, consulta
[Modo express de Vertex AI](https://cloud.google.com/vertex-ai/generative-ai/docs/start/express-mode/overview).

!!! example "Lanzamiento en vista previa"
    La característica del modo express de Vertex AI es un lanzamiento en Vista previa. Para
    más información, consulta las
    [descripciones de etapas de lanzamiento](https://cloud.google.com/products#product-launch-stages).

??? info "Limitaciones del modo express de Vertex AI"

    Los proyectos del modo express de Vertex AI solo son válidos por 90 días y solo servicios
    selectos están disponibles para ser usados con cuota limitada. Por ejemplo, el número de
    Agent Engines está restringido a 10 y el despliegue a Agent Engine requiere acceso
    de pago. Para eliminar las restricciones de cuota y usar todos los servicios de Vertex AI,
    agrega una cuenta de facturación a tu proyecto de modo express.

## Configurar contenedor de Agent Engine

Al usar el modo express de Vertex AI, crea un objeto `AgentEngine` para habilitar
la gestión de Vertex AI de componentes del agente como objetos `Session` y `Memory`.
Con este enfoque, los objetos `Session` se manejan como hijos del
objeto `AgentEngine`. Antes de ejecutar tu agente asegúrate de que tus variables de
entorno estén configuradas correctamente, como se muestra a continuación:

```env title="agent/.env"
GOOGLE_GENAI_USE_VERTEXAI=TRUE
GOOGLE_API_KEY=PASTE_YOUR_ACTUAL_EXPRESS_MODE_API_KEY_HERE
```

A continuación, crea tu instancia de Agent Engine usando el SDK de Vertex AI.

1. Importa el SDK de Vertex AI.

    ```py
    import vertexai
    from vertexai import agent_engines
    ```

2. Inicializa el Cliente de Vertex AI con tu clave de API y crea una instancia de agent engine.

    ```py
    # Crear Agent Engine con Gen AI SDK
    client = vertexai.Client(
      api_key="YOUR_API_KEY",
    )

    agent_engine = client.agent_engines.create(
      config={
        "display_name": "Demo Agent Engine",
        "description": "Agent Engine for Session and Memory",
      })
    ```

3. Obtén el nombre e ID del Agent Engine de la respuesta para usar con Memories y Sessions.

    ```py
    APP_ID = agent_engine.api_resource.name.split('/')[-1]
    ```

## Gestionar Sesiones con `VertexAiSessionService` {#vertex-ai-session-service}

[`VertexAiSessionService`](/sessions/session.md#sessionservice-implementations)
es compatible con las claves de API del modo express de Vertex AI. En su lugar puedes inicializar
el objeto de sesión sin ningún proyecto o ubicación.

```py
# Requiere: pip install google-adk[vertexai]
# Más configuración de variable de entorno:
# GOOGLE_GENAI_USE_VERTEXAI=TRUE
# GOOGLE_API_KEY=PASTE_YOUR_ACTUAL_EXPRESS_MODE_API_KEY_HERE
from google.adk.sessions import VertexAiSessionService

# El app_name usado con este servicio debe ser el ID o nombre del Reasoning Engine
APP_ID = "your-reasoning-engine-id"

# Project y location no son requeridos al inicializar con el modo express de Vertex
session_service = VertexAiSessionService(agent_engine_id=APP_ID)
# Usa REASONING_ENGINE_APP_ID al llamar a los métodos del servicio, ej.:
# session = await session_service.create_session(app_name=APP_ID, user_id= ...)
```

!!! info "Cuotas del Session Service"

    Para Proyectos de modo express gratuitos, `VertexAiSessionService` tiene la siguiente cuota:

    - 10 Crear, eliminar o actualizar sesiones de Vertex AI Agent Engine por minuto
    - 30 Agregar evento a sesiones de Vertex AI Agent Engine por minuto

## Gestionar Memoria con `VertexAiMemoryBankService` {#vertex-ai-memory-bank}

[`VertexAiMemoryBankService`](/sessions/memory.md#vertex-ai-memory-bank)
es compatible con las claves de API del modo express de Vertex AI. En su lugar puedes inicializar
el objeto de memoria sin ningún proyecto o ubicación.

```py
# Requiere: pip install google-adk[vertexai]
# Más configuración de variable de entorno:
# GOOGLE_GENAI_USE_VERTEXAI=TRUE
# GOOGLE_API_KEY=PASTE_YOUR_ACTUAL_EXPRESS_MODE_API_KEY_HERE
from google.adk.memory import VertexAiMemoryBankService

# El app_name usado con este servicio debe ser el ID o nombre del Reasoning Engine
APP_ID = "your-reasoning-engine-id"

# Project y location no son requeridos al inicializar con el modo express
memory_service = VertexAiMemoryBankService(agent_engine_id=APP_ID)
# Genera una memoria de esa sesión para que el Agente pueda recordar detalles relevantes sobre el usuario
# memory = await memory_service.add_session_to_memory(session)
```

!!! info "Cuotas del Memory Service"

    Para Proyectos de modo express gratuitos, `VertexAiMemoryBankService` tiene la siguiente cuota:

    - 10 Crear, eliminar o actualizar recursos de memoria de Vertex AI Agent Engine por minuto
    - 10 Obtener, listar o recuperar del Memory Bank de Vertex AI Agent Engine por minuto

### Ejemplo de Código: Agente del Clima con Sesión y Memoria

Este ejemplo de código muestra un agente del clima que utiliza tanto
`VertexAiSessionService` como `VertexAiMemoryBankService` para la gestión de contexto,
permitiendo que tu agente recuerde las preferencias del usuario y las conversaciones.

*   [Agente del Clima con Sesión y Memoria](https://github.com/google/blob/main/examples/python/notebooks/express-mode-weather-agent.ipynb)
    usando el modo express de Vertex AI