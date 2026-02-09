# Memoria: Conocimiento a Largo Plazo con `MemoryService`

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Hemos visto cómo `Session` rastrea el historial (`events`) y los datos temporales (`state`) para una *única conversación en curso*. Pero ¿qué pasa si un agente necesita recordar información de conversaciones *pasadas*? Aquí es donde entra en juego el concepto de **Conocimiento a Largo Plazo** y el **`MemoryService`**.

Piénsalo de esta manera:

* **`Session` / `State`:** Como tu memoria a corto plazo durante un chat específico.
* **Conocimiento a Largo Plazo (`MemoryService`)**: Como un archivo o biblioteca de conocimientos consultable que el agente puede consultar, potencialmente conteniendo información de muchos chats pasados u otras fuentes.

## El Rol del `MemoryService`

El `BaseMemoryService` define la interfaz para gestionar este almacén de conocimiento consultable a largo plazo. Sus responsabilidades principales son:

1. **Ingerir Información (`add_session_to_memory`):** Tomar el contenido de una `Session` (generalmente completada) y agregar información relevante al almacén de conocimiento a largo plazo.
2. **Buscar Información (`search_memory`):** Permitir que un agente (típicamente a través de una `Tool`) consulte el almacén de conocimiento y recupere fragmentos o contexto relevante basado en una consulta de búsqueda.

## Elegir el Memory Service Correcto

El ADK ofrece dos implementaciones distintas de `MemoryService`, cada una adaptada a diferentes casos de uso. Utiliza la tabla a continuación para decidir cuál es la más adecuada para tu agente.

| **Característica** | **InMemoryMemoryService** | **VertexAiMemoryBankService** |
| :--- | :--- | :--- |
| **Persistencia** | Ninguna (los datos se pierden al reiniciar) | Sí (Gestionado por Vertex AI) |
| **Caso de Uso Principal** | Prototipado, desarrollo local y pruebas simples. | Construir memorias significativas y evolutivas a partir de conversaciones de usuarios. |
| **Extracción de Memoria** | Almacena la conversación completa | Extrae [información significativa](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/memory-bank/generate-memories) de las conversaciones y la consolida con memorias existentes (impulsado por LLM) |
| **Capacidad de Búsqueda** | Coincidencia básica de palabras clave. | Búsqueda semántica avanzada. |
| **Complejidad de Configuración** | Ninguna. Es el predeterminado. | Baja. Requiere una instancia de [Agent Engine](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/memory-bank/overview) en Vertex AI. |
| **Dependencias** | Ninguna. | Proyecto de Google Cloud, API de Vertex AI |
| **Cuándo usarlo** | Cuando quieras buscar en los historiales de chat de múltiples sesiones para prototipado. | Cuando quieras que tu agente recuerde y aprenda de interacciones pasadas. |

## Memoria In-Memory

El `InMemoryMemoryService` almacena información de la sesión en la memoria de la aplicación y realiza coincidencias básicas de palabras clave para las búsquedas. No requiere configuración y es ideal para escenarios de prototipado y pruebas simples donde no se requiere persistencia.

=== "Python"

    ```py
    from google.adk.memory import InMemoryMemoryService
    memory_service = InMemoryMemoryService()
    ```

=== "Go"
    ```go
    import (
      "google.golang.org/adk/memory"
      "google.golang.org/adk/session"
    )

    // Los servicios deben compartirse entre runners para compartir estado y memoria.
    sessionService := session.InMemoryService()
    memoryService := memory.InMemoryService()
    ```


**Ejemplo: Agregar y Buscar Memoria**

Este ejemplo demuestra el flujo básico usando el `InMemoryMemoryService` para simplificar.

