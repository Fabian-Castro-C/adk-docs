# Session: Seguimiento de Conversaciones Individuales

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Siguiendo nuestra Introducción, profundicemos en `Session`. Recuerda el
concepto de "hilo de conversación". Así como no comenzarías cada mensaje de texto
desde cero, los agentes necesitan contexto sobre la interacción en curso.
**`Session`** es el objeto del ADK diseñado específicamente para rastrear y gestionar estos
hilos de conversación individuales.

## El Objeto `Session`

Cuando un usuario comienza a interactuar con tu agente, el `SessionService` crea un
objeto `Session` (`google.adk.sessions.Session`). Este objeto actúa como el
contenedor que mantiene todo lo relacionado con *ese hilo de chat específico*. Aquí
están sus propiedades clave:

*   **Identificación (`id`, `appName`, `userId`):** Etiquetas únicas para la
    conversación.
    * `id`: Un identificador único para *este hilo de conversación específico*, esencial para recuperarlo más tarde. Un objeto SessionService puede manejar múltiples `Session`(s). Este campo identifica a qué objeto de sesión en particular nos estamos refiriendo. Por ejemplo, "test_id_modification".
    * `app_name`: Identifica a qué aplicación de agente pertenece esta conversación. Por ejemplo, "id_modifier_workflow".
    *   `userId`: Vincula la conversación a un usuario en particular.
*   **Historial (`events`):** Una secuencia cronológica de todas las interacciones
    (objetos `Event` – mensajes de usuario, respuestas del agente, acciones de herramientas) que han
    ocurrido dentro de este hilo específico.
*   **Estado de Sesión (`state`):** Un lugar para almacenar datos temporales relevantes *solo*
    para esta conversación específica en curso. Esto actúa como un bloc de notas para el
    agente durante la interacción. Cubriremos cómo usar y gestionar `state` en
    detalle en la siguiente sección.
*   **Seguimiento de Actividad (`lastUpdateTime`):** Una marca de tiempo que indica la última
    vez que ocurrió un evento en este hilo de conversación.

### Ejemplo: Examinar Propiedades de Session


=== "Python"

       ```py
        from google.adk.sessions import InMemorySessionService, Session

        # Crear una sesión simple para examinar sus propiedades
        temp_service = InMemorySessionService()
        example_session = await temp_service.create_session(
            app_name="my_app",
            user_id="example_user",
            state={"initial_key": "initial_value"} # El estado puede ser inicializado
        )

        print(f"--- Examining Session Properties ---")
        print(f"ID (`id`):                {example_session.id}")
        print(f"Application Name (`app_name`): {example_session.app_name}")
        print(f"User ID (`user_id`):         {example_session.user_id}")
        print(f"State (`state`):           {example_session.state}") # Nota: Solo muestra el estado inicial aquí
        print(f"Events (`events`):         {example_session.events}") # Inicialmente vacío
        print(f"Last Update (`last_update_time`): {example_session.last_update_time:.2f}")
        print(f"---------------------------------")

        # Limpiar (opcional para este ejemplo)
        temp_service = await temp_service.delete_session(app_name=example_session.app_name,
                                    user_id=example_session.user_id, session_id=example_session.id)
        print("The final status of temp_service - ", temp_service)
       ```

=== "TypeScript"

       ```typescript
        import { InMemorySessionService } from "@google/adk";

        // Crear una sesión simple para examinar sus propiedades
        const tempService = new InMemorySessionService();
        const exampleSession = await tempService.createSession({
            appName: "my_app",
            userId: "example_user",
            state: {"initial_key": "initial_value"} // El estado puede ser inicializado
        });

        console.log("--- Examining Session Properties ---");
        console.log(`ID ('id'):                ${exampleSession.id}`);
        console.log(`Application Name ('appName'): ${exampleSession.appName}`);
        console.log(`User ID ('userId'):         ${exampleSession.userId}`);
        console.log(`State ('state'):           ${JSON.stringify(exampleSession.state)}`); // Nota: Solo muestra el estado inicial aquí
        console.log(`Events ('events'):         ${JSON.stringify(exampleSession.events)}`); // Inicialmente vacío
        console.log(`Last Update ('lastUpdateTime'): ${exampleSession.lastUpdateTime}`);
        console.log("---------------------------------");

        // Limpiar (opcional para este ejemplo)
        const finalStatus = await tempService.deleteSession({
            appName: exampleSession.appName,
            userId: exampleSession.userId,
            sessionId: exampleSession.id
        });
        console.log("The final status of temp_service - ", finalStatus);
       ```

