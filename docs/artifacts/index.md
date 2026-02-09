# Artefactos

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

En ADK, los **Artefactos** representan un mecanismo crucial para gestionar datos binarios nombrados y versionados, asociados ya sea con una sesión de interacción de usuario específica o de manera persistente con un usuario a través de múltiples sesiones. Permiten que tus agentes y herramientas manejen datos más allá de simples cadenas de texto, habilitando interacciones más ricas que involucran archivos, imágenes, audio y otros formatos binarios.

!!! Note
    Los parámetros específicos o nombres de métodos para las primitivas pueden variar ligeramente según el lenguaje del SDK (por ejemplo, `save_artifact` en Python, `saveArtifact` en Java). Consulta la documentación de la API específica del lenguaje para más detalles.

## ¿Qué son los Artefactos?

*   **Definición:** Un Artefacto es esencialmente un fragmento de datos binarios (como el contenido de un archivo) identificado por una cadena `filename` única dentro de un ámbito específico (sesión o usuario). Cada vez que guardas un artefacto con el mismo nombre de archivo, se crea una nueva versión.

*   **Representación:** Los Artefactos se representan consistentemente usando el objeto estándar `google.genai.types.Part`. Los datos principales típicamente se almacenan dentro de una estructura de datos inline del `Part` (accedida vía `inline_data`), que a su vez contiene:
    *   `data`: El contenido binario sin procesar como bytes.
    *   `mime_type`: Una cadena que indica el tipo de datos (por ejemplo, `"image/png"`, `"application/pdf"`). Esto es esencial para interpretar correctamente los datos más adelante.


=== "Python"

    ```py
    # Ejemplo de cómo un artefacto podría representarse como un types.Part
    import google.genai.types as types

    # Asume que 'image_bytes' contiene los datos binarios de una imagen PNG
    image_bytes = b'\x89PNG\r\n\x1a\n...' # Marcador de posición para bytes de imagen reales

    image_artifact = types.Part(
        inline_data=types.Blob(
            mime_type="image/png",
            data=image_bytes
        )
    )

    # También puedes usar el constructor de conveniencia:
    # image_artifact_alt = types.Part.from_bytes(data=image_bytes, mime_type="image/png")

    print(f"Artifact MIME Type: {image_artifact.inline_data.mime_type}")
    print(f"Artifact Data (first 10 bytes): {image_artifact.inline_data.data[:10]}...")
    ```

=== "Typescript"

    ```typescript
    import type { Part } from '@google/genai';
    import { createPartFromBase64 } from '@google/genai';

    // Asume que 'imageBytes' contiene los datos binarios de una imagen PNG
    const imageBytes = new Uint8Array([0x89, 0x50, 0x4e, 0x47, 0x0d, 0x0a, 0x1a, 0x0a]); // Marcador de posición

    const imageArtifact: Part = createPartFromBase64(imageBytes.toString('base64'), "image/png");

    console.log(`Artifact MIME Type: ${imageArtifact.inlineData?.mimeType}`);
    // Nota: Acceder a los bytes sin procesar requeriría decodificar desde base64.
    ```

=== "Go"

    ```go
    import (
      "log"

      "google.golang.org/genai"
    )

    --8<-- "examples/go/snippets/artifacts/main.go:representation"
    ```

=== "Java"

    ```java
    import com.google.genai.types.Part;
    import java.nio.charset.StandardCharsets;

    public class ArtifactExample {
        public static void main(String[] args) {
            // Asume que 'imageBytes' contiene los datos binarios de una imagen PNG
            byte[] imageBytes = {(byte) 0x89, (byte) 0x50, (byte) 0x4E, (byte) 0x47, (byte) 0x0D, (byte) 0x0A, (byte) 0x1A, (byte) 0x0A, (byte) 0x01, (byte) 0x02}; // Marcador de posición para bytes de imagen reales

            // Crea un artefacto de imagen usando Part.fromBytes
            Part imageArtifact = Part.fromBytes(imageBytes, "image/png");

            System.out.println("Artifact MIME Type: " + imageArtifact.inlineData().get().mimeType().get());
            System.out.println(
                "Artifact Data (first 10 bytes): "
                    + new String(imageArtifact.inlineData().get().data().get(), 0, 10, StandardCharsets.UTF_8)
                    + "...");
        }
    }
    ```