=== "Python"

    ```py
    import asyncio
    from google.adk.agents import LlmAgent
    from google.adk.sessions import InMemorySessionService, Session
    from google.adk.memory import InMemoryMemoryService # Importar MemoryService
    from google.adk.runners import Runner
    from google.adk.tools import load_memory # Herramienta para consultar memoria
    from google.genai.types import Content, Part

    # --- Constantes ---
    APP_NAME = "memory_example_app"
    USER_ID = "mem_user"
    MODEL = "gemini-2.0-flash" # Usar un modelo válido

    # --- Definiciones de Agentes ---
    # Agente 1: Agente simple para capturar información
    info_capture_agent = LlmAgent(
        model=MODEL,
        name="InfoCaptureAgent",
        instruction="Acknowledge the user's statement.",
    )

    # Agente 2: Agente que puede usar memoria
    memory_recall_agent = LlmAgent(
        model=MODEL,
        name="MemoryRecallAgent",
        instruction="Answer the user's question. Use the 'load_memory' tool "
                    "if the answer might be in past conversations.",
        tools=[load_memory] # Dar al agente la herramienta
    )

    # --- Servicios ---
    # Los servicios deben compartirse entre runners para compartir estado y memoria
    session_service = InMemorySessionService()
    memory_service = InMemoryMemoryService() # Usar in-memory para demostración

    async def run_scenario():
        # --- Escenario ---

        # Turno 1: Capturar algo de información en una sesión
        print("--- Turn 1: Capturing Information ---")
        runner1 = Runner(
            # Comenzar con el agente de captura de información
            agent=info_capture_agent,
            app_name=APP_NAME,
            session_service=session_service,
            memory_service=memory_service # Proporcionar el servicio de memoria al Runner
        )
        session1_id = "session_info"
        await runner1.session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=session1_id)
        user_input1 = Content(parts=[Part(text="My favorite project is Project Alpha.")], role="user")

        # Ejecutar el agente
        final_response_text = "(No final response)"
        async for event in runner1.run_async(user_id=USER_ID, session_id=session1_id, new_message=user_input1):
            if event.is_final_response() and event.content and event.content.parts:
                final_response_text = event.content.parts[0].text
        print(f"Agent 1 Response: {final_response_text}")

        # Obtener la sesión completada
        completed_session1 = await runner1.session_service.get_session(app_name=APP_NAME, user_id=USER_ID, session_id=session1_id)

        # Agregar el contenido de esta sesión al Memory Service
        print("\n--- Adding Session 1 to Memory ---")
        await memory_service.add_session_to_memory(completed_session1)
        print("Session added to memory.")

        # Turno 2: Recordar la información en una nueva sesión
        print("\n--- Turn 2: Recalling Information ---")
        runner2 = Runner(
            # Usar el segundo agente, que tiene la herramienta de memoria
            agent=memory_recall_agent,
            app_name=APP_NAME,
            session_service=session_service, # Reutilizar el mismo servicio
            memory_service=memory_service   # Reutilizar el mismo servicio
        )
        session2_id = "session_recall"
        await runner2.session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=session2_id)
        user_input2 = Content(parts=[Part(text="What is my favorite project?")], role="user")

        # Ejecutar el segundo agente
        final_response_text_2 = "(No final response)"
        async for event in runner2.run_async(user_id=USER_ID, session_id=session2_id, new_message=user_input2):
            if event.is_final_response() and event.content and event.content.parts:
                final_response_text_2 = event.content.parts[0].text
        print(f"Agent 2 Response: {final_response_text_2}")

    # Para ejecutar este ejemplo, puedes usar el siguiente fragmento:
    # asyncio.run(run_scenario())

    # await run_scenario()
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/sessions/memory_example/memory_example.go:full_example"
    ```


### Buscar Memoria Dentro de una Herramienta

También puedes buscar memoria desde dentro de una herramienta personalizada usando el `tool.Context`.

=== "Go"

    ```go
    --8<-- "examples/go/snippets/sessions/memory_example/memory_example.go:tool_search"
    ```

## Vertex AI Memory Bank

El `VertexAiMemoryBankService` conecta tu agente a [Vertex AI Memory Bank](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/memory-bank/overview), un servicio de Google Cloud completamente gestionado que proporciona capacidades de memoria sofisticadas y persistentes para agentes conversacionales.

### Cómo Funciona

El servicio maneja dos operaciones clave:

*   **Generar Memorias:** Al final de una conversación, puedes enviar los eventos de la sesión al Memory Bank, que procesa y almacena inteligentemente la información como "memorias".
*   **Recuperar Memorias:** Tu código de agente puede emitir una consulta de búsqueda contra el Memory Bank para recuperar memorias relevantes de conversaciones pasadas.

### Requisitos Previos

Antes de poder usar esta característica, debes tener:

1.  **Un Proyecto de Google Cloud:** Con la API de Vertex AI habilitada.
2.  **Un Agent Engine:** Necesitas crear un Agent Engine en Vertex AI. No necesitas desplegar tu agente en Agent Engine Runtime para usar Memory Bank. Esto te proporcionará el **ID del Agent Engine** requerido para la configuración.
3.  **Autenticación:** Asegúrate de que tu entorno local esté autenticado para acceder a los servicios de Google Cloud. La forma más simple es ejecutar:
    ```bash
    gcloud auth application-default login
    ```
4.  **Variables de Entorno:** El servicio requiere tu ID de Proyecto de Google Cloud y Ubicación. Configúralos como variables de entorno:
    ```bash
    export GOOGLE_CLOUD_PROJECT="your-gcp-project-id"
    export GOOGLE_CLOUD_LOCATION="your-gcp-location"
    ```

### Configuración

Para conectar tu agente al Memory Bank, usas la bandera `--memory_service_uri` al iniciar el servidor ADK (`adk web` o `adk api_server`). El URI debe estar en el formato `agentengine://<agent_engine_id>`.

```bash title="bash"
adk web path/to/your/agents_dir --memory_service_uri="agentengine://1234567890"
```

O puedes configurar tu agente para usar el Memory Bank instanciando manualmente el `VertexAiMemoryBankService` y pasándolo al `Runner`.

=== "Python"
  ```py
  from google.adk.memory import VertexAiMemoryBankService

  agent_engine_id = agent_engine.api_resource.name.split("/")[-1]

  memory_service = VertexAiMemoryBankService(
      project="PROJECT_ID",
      location="LOCATION",
      agent_engine_id=agent_engine_id
  )

  runner = adk.Runner(
      ...
      memory_service=memory_service
  )
  ```

## Usar Memoria en tu Agente

Cuando un servicio de memoria está configurado, tu agente puede usar una herramienta o callback para recuperar memorias. ADK incluye dos herramientas preconstruidas para recuperar memorias:

* `PreloadMemory`: Siempre recupera memoria al comienzo de cada turno (similar a un callback).
* `LoadMemory`: Recupera memoria cuando tu agente decide que sería útil.

**Ejemplo:**

=== "Python"
```python
from google.adk.agents import Agent
from google.adk.tools.preload_memory_tool import PreloadMemoryTool

agent = Agent(
    model=MODEL_ID,
    name='weather_sentiment_agent',
    instruction="...",
    tools=[PreloadMemoryTool()]
)
```

Para extraer memorias de tu sesión, necesitas llamar a `add_session_to_memory`. Por ejemplo, puedes automatizar esto a través de un callback:

=== "Python"
```python
from google import adk

async def auto_save_session_to_memory_callback(callback_context):
    await callback_context._invocation_context.memory_service.add_session_to_memory(
        callback_context._invocation_context.session)

agent = Agent(
    model=MODEL,
    name="Generic_QA_Agent",
    instruction="Answer the user's questions",
    tools=[adk.tools.preload_memory_tool.PreloadMemoryTool()],
    after_agent_callback=auto_save_session_to_memory_callback,
)
```

## Conceptos Avanzados

### Cómo Funciona la Memoria en la Práctica

El flujo de trabajo de memoria internamente involucra estos pasos:

1. **Interacción de Sesión:** Un usuario interactúa con un agente a través de una `Session`, gestionada por un `SessionService`. Se agregan eventos, y el estado puede actualizarse.
2. **Ingestión en Memoria:** En algún punto (a menudo cuando una sesión se considera completa o ha producido información significativa), tu aplicación llama a `memory_service.add_session_to_memory(session)`. Esto extrae información relevante de los eventos de la sesión y la agrega al almacén de conocimiento a largo plazo (diccionario en memoria o Agent Engine Memory Bank).
3. **Consulta Posterior:** En una sesión *diferente* (o la misma), el usuario podría hacer una pregunta que requiere contexto pasado (por ejemplo, "¿De qué hablamos sobre el proyecto X la semana pasada?").
4. **El Agente Usa la Herramienta de Memoria:** Un agente equipado con una herramienta de recuperación de memoria (como la herramienta incorporada `load_memory`) reconoce la necesidad de contexto pasado. Llama a la herramienta, proporcionando una consulta de búsqueda (por ejemplo, "discusión proyecto X la semana pasada").
5. **Ejecución de Búsqueda:** La herramienta internamente llama a `memory_service.search_memory(app_name, user_id, query)`.
6. **Resultados Devueltos:** El `MemoryService` busca en su almacén (usando coincidencia de palabras clave o búsqueda semántica) y devuelve fragmentos relevantes como un `SearchMemoryResponse` que contiene una lista de objetos `MemoryResult` (cada uno potencialmente conteniendo eventos de una sesión pasada relevante).
7. **El Agente Usa los Resultados:** La herramienta devuelve estos resultados al agente, generalmente como parte del contexto o respuesta de función. El agente puede entonces usar esta información recuperada para formular su respuesta final al usuario.

### ¿Puede un agente tener acceso a más de un servicio de memoria?

*   **A través de la Configuración Estándar: No.** El framework (`adk web`, `adk api_server`) está diseñado para configurarse con un único servicio de memoria a la vez a través de la bandera `--memory_service_uri`. Este único servicio se proporciona entonces al agente y se accede a través del método incorporado `self.search_memory()`. Desde un punto de vista de configuración, solo puedes elegir un backend (`InMemory`, `VertexAiMemoryBankService`) para todos los agentes servidos por ese proceso.

*   **Dentro del Código de tu Agente: Sí, absolutamente.** No hay nada que te impida importar e instanciar manualmente otro servicio de memoria directamente dentro del código de tu agente. Esto te permite acceder a múltiples fuentes de memoria dentro de un único turno de agente.

Por ejemplo, tu agente podría usar el `InMemoryMemoryService` configurado por el framework para recordar el historial conversacional, y también instanciar manualmente un `VertexAiMemoryBankService` para buscar información en un manual técnico.

#### Ejemplo: Usar Dos Servicios de Memoria

Aquí está cómo podrías implementar eso en el código de tu agente:

=== "Python"
```python
from google.adk.agents import Agent
from google.adk.memory import InMemoryMemoryService, VertexAiMemoryBankService
from google.genai import types

class MultiMemoryAgent(Agent):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

        self.memory_service = InMemoryMemoryService()
        # Instanciar manualmente un segundo servicio de memoria para búsquedas de documentos
        self.vertexai_memorybank_service = VertexAiMemoryBankService(
            project="PROJECT_ID",
            location="LOCATION",
            agent_engine_id="AGENT_ENGINE_ID"
        )

    async def run(self, request: types.Content, **kwargs) -> types.Content:
        user_query = request.parts[0].text

        # 1. Buscar historial conversacional usando la memoria proporcionada por el framework
        #    (Esto sería InMemoryMemoryService si está configurado)
        conversation_context = await self.memory_service.search_memory(query=user_query)

        # 2. Buscar en la base de conocimientos de documentos usando el servicio creado manualmente
        document_context = await self.vertexai_memorybank_service.search_memory(query=user_query)

        # Combinar el contexto de ambas fuentes para generar una mejor respuesta
        prompt = "From our past conversations, I remember:\n"
        prompt += f"{conversation_context.memories}\n\n"
        prompt += "From the technical manuals, I found:\n"
        prompt += f"{document_context.memories}\n\n"
        prompt += f"Based on all this, here is my answer to '{user_query}':"

        return await self.llm.generate_content_async(prompt)
```