=== "Go"

       ```go
       --8<-- "examples/go/snippets/sessions/session_management_example/session_management_example.go:examine_session"
       ```

=== "Java"

       ```java
        import com.google.adk.sessions.InMemorySessionService;
        import com.google.adk.sessions.Session;
        import java.util.concurrent.ConcurrentMap;
        import java.util.concurrent.ConcurrentHashMap;

        String sessionId = "123";
        String appName = "example-app"; // Nombre de app de ejemplo
        String userId = "example-user"; // ID de usuario de ejemplo
        ConcurrentMap<String, Object> initialState = new ConcurrentHashMap<>(Map.of("newKey", "newValue"));
        InMemorySessionService exampleSessionService = new InMemorySessionService();

        // Crear Session
        Session exampleSession = exampleSessionService.createSession(
            appName, userId, initialState, Optional.of(sessionId)).blockingGet();
        System.out.println("Session created successfully.");

        System.out.println("--- Examining Session Properties ---");
        System.out.printf("ID (`id`): %s%n", exampleSession.id());
        System.out.printf("Application Name (`appName`): %s%n", exampleSession.appName());
        System.out.printf("User ID (`userId`): %s%n", exampleSession.userId());
        System.out.printf("State (`state`): %s%n", exampleSession.state());
        System.out.println("------------------------------------");


        // Limpiar (opcional para este ejemplo)
        var unused = exampleSessionService.deleteSession(appName, userId, sessionId);
       ```

*(**Nota:** El estado mostrado arriba es solo el estado inicial. Las actualizaciones de estado
ocurren a través de eventos, como se discute en la sección Estado.)*

## Gestión de Sessions con un `SessionService`

Como se vio arriba, normalmente no creas ni gestionas objetos `Session` directamente.
En su lugar, usas un **`SessionService`**. Este servicio actúa como el gestor
central responsable del ciclo de vida completo de tus sesiones de conversación.

Sus responsabilidades principales incluyen:

*   **Iniciar Nuevas Conversaciones:** Crear objetos `Session` nuevos cuando un usuario
    comienza una interacción.
*   **Reanudar Conversaciones Existentes:** Recuperar una `Session` específica (usando
    su ID) para que el agente pueda continuar donde lo dejó.
*   **Guardar Progreso:** Agregar nuevas interacciones (objetos `Event`) al
    historial de una sesión. Este es también el mecanismo a través del cual el `state` de la sesión
    se actualiza (más en la sección `State`).
*   **Listar Conversaciones:** Encontrar los hilos de sesión activos para un
    usuario y aplicación en particular.
*   **Limpiar:** Eliminar objetos `Session` y sus datos asociados cuando
    las conversaciones han terminado o ya no son necesarias.

## Implementaciones de `SessionService`

ADK proporciona diferentes implementaciones de `SessionService`, permitiéndote elegir
el backend de almacenamiento que mejor se adapte a tus necesidades:

### `InMemorySessionService`

*   **Cómo funciona:** Almacena todos los datos de sesión directamente en la memoria de la aplicación.
*   **Persistencia:** Ninguna. **Todos los datos de conversación se pierden si la
    aplicación se reinicia.**
*   **Requiere:** Nada adicional.
*   **Ideal para:** Desarrollo rápido, pruebas locales, ejemplos y escenarios
    donde no se requiere persistencia a largo plazo.