*   **Persistencia y Gestión:** Los Artefactos no se almacenan directamente dentro del estado del agente o la sesión. Su almacenamiento y recuperación son gestionados por un **Servicio de Artefactos** dedicado (una implementación de `BaseArtifactService`, definida en `google.adk.artifacts`. ADK proporciona varias implementaciones, como:
    *   Un servicio en memoria para pruebas o almacenamiento temporal (por ejemplo, `InMemoryArtifactService` en Python, definido en `google.adk.artifacts.in_memory_artifact_service.py`).
    *   Un servicio para almacenamiento persistente usando Google Cloud Storage (GCS) (por ejemplo, `GcsArtifactService` en Python, definido en `google.adk.artifacts.gcs_artifact_service.py`).
    La implementación del servicio elegida maneja el versionado automáticamente cuando guardas datos.

## ¿Por qué usar Artefactos?

Mientras que el `state` de sesión es adecuado para almacenar pequeños fragmentos de configuración o contexto conversacional (como cadenas, números, booleanos o diccionarios/listas pequeños), los Artefactos están diseñados para escenarios que involucran datos binarios o grandes:

1. **Manejo de Datos No Textuales:** Almacena y recupera fácilmente imágenes, clips de audio, fragmentos de video, PDFs, hojas de cálculo o cualquier otro formato de archivo relevante para la función de tu agente.
2. **Persistencia de Datos Grandes:** El estado de sesión generalmente no está optimizado para almacenar grandes cantidades de datos. Los Artefactos proporcionan un mecanismo dedicado para persistir blobs más grandes sin saturar el estado de sesión.
3. **Gestión de Archivos de Usuario:** Proporciona capacidades para que los usuarios suban archivos (que pueden guardarse como artefactos) y recuperen o descarguen archivos generados por el agente (cargados desde artefactos).
4. **Compartir Salidas:** Permite que las herramientas o agentes generen salidas binarias (como un informe PDF o una imagen generada) que pueden guardarse vía `save_artifact` y accederse posteriormente por otras partes de la aplicación o incluso en sesiones posteriores (si se usa el espacio de nombres de usuario).
5. **Caché de Datos Binarios:** Almacena los resultados de operaciones computacionalmente costosas que producen datos binarios (por ejemplo, renderizar una imagen de gráfico compleja) como artefactos para evitar regenerarlos en solicitudes posteriores.

En esencia, siempre que tu agente necesite trabajar con datos binarios similares a archivos que necesitan ser persistidos, versionados o compartidos, los Artefactos gestionados por un `ArtifactService` son el mecanismo apropiado dentro de ADK.


## Casos de Uso Comunes

Los Artefactos proporcionan una forma flexible de manejar datos binarios dentro de tus aplicaciones ADK.

Aquí hay algunos escenarios típicos donde demuestran ser valiosos:

* **Informes/Archivos Generados:**
    * Una herramienta o agente genera un informe (por ejemplo, un análisis en PDF, una exportación de datos CSV, un gráfico de imagen).

* **Manejo de Cargas de Usuario:**

    * Un usuario sube un archivo (por ejemplo, una imagen para análisis, un documento para resumir) a través de una interfaz front-end.

* **Almacenamiento de Resultados Binarios Intermedios:**

    * Un agente realiza un proceso complejo de múltiples pasos donde un paso genera datos binarios intermedios (por ejemplo, síntesis de audio, resultados de simulación).

* **Datos de Usuario Persistentes:**

    * Almacenar configuración o datos específicos del usuario que no son un estado simple de clave-valor.

* **Caché de Contenido Binario Generado:**

    * Un agente genera frecuentemente la misma salida binaria basada en ciertas entradas (por ejemplo, una imagen de logotipo de empresa, un saludo de audio estándar).



## Conceptos Centrales

Comprender los artefactos implica captar algunos componentes clave: el servicio que los gestiona, la estructura de datos utilizada para contenerlos, y cómo se identifican y versionan.

### Servicio de Artefactos (`BaseArtifactService`)

* **Rol:** El componente central responsable de la lógica real de almacenamiento y recuperación de artefactos. Define *cómo* y *dónde* se persisten los artefactos.

* **Interfaz:** Definida por la clase base abstracta `BaseArtifactService`. Cualquier implementación concreta debe proporcionar métodos para:

    * `Save Artifact`: Almacena los datos del artefacto y devuelve su número de versión asignado.
    * `Load Artifact`: Recupera una versión específica (o la última) de un artefacto.
    * `List Artifact keys`: Lista los nombres de archivo únicos de artefactos dentro de un ámbito dado.
    * `Delete Artifact`: Elimina un artefacto (y potencialmente todas sus versiones, dependiendo de la implementación).
    * `List versions`: Lista todos los números de versión disponibles para un nombre de archivo de artefacto específico.

* **Configuración:** Proporcionas una instancia de un servicio de artefactos (por ejemplo, `InMemoryArtifactService`, `GcsArtifactService`) al inicializar el `Runner`. El `Runner` entonces hace que este servicio esté disponible para agentes y herramientas vía el `InvocationContext`.

=== "Python"

    ```py
    from google.adk.runners import Runner
    from google.adk.artifacts import InMemoryArtifactService # O GcsArtifactService
    from google.adk.agents import LlmAgent # Cualquier agente
    from google.adk.sessions import InMemorySessionService

    # Ejemplo: Configurando el Runner con un Servicio de Artefactos
    my_agent = LlmAgent(name="artifact_user_agent", model="gemini-2.0-flash")
    artifact_service = InMemoryArtifactService() # Elige una implementación
    session_service = InMemorySessionService()

    runner = Runner(
        agent=my_agent,
        app_name="my_artifact_app",
        session_service=session_service,
        artifact_service=artifact_service # Proporciona la instancia del servicio aquí
    )
    # Ahora, los contextos dentro de ejecuciones gestionadas por este runner pueden usar métodos de artefactos
    ```

=== "Typescript"

    ```typescript
    import { InMemoryRunner } from '@google/adk';
    import { LlmAgent } from '@google/adk';
    import { InMemoryArtifactService } from '@google/adk';

    // Ejemplo: Configurando el Runner con un Servicio de Artefactos
    const myAgent = new LlmAgent({name: "artifact_user_agent", model: "gemini-2.5-flash"});
    const artifactService = new InMemoryArtifactService(); // Elige una implementación
    const sessionService = new InMemoryArtifactService();

    const runner = new InMemoryRunner({
        agent: myAgent,
        appName: "my_artifact_app",
        sessionService: sessionService,
        artifactService: artifactService, // Proporciona la instancia del servicio aquí
    });
    // Ahora, los contextos dentro de ejecuciones gestionadas por este runner pueden usar métodos de artefactos
    ```

=== "Go"

    ```go
    import (
      "context"
      "log"

      "google.golang.org/adk/agent/llmagent"
      "google.golang.org/adk/artifactservice"
      "google.golang.org/adk/llm/gemini"
      "google.golang.org/adk/runner"
      "google.golang.org/adk/sessionservice"
      "google.golang.org/genai"
	)

	--8<-- "examples/go/snippets/artifacts/main.go:configure-runner"
    ```

=== "Java"

    ```java
    import com.google.adk.agents.LlmAgent;
    import com.google.adk.runner.Runner;
    import com.google.adk.sessions.InMemorySessionService;
    import com.google.adk.artifacts.InMemoryArtifactService;

    // Ejemplo: Configurando el Runner con un Servicio de Artefactos
    LlmAgent myAgent =  LlmAgent.builder()
      .name("artifact_user_agent")
      .model("gemini-2.0-flash")
      .build();
    InMemoryArtifactService artifactService = new InMemoryArtifactService(); // Elige una implementación
    InMemorySessionService sessionService = new InMemorySessionService();

    Runner runner = new Runner(myAgent, "my_artifact_app", artifactService, sessionService); // Proporciona la instancia del servicio aquí
    // Ahora, los contextos dentro de ejecuciones gestionadas por este runner pueden usar métodos de artefactos
    ```

### Datos de Artefactos

* **Representación Estándar:** El contenido de los artefactos se representa universalmente usando el objeto `google.genai.types.Part`, la misma estructura utilizada para partes de mensajes LLM.

* **Atributo Clave (`inline_data`):** Para artefactos, el atributo más relevante es `inline_data`, que es un objeto `google.genai.types.Blob` que contiene:

    * `data` (`bytes`): El contenido binario sin procesar del artefacto.
    * `mime_type` (`str`): Una cadena de tipo MIME estándar (por ejemplo, `'application/pdf'`, `'image/png'`, `'audio/mpeg'`) que describe la naturaleza de los datos binarios. **Esto es crucial para la interpretación correcta al cargar el artefacto.**

=== "Python"

    ```python
    import google.genai.types as types

    # Ejemplo: Creando un Part de artefacto desde bytes sin procesar
    pdf_bytes = b'%PDF-1.4...' # Tus datos PDF sin procesar
    pdf_mime_type = "application/pdf"

    # Usando el constructor
    pdf_artifact_py = types.Part(
        inline_data=types.Blob(data=pdf_bytes, mime_type=pdf_mime_type)
    )

    # Usando el método de clase de conveniencia (equivalente)
    pdf_artifact_alt_py = types.Part.from_bytes(data=pdf_bytes, mime_type=pdf_mime_type)

    print(f"Created Python artifact with MIME type: {pdf_artifact_py.inline_data.mime_type}")
    ```

=== "Typescript"

    ```typescript
    import type { Part } from '@google/genai';
    import { createPartFromBase64 } from '@google/genai';

    // Ejemplo: Creando un Part de artefacto desde bytes sin procesar
    const pdfBytes = new Uint8Array([0x25, 0x50, 0x44, 0x46, 0x2d, 0x31, 0x2e, 0x34]); // Tus datos PDF sin procesar
    const pdfMimeType = "application/pdf";

    const pdfArtifact: Part = createPartFromBase64(pdfBytes.toString('base64'), pdfMimeType);
    console.log(`Created TypeScript artifact with MIME Type: ${pdfArtifact.inlineData?.mimeType}`);
    ```

=== "Go"

    ```go
    import (
      "log"
      "os"

      "google.golang.org/genai"
    )

    --8<-- "examples/go/snippets/artifacts/main.go:artifact-data"
    ```

=== "Java"

    ```java
    --8<-- "examples/java/snippets/src/main/java/artifacts/ArtifactDataExample.java:full_code"
    ```

### Nombre de Archivo

* **Identificador:** Una cadena simple utilizada para nombrar y recuperar un artefacto dentro de su espacio de nombres específico.
* **Unicidad:** Los nombres de archivo deben ser únicos dentro de su ámbito (ya sea la sesión o el espacio de nombres del usuario).
* **Mejor Práctica:** Usa nombres descriptivos, potencialmente incluyendo extensiones de archivo (por ejemplo, `"monthly_report.pdf"`, `"user_avatar.jpg"`), aunque la extensión en sí no dicta el comportamiento – el `mime_type` sí lo hace.

### Versionado

* **Versionado Automático:** El servicio de artefactos maneja automáticamente el versionado. Cuando llamas a `save_artifact`, el servicio determina el siguiente número de versión disponible (típicamente comenzando desde 0 e incrementando) para ese nombre de archivo y ámbito específicos.
* **Devuelto por `save_artifact`:** El método `save_artifact` devuelve el número de versión entero que fue asignado al artefacto recién guardado.
* **Recuperación:**
  * `load_artifact(..., version=None)` (predeterminado): Recupera la versión *más reciente* disponible del artefacto.
  * `load_artifact(..., version=N)`: Recupera la versión específica `N`.
* **Listar Versiones:** El método `list_versions` (en el servicio, no en el contexto) puede usarse para encontrar todos los números de versión existentes de un artefacto.

### Espacios de Nombres (Sesión vs. Usuario)

* **Concepto:** Los artefactos pueden tener un ámbito ya sea a una sesión específica o más ampliamente a un usuario a través de todas sus sesiones dentro de la aplicación. Este alcance se determina por el formato del `filename` y es manejado internamente por el `ArtifactService`.

* **Predeterminado (Ámbito de Sesión):** Si usas un nombre de archivo simple como `"report.pdf"`, el artefacto se asocia con el `app_name`, `user_id`, *y* `session_id` específicos. Solo es accesible dentro de ese contexto de sesión exacto.


* **Ámbito de Usuario (prefijo `"user:"`):** Si prefijes el nombre de archivo con `"user:"`, como `"user:profile.png"`, el artefacto se asocia solo con el `app_name` y `user_id`. Puede accederse o actualizarse desde *cualquier* sesión perteneciente a ese usuario dentro de la aplicación.


=== "Python"

    ```python
    # Ejemplo ilustrando la diferencia de espacio de nombres (conceptual)

    # Nombre de archivo de artefacto específico de sesión
    session_report_filename = "summary.txt"

    # Nombre de archivo de artefacto específico de usuario
    user_config_filename = "user:settings.json"

    # Cuando se guarda 'summary.txt' vía context.save_artifact,
    # está ligado al app_name, user_id y session_id actuales.

    # Cuando se guarda 'user:settings.json' vía context.save_artifact,
    # la implementación de ArtifactService debe reconocer el prefijo "user:"
    # y darle ámbito a app_name y user_id, haciéndolo accesible a través de sesiones para ese usuario.
    ```

=== "Typescript"

    ```typescript
    // Ejemplo ilustrando la diferencia de espacio de nombres (conceptual)

    // Nombre de archivo de artefacto específico de sesión
    const sessionReportFilename = "summary.txt";

    // Nombre de archivo de artefacto específico de usuario
    const userConfigFilename = "user:settings.json";

    // Cuando se guarda 'summary.txt' vía context.saveArtifact, está ligado al appName, userId y sessionId actuales.
    // Cuando se guarda 'user:settings.json' vía context.saveArtifact, la implementación de ArtifactService reconoce el prefijo "user:" y le da ámbito a appName y userId, haciéndolo accesible a través de sesiones para ese usuario.
    ```

=== "Go"

    ```go
    import (
      "log"
    )

    --8<-- "examples/go/snippets/artifacts/main.go:namespacing"
    ```

=== "Java"

    ```java
    // Ejemplo ilustrando la diferencia de espacio de nombres (conceptual)

    // Nombre de archivo de artefacto específico de sesión
    String sessionReportFilename = "summary.txt";

    // Nombre de archivo de artefacto específico de usuario
    String userConfigFilename = "user:settings.json"; // El prefijo "user:" es clave

    // Cuando se guarda 'summary.txt' vía context.save_artifact,
    // está ligado al app_name, user_id y session_id actuales.
    // artifactService.saveArtifact(appName, userId, sessionId1, sessionReportFilename, someData);

    // Cuando se guarda 'user:settings.json' vía context.save_artifact,
    // la implementación de ArtifactService debe reconocer el prefijo "user:"
    // y darle ámbito a app_name y user_id, haciéndolo accesible a través de sesiones para ese usuario.
    // artifactService.saveArtifact(appName, userId, sessionId1, userConfigFilename, someData);
    ```

Estos conceptos centrales trabajan juntos para proporcionar un sistema flexible para gestionar datos binarios dentro del framework ADK.

## Interactuando con Artefactos (vía Objetos de Contexto)

La forma principal en que interactúas con artefactos dentro de la lógica de tu agente (específicamente dentro de callbacks o herramientas) es a través de métodos proporcionados por los objetos `CallbackContext` y `ToolContext`. Estos métodos abstraen los detalles de almacenamiento subyacentes gestionados por el `ArtifactService`.

### Prerrequisito: Configurando el `ArtifactService`

Antes de poder usar cualquier método de artefactos vía los objetos de contexto, **debes** proporcionar una instancia de una [implementación de `BaseArtifactService`](#available-implementations) (como [`InMemoryArtifactService`](#inmemoryartifactservice) o [`GcsArtifactService`](#gcsartifactservice)) al inicializar tu `Runner`.

=== "Python"

    En Python, proporcionas esta instancia al inicializar tu `Runner`.

    ```python
    from google.adk.runners import Runner
    from google.adk.artifacts import InMemoryArtifactService # O GcsArtifactService
    from google.adk.agents import LlmAgent
    from google.adk.sessions import InMemorySessionService

    # Tu definición de agente
    agent = LlmAgent(name="my_agent", model="gemini-2.0-flash")

    # Instancia el servicio de artefactos deseado
    artifact_service = InMemoryArtifactService()

    # Proporciónalo al Runner
    runner = Runner(
        agent=agent,
        app_name="artifact_app",
        session_service=InMemorySessionService(),
        artifact_service=artifact_service # El servicio debe proporcionarse aquí
    )
    ```
    Si no se configura ningún `artifact_service` en el `InvocationContext` (lo cual sucede si no se pasa al `Runner`), llamar a `save_artifact`, `load_artifact` o `list_artifacts` en los objetos de contexto lanzará un `ValueError`.

=== "Typescript"

    ```typescript
    import { LlmAgent, InMemoryRunner, InMemoryArtifactService } from '@google/adk';

    // Tu definición de agente
    const agent = new LlmAgent({name: "my_agent", model: "gemini-2.5-flash"});

    // Instancia el servicio de artefactos deseado
    const artifactService = new InMemoryArtifactService();

    // Proporciónalo al Runner
    const runner = new InMemoryRunner({
        agent: agent,
        appName: "artifact_app",
        sessionService: new InMemoryArtifactService(),
        artifactService: artifactService, // El servicio debe proporcionarse aquí
    });
    // Si no se configura artifactService, llamar a métodos de artefactos en objetos de contexto lanzará un error.
    ```
    En Java, si una instancia de `ArtifactService` no está disponible (por ejemplo, `null`) cuando se intentan operaciones de artefactos, típicamente resultaría en una `NullPointerException` o un error personalizado, dependiendo de cómo esté estructurada tu aplicación. Las aplicaciones robustas a menudo usan frameworks de inyección de dependencias para gestionar ciclos de vida de servicios y asegurar disponibilidad.

=== "Go"

    ```go
    import (
      "context"
      "log"

      "google.golang.org/adk/agent/llmagent"
      "google.golang.org/adk/artifactservice"
      "google.golang.org/adk/llm/gemini"
      "google.golang.org/adk/runner"
      "google.golang.org/adk/sessionservice"
      "google.golang.org/genai"
    )

    --8<-- "examples/go/snippets/artifacts/main.go:prerequisite"
    ```

=== "Java"

    En Java, instanciarías una implementación de `BaseArtifactService` y luego te asegurarías de que sea accesible para las partes de tu aplicación que gestionan artefactos. Esto a menudo se hace a través de inyección de dependencias o pasando explícitamente la instancia del servicio.

    ```java
    import com.google.adk.agents.LlmAgent;
    import com.google.adk.artifacts.InMemoryArtifactService; // O GcsArtifactService
    import com.google.adk.runner.Runner;
    import com.google.adk.sessions.InMemorySessionService;

    public class SampleArtifactAgent {

      public static void main(String[] args) {

        // Tu definición de agente
        LlmAgent agent = LlmAgent.builder()
            .name("my_agent")
            .model("gemini-2.0-flash")
            .build();

        // Instancia el servicio de artefactos deseado
        InMemoryArtifactService artifactService = new InMemoryArtifactService();

        // Proporciónalo al Runner
        Runner runner = new Runner(agent,
            "APP_NAME",
            artifactService, // El servicio debe proporcionarse aquí
            new InMemorySessionService());

      }
    }
    ```

### Accediendo a los Métodos

Los métodos de interacción con artefactos están disponibles directamente en instancias de `CallbackContext` (pasado a callbacks de agente y modelo) y `ToolContext` (pasado a callbacks de herramienta). Recuerda que `ToolContext` hereda de `CallbackContext`.

#### Guardando Artefactos

*   **Ejemplo de Código:**

    === "Python"

        ```python
        import google.genai.types as types
        from google.adk.agents.callback_context import CallbackContext # O ToolContext

        async def save_generated_report_py(context: CallbackContext, report_bytes: bytes):
            """Guarda bytes de informe PDF generado como un artefacto."""
            report_artifact = types.Part.from_bytes(
                data=report_bytes,
                mime_type="application/pdf"
            )
            filename = "generated_report.pdf"

            try:
                version = await context.save_artifact(filename=filename, artifact=report_artifact)
                print(f"Successfully saved Python artifact '{filename}' as version {version}.")
                # El evento generado después de este callback contendrá:
                # event.actions.artifact_delta == {"generated_report.pdf": version}
            except ValueError as e:
                print(f"Error saving Python artifact: {e}. Is ArtifactService configured in Runner?")
            except Exception as e:
                # Maneja posibles errores de almacenamiento (por ejemplo, permisos de GCS)
                print(f"An unexpected error occurred during Python artifact save: {e}")

        # --- Concepto de Uso de Ejemplo (Python) ---
        # async def main_py():
        #   callback_context: CallbackContext = ... # obtener contexto
        #   report_data = b'...' # Asume que esto contiene los bytes del PDF
        #   await save_generated_report_py(callback_context, report_data)
        ```

    === "Typescript"

        ```typescript
        import type { Part } from '@google/genai';
        import { createPartFromBase64 } from '@google/genai';
        import { CallbackContext } from '@google/adk';

        async function saveGeneratedReport(context: CallbackContext, reportBytes: Uint8Array): Promise<void> {
            /**Guarda bytes de informe PDF generado como un artefacto.*/
            const reportArtifact: Part = createPartFromBase64(reportBytes.toString('base64'), "application/pdf");

            const filename = "generated_report.pdf";

            try {
                const version = await context.saveArtifact(filename, reportArtifact);
                console.log(`Successfully saved TypeScript artifact '${filename}' as version ${version}.`);
            } catch (e: any) {
                console.error(`Error saving TypeScript artifact: ${e.message}. Is ArtifactService configured in Runner?`);
            }
        }
        ```
    === "Go"

        ```go
        import (
          "log"

          "google.golang.org/adk/agent"
          "google.golang.org/adk/llm"
          "google.golang.org/genai"
        )

        --8<-- "examples/go/snippets/artifacts/main.go:saving-artifacts"
        ```

    === "Java"

        ```java
        import com.google.adk.agents.CallbackContext;
        import com.google.adk.artifacts.BaseArtifactService;
        import com.google.adk.artifacts.InMemoryArtifactService;
        import com.google.genai.types.Part;
        import java.nio.charset.StandardCharsets;

        public class SaveArtifactExample {

        public void saveGeneratedReport(CallbackContext callbackContext, byte[] reportBytes) {
        // Guarda bytes de informe PDF generado como un artefacto.
        Part reportArtifact = Part.fromBytes(reportBytes, "application/pdf");
        String filename = "generatedReport.pdf";

            callbackContext.saveArtifact(filename, reportArtifact);
            System.out.println("Successfully saved Java artifact '" + filename);
            // El evento generado después de este callback contendrá:
            // event().actions().artifactDelta == {"generated_report.pdf": version}
        }

        // --- Concepto de Uso de Ejemplo (Java) ---
        public static void main(String[] args) {
            BaseArtifactService service = new InMemoryArtifactService(); // O GcsArtifactService
            SaveArtifactExample myTool = new SaveArtifactExample();
            byte[] reportData = "...".getBytes(StandardCharsets.UTF_8); // Bytes del PDF
            CallbackContext callbackContext; // ... obtener callback context de tu aplicación
            myTool.saveGeneratedReport(callbackContext, reportData);
            // Debido a la naturaleza asíncrona, en una aplicación real, asegura que el programa espere o maneje la finalización.
          }
        }
        ```

#### Cargando Artefactos

*   **Ejemplo de Código:**

    === "Python"

        ```python
        import google.genai.types as types
        from google.adk.agents.callback_context import CallbackContext # O ToolContext

        async def process_latest_report_py(context: CallbackContext):
            """Carga el último artefacto de informe y procesa sus datos."""
            filename = "generated_report.pdf"
            try:
                # Carga la última versión
                report_artifact = await context.load_artifact(filename=filename)

                if report_artifact and report_artifact.inline_data:
                    print(f"Successfully loaded latest Python artifact '{filename}'.")
                    print(f"MIME Type: {report_artifact.inline_data.mime_type}")
                    # Procesa el report_artifact.inline_data.data (bytes)
                    pdf_bytes = report_artifact.inline_data.data
                    print(f"Report size: {len(pdf_bytes)} bytes.")
                    # ... procesamiento adicional ...
                else:
                    print(f"Python artifact '{filename}' not found.")

                # Ejemplo: Carga una versión específica (si existe la versión 0)
                # specific_version_artifact = await context.load_artifact(filename=filename, version=0)
                # if specific_version_artifact:
                #     print(f"Loaded version 0 of '{filename}'.")

            except ValueError as e:
                print(f"Error loading Python artifact: {e}. Is ArtifactService configured?")
            except Exception as e:
                # Maneja posibles errores de almacenamiento
                print(f"An unexpected error occurred during Python artifact load: {e}")

        # --- Concepto de Uso de Ejemplo (Python) ---
        # async def main_py():
        #   callback_context: CallbackContext = ... # obtener contexto
        #   await process_latest_report_py(callback_context)
        ```

    === "Typescript"

        ```typescript
        import { CallbackContext } from '@google/adk';

        async function processLatestReport(context: CallbackContext): Promise<void> {
            /**Carga el último artefacto de informe y procesa sus datos.*/
            const filename = "generated_report.pdf";
            try {
                // Carga la última versión
                const reportArtifact = await context.loadArtifact(filename);

                if (reportArtifact?.inlineData) {
                    console.log(`Successfully loaded latest TypeScript artifact '${filename}'.`);
                    console.log(`MIME Type: ${reportArtifact.inlineData.mimeType}`);
                    // Procesa el reportArtifact.inlineData.data (cadena base64)
                    const pdfData = Buffer.from(reportArtifact.inlineData.data, 'base64');
                    console.log(`Report size: ${pdfData.length} bytes.`);
                    // ... procesamiento adicional ...
                } else {
                    console.log(`TypeScript artifact '${filename}' not found.`);
                }

            } catch (e: any) {
                console.error(`Error loading TypeScript artifact: ${e.message}. Is ArtifactService configured?`);
            }
        }
        ```

    === "Go"

        ```go
        import (
          "log"

          "google.golang.org/adk/agent"
          "google.golang.org/adk/llm"
        )

        --8<-- "examples/go/snippets/artifacts/main.go:loading-artifacts"
        ```

    === "Java"

        ```java
        import com.google.adk.artifacts.BaseArtifactService;
        import com.google.genai.types.Part;
        import io.reactivex.rxjava3.core.MaybeObserver;
        import io.reactivex.rxjava3.disposables.Disposable;
        import java.util.Optional;

        public class MyArtifactLoaderService {

            private final BaseArtifactService artifactService;
            private final String appName;

            public MyArtifactLoaderService(BaseArtifactService artifactService, String appName) {
                this.artifactService = artifactService;
                this.appName = appName;
            }

            public void processLatestReportJava(String userId, String sessionId, String filename) {
                // Carga la última versión pasando Optional.empty() para la versión
                artifactService
                        .loadArtifact(appName, userId, sessionId, filename, Optional.empty())
                        .subscribe(
                                new MaybeObserver<Part>() {
                                    @Override
                                    public void onSubscribe(Disposable d) {
                                        // Opcional: manejar suscripción
                                    }

                                    @Override
                                    public void onSuccess(Part reportArtifact) {
                                        System.out.println(
                                                "Successfully loaded latest Java artifact '" + filename + "'.");
                                        reportArtifact
                                                .inlineData()
                                                .ifPresent(
                                                        blob -> {
                                                            System.out.println(
                                                                    "MIME Type: " + blob.mimeType().orElse("N/A"));
                                                            byte[] pdfBytes = blob.data().orElse(new byte[0]);
                                                            System.out.println("Report size: " + pdfBytes.length + " bytes.");
                                                            // ... procesamiento adicional de pdfBytes ...
                                                        });
                                    }

                                    @Override
                                    public void onError(Throwable e) {
                                        // Maneja posibles errores de almacenamiento u otras excepciones
                                        System.err.println(
                                                "An error occurred during Java artifact load for '"
                                                        + filename
                                                        + "': "
                                                        + e.getMessage());
                                    }

                                    @Override
                                    public void onComplete() {
                                        // Llamado si el artefacto (última versión) no se encuentra
                                        System.out.println("Java artifact '" + filename + "' not found.");
                                    }
                                });

                // Ejemplo: Carga una versión específica (por ejemplo, versión 0)
                /*
                artifactService.loadArtifact(appName, userId, sessionId, filename, Optional.of(0))
                    .subscribe(part -> {
                        System.out.println("Loaded version 0 of Java artifact '" + filename + "'.");
                    }, throwable -> {
                        System.err.println("Error loading version 0 of '" + filename + "': " + throwable.getMessage());
                    }, () -> {
                        System.out.println("Version 0 of Java artifact '" + filename + "' not found.");
                    });
                */
            }

            // --- Concepto de Uso de Ejemplo (Java) ---
            public static void main(String[] args) {
                // BaseArtifactService service = new InMemoryArtifactService(); // O GcsArtifactService
                // MyArtifactLoaderService loader = new MyArtifactLoaderService(service, "myJavaApp");
                // loader.processLatestReportJava("user123", "sessionABC", "java_report.pdf");
                // Debido a la naturaleza asíncrona, en una aplicación real, asegura que el programa espere o maneje la finalización.
            }
        }
        ```

#### Listando Nombres de Archivo de Artefactos

*   **Ejemplo de Código:**

    === "Python"

        ```python
        from google.adk.tools.tool_context import ToolContext

        def list_user_files_py(tool_context: ToolContext) -> str:
            """Herramienta para listar artefactos disponibles para el usuario."""
            try:
                available_files = await tool_context.list_artifacts()
                if not available_files:
                    return "You have no saved artifacts."
                else:
                    # Formatea la lista para el usuario/LLM
                    file_list_str = "\n".join([f"- {fname}" for fname in available_files])
                    return f"Here are your available Python artifacts:\n{file_list_str}"
            except ValueError as e:
                print(f"Error listing Python artifacts: {e}. Is ArtifactService configured?")
                return "Error: Could not list Python artifacts."
            except Exception as e:
                print(f"An unexpected error occurred during Python artifact list: {e}")
                return "Error: An unexpected error occurred while listing Python artifacts."

        # Esta función típicamente se envolvería en un FunctionTool
        # from google.adk.tools import FunctionTool
        # list_files_tool = FunctionTool(func=list_user_files_py)
        ```

    === "Typescript"

        ```typescript
        import { ToolContext } from '@google/adk';

        async function listUserFiles(toolContext: ToolContext): Promise<string> {
            /**Herramienta para listar artefactos disponibles para el usuario.*/
            try {
                const availableFiles = await toolContext.listArtifacts();
                if (!availableFiles || availableFiles.length === 0) {
                    return "You have no saved artifacts.";
                } else {
                    // Formatea la lista para el usuario/LLM
                    const fileListStr = availableFiles.map(fname => `- ${fname}`).join("\n");
                    return `Here are your available TypeScript artifacts:\n${fileListStr}`;
                }
            } catch (e: any) {
                console.error(`Error listing TypeScript artifacts: ${e.message}. Is ArtifactService configured?`);
                return "Error: Could not list TypeScript artifacts.";
            }
        }
        ```

    === "Go"

        ```go
        import (
          "fmt"
          "log"
          "strings"

          "google.golang.org/adk/agent"
          "google.golang.org/adk/llm"
          "google.golang.org/genai"
        )

        --8<-- "examples/go/snippets/artifacts/main.go:listing-artifacts"
        ```

    === "Java"

        ```java
        import com.google.adk.artifacts.BaseArtifactService;
        import com.google.adk.artifacts.ListArtifactsResponse;
        import com.google.common.collect.ImmutableList;
        import io.reactivex.rxjava3.core.SingleObserver;
        import io.reactivex.rxjava3.disposables.Disposable;

        public class MyArtifactListerService {

            private final BaseArtifactService artifactService;
            private final String appName;

            public MyArtifactListerService(BaseArtifactService artifactService, String appName) {
                this.artifactService = artifactService;
                this.appName = appName;
            }

            // Método de ejemplo que podría ser llamado por una herramienta o lógica de agente
            public void listUserFilesJava(String userId, String sessionId) {
                artifactService
                        .listArtifactKeys(appName, userId, sessionId)
                        .subscribe(
                                new SingleObserver<ListArtifactsResponse>() {
                                    @Override
                                    public void onSubscribe(Disposable d) {
                                        // Opcional: manejar suscripción
                                    }

                                    @Override
                                    public void onSuccess(ListArtifactsResponse response) {
                                        ImmutableList<String> availableFiles = response.filenames();
                                        if (availableFiles.isEmpty()) {
                                            System.out.println(
                                                    "User "
                                                            + userId
                                                            + " in session "
                                                            + sessionId
                                                            + " has no saved Java artifacts.");
                                        } else {
                                            StringBuilder fileListStr =
                                                    new StringBuilder(
                                                            "Here are the available Java artifacts for user "
                                                                    + userId
                                                                    + " in session "
                                                                    + sessionId
                                                                    + ":\n");
                                            for (String fname : availableFiles) {
                                                fileListStr.append("- ").append(fname).append("\n");
                                            }
                                            System.out.println(fileListStr.toString());
                                        }
                                    }

                                    @Override
                                    public void onError(Throwable e) {
                                        System.err.println(
                                                "Error listing Java artifacts for user "
                                                        + userId
                                                        + " in session "
                                                        + sessionId
                                                        + ": "
                                                        + e.getMessage());
                                        // En una aplicación real, podrías devolver un mensaje de error al usuario/LLM
                                    }
                                });
            }

            // --- Concepto de Uso de Ejemplo (Java) ---
            public static void main(String[] args) {
                // BaseArtifactService service = new InMemoryArtifactService(); // O GcsArtifactService
                // MyArtifactListerService lister = new MyArtifactListerService(service, "myJavaApp");
                // lister.listUserFilesJava("user123", "sessionABC");
                // Debido a la naturaleza asíncrona, en una aplicación real, asegura que el programa espere o maneje la finalización.
            }
        }
        ```

Estos métodos para guardar, cargar y listar proporcionan una forma conveniente y consistente de gestionar la persistencia de datos binarios dentro de ADK, ya sea usando objetos de contexto de Python o interactuando directamente con el `BaseArtifactService` en Java, independientemente de la implementación de almacenamiento backend elegida.

## Implementaciones Disponibles

ADK proporciona implementaciones concretas de la interfaz `BaseArtifactService`, ofreciendo diferentes backends de almacenamiento adecuados para varias etapas de desarrollo y necesidades de implementación. Estas implementaciones manejan los detalles de almacenar, versionar y recuperar datos de artefactos basándose en el `app_name`, `user_id`, `session_id` y `filename` (incluyendo el prefijo de espacio de nombres `user:`).

### InMemoryArtifactService

*   **Mecanismo de Almacenamiento:**
    *   Python: Usa un diccionario de Python (`self.artifacts`) mantenido en la memoria de la aplicación. Las claves del diccionario representan la ruta del artefacto, y los valores son listas de `types.Part`, donde cada elemento de lista es una versión.
    *   Java: Usa instancias de `HashMap` anidadas (`private final Map<String, Map<String, Map<String, Map<String, List<Part>>>>> artifacts;`) mantenidas en memoria. Las claves en cada nivel son `appName`, `userId`, `sessionId` y `filename` respectivamente. La `List<Part>` más interna almacena las versiones del artefacto, donde el índice de lista corresponde al número de versión.
*   **Características Clave:**
    *   **Simplicidad:** No requiere configuración externa o dependencias más allá de la biblioteca central de ADK.
    *   **Velocidad:** Las operaciones típicamente son muy rápidas ya que involucran búsquedas de diccionario/mapa en memoria y manipulaciones de listas.
    *   **Efímero:** Todos los artefactos almacenados se **pierden** cuando el proceso de la aplicación termina. Los datos no persisten entre reinicios de aplicación.
*   **Casos de Uso:**
    *   Ideal para desarrollo local y pruebas donde no se requiere persistencia.
    *   Adecuado para demostraciones de corta duración o escenarios donde los datos de artefactos son puramente temporales dentro de una sola ejecución de la aplicación.
*   **Instanciación:**

    === "Python"

        ```python
        from google.adk.artifacts import InMemoryArtifactService

        # Simplemente instancia la clase
        in_memory_service_py = InMemoryArtifactService()

        # Luego pásala al Runner
        # runner = Runner(..., artifact_service=in_memory_service_py)
        ```

    === "Typescript"

        ```typescript
        import { InMemoryArtifactService } from '@google/adk';

        // Simplemente instancia la clase
        const inMemoryService = new InMemoryArtifactService();

        // Esta instancia entonces se proporcionaría a tu Runner.
        // const runner = new InMemoryRunner({
        //     /* otros servicios */,
        //     artifactService: inMemoryService
        // });
        ```

    === "Go"

        ```go
        import (
          "google.golang.org/adk/artifactservice"
        )

        --8<-- "examples/go/snippets/artifacts/main.go:in-memory-service"
        ```

    === "Java"

        ```java
        import com.google.adk.artifacts.BaseArtifactService;
        import com.google.adk.artifacts.InMemoryArtifactService;

        public class InMemoryServiceSetup {
            public static void main(String[] args) {
                // Simplemente instancia la clase
                BaseArtifactService inMemoryServiceJava = new InMemoryArtifactService();

                System.out.println("InMemoryArtifactService (Java) instantiated: " + inMemoryServiceJava.getClass().getName());

                // Esta instancia entonces se proporcionaría a tu Runner.
                // Runner runner = new Runner(
                //     /* otros servicios */,
                //     inMemoryServiceJava
                // );
            }
        }
        ```

### GcsArtifactService


*   **Mecanismo de Almacenamiento:** Aprovecha Google Cloud Storage (GCS) para almacenamiento persistente de artefactos. Cada versión de un artefacto se almacena como un objeto separado (blob) dentro de un bucket de GCS especificado.
*   **Convención de Nomenclatura de Objetos:** Construye nombres de objetos de GCS (nombres de blob) usando una estructura de ruta jerárquica.
*   **Características Clave:**
    *   **Persistencia:** Los artefactos almacenados en GCS persisten a través de reinicios y despliegues de aplicación.
    *   **Escalabilidad:** Aprovecha la escalabilidad y durabilidad de Google Cloud Storage.
    *   **Versionado:** Almacena explícitamente cada versión como un objeto de GCS distinto. El método `saveArtifact` en `GcsArtifactService`.
    *   **Permisos Requeridos:** El entorno de la aplicación necesita credenciales apropiadas (por ejemplo, Application Default Credentials) y permisos IAM para leer y escribir en el bucket de GCS especificado.
*   **Casos de Uso:**
    *   Entornos de producción que requieren almacenamiento persistente de artefactos.
    *   Escenarios donde los artefactos necesitan ser compartidos entre diferentes instancias de aplicación o servicios (accediendo al mismo bucket de GCS).
    *   Aplicaciones que necesitan almacenamiento y recuperación a largo plazo de datos de usuario o sesión.
*   **Instanciación:**

    === "Python"

        ```python
        from google.adk.artifacts import GcsArtifactService

        # Especifica el nombre del bucket de GCS
        gcs_bucket_name_py = "your-gcs-bucket-for-adk-artifacts" # Reemplaza con el nombre de tu bucket

        try:
            gcs_service_py = GcsArtifactService(bucket_name=gcs_bucket_name_py)
            print(f"Python GcsArtifactService initialized for bucket: {gcs_bucket_name_py}")
            # Asegura que tu entorno tenga credenciales para acceder a este bucket.
            # por ejemplo, vía Application Default Credentials (ADC)

            # Luego pásala al Runner
            # runner = Runner(..., artifact_service=gcs_service_py)

        except Exception as e:
            # Captura posibles errores durante la inicialización del cliente de GCS (por ejemplo, problemas de autenticación)
            print(f"Error initializing Python GcsArtifactService: {e}")
            # Maneja el error apropiadamente - tal vez recurre a InMemory o lanza
        ```

    === "Java"

        ```java
        --8<-- "examples/java/snippets/src/main/java/artifacts/GcsServiceSetup.java:full_code"
        ```

Elegir la implementación apropiada de `ArtifactService` depende de los requisitos de tu aplicación para persistencia de datos, escalabilidad y entorno operacional.

## Mejores Prácticas

Para usar artefactos de manera efectiva y mantenible:

* **Elige el Servicio Correcto:** Usa `InMemoryArtifactService` para prototipado rápido, pruebas y escenarios donde no se necesita persistencia. Usa `GcsArtifactService` (o implementa tu propio `BaseArtifactService` para otros backends) para entornos de producción que requieren persistencia de datos y escalabilidad.
* **Nombres de Archivo Significativos:** Usa nombres de archivo claros y descriptivos. Incluir extensiones relevantes (`.pdf`, `.png`, `.wav`) ayuda a los humanos a entender el contenido, aunque el `mime_type` dicta el manejo programático. Establece convenciones para nombres de artefactos temporales vs. persistentes.
* **Especifica Tipos MIME Correctos:** Siempre proporciona un `mime_type` preciso al crear el `types.Part` para `save_artifact`. Esto es crítico para aplicaciones o herramientas que posteriormente hacen `load_artifact` para interpretar correctamente los datos `bytes`. Usa tipos MIME estándar de IANA cuando sea posible.
* **Entiende el Versionado:** Recuerda que `load_artifact()` sin un argumento `version` específico recupera la versión *más reciente*. Si tu lógica depende de una versión histórica específica de un artefacto, asegúrate de proporcionar el número de versión entero al cargar.
* **Usa el Espacio de Nombres (`user:`) Deliberadamente:** Solo usa el prefijo `"user:"` para nombres de archivo cuando los datos realmente pertenezcan al usuario y deban ser accesibles a través de todas sus sesiones. Para datos específicos de una sola conversación o sesión, usa nombres de archivo regulares sin el prefijo.
* **Manejo de Errores:**
    * Siempre verifica si un `artifact_service` está realmente configurado antes de llamar a métodos de contexto (`save_artifact`, `load_artifact`, `list_artifacts`) – lanzarán un `ValueError` si el servicio es `None`.
    * Verifica el valor de retorno de `load_artifact`, ya que será `None` si el artefacto o versión no existe. No asumas que siempre devuelve un `Part`.
    * Prepárate para manejar excepciones del servicio de almacenamiento subyacente, especialmente con `GcsArtifactService` (por ejemplo, `google.api_core.exceptions.Forbidden` para problemas de permisos, `NotFound` si el bucket no existe, errores de red).
* **Consideraciones de Tamaño:** Los artefactos son adecuados para tamaños de archivo típicos, pero ten en cuenta los posibles costos e impactos de rendimiento con archivos extremadamente grandes, especialmente con almacenamiento en la nube. `InMemoryArtifactService` puede consumir memoria significativa si almacena muchos artefactos grandes. Evalúa si datos muy grandes podrían manejarse mejor a través de enlaces directos de GCS u otras soluciones de almacenamiento especializadas en lugar de pasar arreglos de bytes completos en memoria.
* **Estrategia de Limpieza:** Para almacenamiento persistente como `GcsArtifactService`, los artefactos permanecen hasta que se eliminen explícitamente. Si los artefactos representan datos temporales o tienen un tiempo de vida limitado, implementa una estrategia de limpieza. Esto podría involucrar:
    * Usar políticas de ciclo de vida de GCS en el bucket.
    * Construir herramientas específicas o funciones administrativas que utilicen el método `artifact_service.delete_artifact` (nota: delete *no* se expone vía objetos de contexto por seguridad).
    * Gestionar cuidadosamente los nombres de archivo para permitir eliminación basada en patrones si es necesario.