=== "Python"

      ```py
        from google.adk.sessions import InMemorySessionService
        session_service = InMemorySessionService()
      ```
=== "TypeScript"

      ```typescript
        import { InMemorySessionService } from "@google/adk";
        const sessionService = new InMemorySessionService();
      ```

=== "Go"

      ```go
        import "google.golang.org/adk/session"
        inMemoryService := session.InMemoryService()
      ```

=== "Java"

      ```java
        import com.google.adk.sessions.InMemorySessionService;
        InMemorySessionService exampleSessionService = new InMemorySessionService();
      ```

### `VertexAiSessionService`

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

*   **Cómo funciona:** Utiliza la infraestructura de Google Cloud Vertex AI a través de llamadas API
    para la gestión de sesiones.
*   **Persistencia:** Sí. Los datos se gestionan de manera confiable y escalable a través de
    [Vertex AI Agent Engine](https://google.github.io/deploy/agent-engine/).
*   **Requiere:**
    *   Un proyecto de Google Cloud (`pip install vertexai`)
    *   Un bucket de almacenamiento de Google Cloud que puede ser configurado con este
        [paso](https://cloud.google.com/vertex-ai/docs/pipelines/configure-project#storage).
    *   Un nombre/ID de recurso de Reasoning Engine que puede configurarse siguiendo este
        [tutorial](https://google.github.io/deploy/agent-engine/).
    *   Si no tienes un proyecto de Google Cloud y quieres probar el VertexAiSessionService, consulta [Vertex AI Express Mode](/tools/google-cloud/express-mode/).
*   **Ideal para:** Aplicaciones de producción escalables desplegadas en Google Cloud,
    especialmente al integrarse con otras características de Vertex AI.

=== "Python"

    ```py
    # Requiere: pip install google-adk[vertexai]
    # Más configuración de GCP y autenticación
    from google.adk.sessions import VertexAiSessionService

    PROJECT_ID = "your-gcp-project-id"
    LOCATION = "us-central1"
    # El app_name usado con este servicio debe ser el ID o nombre del Reasoning Engine
    REASONING_ENGINE_APP_NAME = "projects/your-gcp-project-id/locations/us-central1/reasoningEngines/your-engine-id"

    session_service = VertexAiSessionService(project=PROJECT_ID, location=LOCATION)
    # Usar REASONING_ENGINE_APP_NAME al llamar métodos del servicio, ej.:
    # session_service = await session_service.create_session(app_name=REASONING_ENGINE_APP_NAME, ...)
    ```

=== "Go"

    ```go
    import "google.golang.org/adk/session"

    // 2. VertexAIService
    // Antes de ejecutar, asegúrate de que tu entorno esté autenticado:
    // gcloud auth application-default login
    // export GOOGLE_CLOUD_PROJECT="your-gcp-project-id"
    // export GOOGLE_CLOUD_LOCATION="your-gcp-location"

    modelName := "gemini-flash-latest" // Reemplazar con tu modelo deseado
    vertexService, err := session.VertexAIService(ctx, modelName)
    if err != nil {
      log.Printf("Could not initialize VertexAIService (this is expected if the gcloud project is not set): %v", err)
    } else {
      fmt.Println("Successfully initialized VertexAIService.")
    }
    ```

=== "Java"

    ```java
    // Por favor revisa el conjunto de requisitos arriba, consecuentemente exporta lo siguiente en tu archivo bashrc:
    // export GOOGLE_CLOUD_PROJECT=my_gcp_project
    // export GOOGLE_CLOUD_LOCATION=us-central1
    // export GOOGLE_API_KEY=my_api_key

    import com.google.adk.sessions.VertexAiSessionService;
    import java.util.UUID;

    String sessionId = UUID.randomUUID().toString();
    String reasoningEngineAppName = "123456789";
    String userId = "u_123"; // ID de usuario de ejemplo
    ConcurrentMap<String, Object> initialState = new
        ConcurrentHashMap<>(); // No se necesita estado inicial para este ejemplo

    VertexAiSessionService sessionService = new VertexAiSessionService();
    Session mySession =
        sessionService
            .createSession(reasoningEngineAppName, userId, initialState, Optional.of(sessionId))
            .blockingGet();
    ```

### `DatabaseSessionService`

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-go">Go v0.1.0</span>
</div>

*   **Cómo funciona:** Se conecta a una base de datos relacional (ej., PostgreSQL,
    MySQL, SQLite) para almacenar datos de sesión de manera persistente en tablas.
*   **Persistencia:** Sí. Los datos sobreviven a reinicios de la aplicación.
*   **Requiere:** Una base de datos configurada.
*   **Ideal para:** Aplicaciones que necesitan almacenamiento persistente y confiable que tú
    gestionas tú mismo.

```py
from google.adk.sessions import DatabaseSessionService
# Ejemplo usando un archivo SQLite local:
# Nota: La implementación requiere un controlador de base de datos asíncrono.
# Para SQLite, usa 'sqlite+aiosqlite' en lugar de 'sqlite' para garantizar compatibilidad asíncrona.
db_url = "sqlite+aiosqlite:///./my_agent_data.db"
session_service = DatabaseSessionService(db_url=db_url)
```

!!! warning "Requisito de Controlador Asíncrono"

    `DatabaseSessionService` requiere un controlador de base de datos asíncrono. Al usar SQLite,
    debes usar `sqlite+aiosqlite` en lugar de `sqlite` en tu cadena de conexión.
    Para otras bases de datos (PostgreSQL, MySQL), asegúrate de estar usando un controlador
    compatible con asíncrono, como `asyncpg` para PostgreSQL, `aiomysql` para MySQL.

!!! note "Cambio de esquema de base de datos de sesión en ADK Python v1.22.0"

    El esquema para la base de datos de sesión cambió en ADK Python v1.22.0, lo cual
    requiere migración de la Base de Datos de Sesión. Para más información, consulta
    [Migración del esquema de base de datos de sesión](/sessions/session/migrate/).

## El Ciclo de Vida de Session

<img src="../../assets/session_lifecycle.png" alt="Session lifecycle">

Aquí hay un flujo simplificado de cómo `Session` y `SessionService` trabajan juntos
durante un turno de conversación:

1.  **Iniciar o Reanudar:** Tu aplicación necesita usar el `SessionService` para
    ya sea `create_session` (para un chat nuevo) o usar un id de sesión existente.
2.  **Contexto Proporcionado:** El `Runner` obtiene el objeto `Session` apropiado
    del método de servicio apropiado, proporcionando al agente acceso al
    `state` y `events` de la Session correspondiente.
3.  **Procesamiento del Agente:** El usuario le hace una consulta al agente. El agente
    analiza la consulta y potencialmente el `state` de la sesión y el historial de `events`
    para determinar la respuesta.
4.  **Respuesta y Actualización de Estado:** El agente genera una respuesta (y potencialmente
    marca datos para ser actualizados en el `state`). El `Runner` empaqueta esto como un
    `Event`.
5.  **Guardar Interacción:** El `Runner` llama a
    `sessionService.append_event(session, event)` con la `session` y el nuevo
    `event` como argumentos. El servicio agrega el `Event` al historial y
    actualiza el `state` de la sesión en el almacenamiento basándose en información dentro del
    evento. El `last_update_time` de la sesión también se actualiza.
6.  **Listo para el Siguiente:** La respuesta del agente va al usuario. La `Session` actualizada
    ahora está almacenada por el `SessionService`, lista para el siguiente turno
    (que reinicia el ciclo en el paso 1, usualmente con la continuación de la
    conversación en la sesión actual).
7.  **Finalizar Conversación:** Cuando la conversación ha terminado, tu aplicación llama a
    `sessionService.delete_session(...)` para limpiar los datos de sesión almacenados si
    ya no son requeridos.

Este ciclo destaca cómo el `SessionService` asegura continuidad conversacional
al gestionar el historial y estado asociados con cada objeto `Session`.