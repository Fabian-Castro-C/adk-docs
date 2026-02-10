# Construye un agente con streaming usando Java

Esta guía de inicio rápido te guiará a través del proceso de creación de un agente básico y el aprovechamiento del ADK Streaming con Java para facilitar interacciones de voz bidireccionales de baja latencia.

Comenzarás configurando tu entorno de Java y Maven, estructurando tu proyecto y definiendo las dependencias necesarias. Después de esto, crearás un simple `ScienceTeacherAgent`, probarás sus capacidades de streaming basado en texto usando la Dev UI, y luego progresarás hacia la habilitación de comunicación de audio en vivo, transformando tu agente en una aplicación interactiva impulsada por voz.

## **Crea tu primer agente** {#create-your-first-agent}

### **Prerrequisitos**

* En esta guía de inicio, programarás en Java. Verifica si **Java** está instalado en tu máquina. Idealmente, deberías estar usando Java 17 o superior (puedes verificarlo escribiendo **java \-version**)

* También estarás usando la herramienta de construcción **Maven** para Java. Así que asegúrate de tener [Maven instalado](https://maven.apache.org/install.html) en tu máquina antes de continuar (este es el caso para Cloud Top o Cloud Shell, pero no necesariamente para tu laptop).

### **Prepara la estructura del proyecto**

Para comenzar con ADK Java, creemos un proyecto Maven con la siguiente estructura de directorios:

```
adk-agents/
├── pom.xml
└── src/
    └── main/
        └── java/
            └── agents/
                └── ScienceTeacherAgent.java
```

Sigue las instrucciones en la página de [Instalación](../../get-started/installation.md) para agregar `pom.xml` para usar el paquete ADK.

!!! Note
    Siéntete libre de usar el nombre que prefieras para el directorio raíz de tu proyecto (en lugar de adk-agents)

### **Ejecutando una compilación**

Veamos si Maven está satisfecho con esta construcción, ejecutando una compilación (comando **mvn compile**):

```shell
$ mvn compile
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------< adk-agents:adk-agents >--------------------
[INFO] Building adk-agents 1.0-SNAPSHOT
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- resources:3.3.1:resources (default-resources) @ adk-demo ---
[INFO] skip non existing resourceDirectory /home/user/adk-demo/src/main/resources
[INFO]
[INFO] --- compiler:3.13.0:compile (default-compile) @ adk-demo ---
[INFO] Nothing to compile - all classes are up to date.
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.347 s
[INFO] Finished at: 2025-05-06T15:38:08Z
[INFO] ------------------------------------------------------------------------
```

¡Parece que el proyecto está configurado correctamente para la compilación!

### **Creando un agente**

Crea el archivo **ScienceTeacherAgent.java** bajo el directorio `src/main/java/agents/` con el siguiente contenido:

```java
package samples.liveaudio;

import com.google.adk.agents.BaseAgent;
import com.google.adk.agents.LlmAgent;

/** Agente profesor de ciencias. */
public class ScienceTeacherAgent {

  // Campo esperado por la Dev UI para cargar el agente dinámicamente
  // (el agente debe ser inicializado en el momento de la declaración)
  public static final BaseAgent ROOT_AGENT = initAgent();

  // Por favor completa el ID del modelo más reciente que soporte la API en vivo desde
  // https://google.github.io/get-started/streaming/quickstart-streaming/#supported-models
  public static BaseAgent initAgent() {
    return LlmAgent.builder()
        .name("science-app")
        .description("Science teacher agent")
        .model("...") // Por favor completa el ID del modelo más reciente para la API en vivo
        .instruction("""
            You are a helpful science teacher that explains
            science concepts to kids and teenagers.
            """)
        .build();
  }
}
```

Usaremos `Dev UI` para ejecutar este agente más adelante. Para que la herramienta reconozca automáticamente el agente, su clase Java debe cumplir con las siguientes dos reglas:

* El agente debe almacenarse en una variable global **public static** llamada **ROOT\_AGENT** de tipo **BaseAgent** e inicializada en el momento de la declaración.
* La definición del agente tiene que ser un método **static** para que pueda ser cargado durante la inicialización de la clase por el classloader de compilación dinámica.

## **Ejecuta el agente con Dev UI** {#run-agent-with-adk-web-server}

`Dev UI` es un servidor web donde puedes ejecutar y probar rápidamente tus agentes con fines de desarrollo, sin construir tu propia aplicación de interfaz de usuario para los agentes.

### **Define variables de entorno**

Para ejecutar el servidor, necesitarás exportar dos variables de entorno:

* una clave de Gemini que puedes [obtener de AI Studio](https://ai.google.dev/gemini-api/docs/api-key),
* una variable para especificar que no estamos usando Vertex AI esta vez.

```shell
export GOOGLE_GENAI_USE_VERTEXAI=FALSE
export GOOGLE_API_KEY=YOUR_API_KEY
```

### **Ejecuta Dev UI**

Ejecuta el siguiente comando desde la terminal para lanzar la Dev UI.

```console title="terminal"
mvn exec:java \
    -Dexec.mainClass="com.google.adk.web.AdkWebServer" \
    -Dexec.args="--adk.agents.source-dir=." \
    -Dexec.classpathScope="compile"
```

**Paso 1:** Abre la URL proporcionada (usualmente `http://localhost:8080` o
`http://127.0.0.1:8080`) directamente en tu navegador.

**Paso 2.** En la esquina superior izquierda de la interfaz de usuario, puedes seleccionar tu agente en
el menú desplegable. Selecciona "science-app".

!!!note "Solución de problemas"

    Si no ves "science-app" en el menú desplegable, asegúrate de que
    estés ejecutando el comando `mvn` desde la raíz de tu proyecto maven.

!!! warning "Precaución: ADK Web solo para desarrollo"

    ADK Web ***no está destinado para uso en implementaciones de producción***. Debes
    usar ADK Web solo con fines de desarrollo y depuración.

## Prueba Dev UI con voz y video

Con tu navegador favorito, navega a: [http://127.0.0.1:8080/](http://127.0.0.1:8080/)

Deberías ver la siguiente interfaz:

![Dev UI](../../assets/quickstart-streaming-devui.png)

Haz clic en el botón del micrófono para habilitar la entrada de voz, y haz una pregunta `What's the electron?` con la voz. Escucharás la respuesta en voz en tiempo real.

Para probar con video, recarga el navegador web, haz clic en el botón de la cámara para habilitar la entrada de video, y haz preguntas como "What do you see?". El agente responderá lo que ve en la entrada de video.

### Advertencia

- No puedes usar el chat de texto con los modelos de audio nativo. Verás errores al ingresar mensajes de texto en `adk web`.

### Detén la herramienta

Detén la herramienta presionando `Ctrl-C` en la consola.

## **Ejecuta el agente con una aplicación de audio en vivo personalizada** {#run-agent-with-live-audio}

Ahora, probemos el streaming de audio con el agente y una aplicación de audio en vivo personalizada.

### **Un archivo de construcción Maven pom.xml para Audio en Vivo**

Reemplaza tu pom.xml existente con el siguiente.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.google.adk.samples</groupId>
  <artifactId>google-adk-sample-live-audio</artifactId>
  <version>0.1.0</version>
  <name>Google ADK - Sample - Live Audio</name>
  <description>
    A sample application demonstrating a live audio conversation using ADK,
    runnable via samples.liveaudio.LiveAudioRun.
  </description>
  <packaging>jar</packaging>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>17</java.version>
    <auto-value.version>1.11.0</auto-value.version>
    <!-- Clase principal para exec-maven-plugin -->
    <exec.mainClass>samples.liveaudio.LiveAudioRun</exec.mainClass>
    <google-adk.version>0.1.0</google-adk.version>
  </properties>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>com.google.cloud</groupId>
        <artifactId>libraries-bom</artifactId>
        <version>26.53.0</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <dependency>
      <groupId>com.google.adk</groupId>
      <artifactId>google-adk</artifactId>
      <version>${google-adk.version}</version>
    </dependency>
    <dependency>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
      <version>1.2</version> <!-- O usa una propiedad si está definida en un POM padre -->
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.13.0</version>
        <configuration>
          <source>${java.version}</source>
          <target>${java.version}</target>
          <parameters>true</parameters>
          <annotationProcessorPaths>
            <path>
              <groupId>com.google.auto.value</groupId>
              <artifactId>auto-value</artifactId>
              <version>${auto-value.version}</version>
            </path>
          </annotationProcessorPaths>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>build-helper-maven-plugin</artifactId>
        <version>3.6.0</version>
        <executions>
          <execution>
            <id>add-source</id>
            <phase>generate-sources</phase>
            <goals>
              <goal>add-source</goal>
            </goals>
            <configuration>
              <sources>
                <source>.</source>
              </sources>
            </configuration>
          </execution>
        </executions>
      </plugin>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>3.2.0</version>
        <configuration>
          <mainClass>${exec.mainClass}</mainClass>
          <classpathScope>runtime</classpathScope>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

### **Creando la herramienta Live Audio Run**

Crea el archivo **LiveAudioRun.java** bajo el directorio `src/main/java/` con el siguiente contenido. Esta herramienta ejecuta el agente con entrada y salida de audio en vivo.

```java

package samples.liveaudio;

import com.google.adk.agents.LiveRequestQueue;
import com.google.adk.agents.RunConfig;
import com.google.adk.events.Event;
import com.google.adk.runner.Runner;
import com.google.adk.sessions.InMemorySessionService;
import com.google.common.collect.ImmutableList;
import com.google.genai.types.Blob;
import com.google.genai.types.Modality;
import com.google.genai.types.PrebuiltVoiceConfig;
import com.google.genai.types.Content;
import com.google.genai.types.Part;
import com.google.genai.types.SpeechConfig;
import com.google.genai.types.VoiceConfig;
import io.reactivex.rxjava3.core.Flowable;
import java.io.ByteArrayOutputStream;
import java.io.InputStream;
import java.net.URL;
import javax.sound.sampled.AudioFormat;
import javax.sound.sampled.AudioInputStream;
import javax.sound.sampled.AudioSystem;
import javax.sound.sampled.DataLine;
import javax.sound.sampled.LineUnavailableException;
import javax.sound.sampled.Mixer;
import javax.sound.sampled.SourceDataLine;
import javax.sound.sampled.TargetDataLine;
import java.util.UUID;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicBoolean;
import agents.ScienceTeacherAgent;

/** Clase principal para demostrar la ejecución del {@link LiveAudioAgent} para una conversación de voz. */
public final class LiveAudioRun {
  private final String userId;
  private final String sessionId;
  private final Runner runner;

  private static final javax.sound.sampled.AudioFormat MIC_AUDIO_FORMAT =
      new javax.sound.sampled.AudioFormat(16000.0f, 16, 1, true, false);

  private static final javax.sound.sampled.AudioFormat SPEAKER_AUDIO_FORMAT =
      new javax.sound.sampled.AudioFormat(24000.0f, 16, 1, true, false);

  private static final int BUFFER_SIZE = 4096;

  public LiveAudioRun() {
    this.userId = "test_user";
    String appName = "LiveAudioApp";
    this.sessionId = UUID.randomUUID().toString();

    InMemorySessionService sessionService = new InMemorySessionService();
    this.runner = new Runner(ScienceTeacherAgent.ROOT_AGENT, appName, null, sessionService);

    ConcurrentMap<String, Object> initialState = new ConcurrentHashMap<>();
    var unused =
        sessionService.createSession(appName, userId, initialState, sessionId).blockingGet();
  }

  private void runConversation() throws Exception {
    System.out.println("Initializing microphone input and speaker output...");

    RunConfig runConfig =
        RunConfig.builder()
            .setStreamingMode(RunConfig.StreamingMode.BIDI)
            .setResponseModalities(ImmutableList.of(new Modality("AUDIO")))
            .setSpeechConfig(
                SpeechConfig.builder()
                    .voiceConfig(
                        VoiceConfig.builder()
                            .prebuiltVoiceConfig(
                                PrebuiltVoiceConfig.builder().voiceName("Aoede").build())
                            .build())
                    .languageCode("en-US")
                    .build())
            .build();

    LiveRequestQueue liveRequestQueue = new LiveRequestQueue();

    Flowable<Event> eventStream =
        this.runner.runLive(
            runner.sessionService().createSession(userId, sessionId).blockingGet(),
            liveRequestQueue,
            runConfig);

    AtomicBoolean isRunning = new AtomicBoolean(true);
    AtomicBoolean conversationEnded = new AtomicBoolean(false);
    ExecutorService executorService = Executors.newFixedThreadPool(2);

    // Tarea para capturar entrada de micrófono
    Future<?> microphoneTask =
        executorService.submit(() -> captureAndSendMicrophoneAudio(liveRequestQueue, isRunning));

    // Tarea para procesar respuestas del agente y reproducir audio
    Future<?> outputTask =
        executorService.submit(
            () -> {
              try {
                processAudioOutput(eventStream, isRunning, conversationEnded);
              } catch (Exception e) {
                System.err.println("Error processing audio output: " + e.getMessage());
                e.printStackTrace();
                isRunning.set(false);
              }
            });

    // Esperar a que el usuario presione Enter para detener la conversación
    System.out.println("Conversation started. Press Enter to stop...");
    System.in.read();

    System.out.println("Ending conversation...");
    isRunning.set(false);

    try {
      // Dar algo de tiempo para que el procesamiento en curso se complete
      microphoneTask.get(2, TimeUnit.SECONDS);
      outputTask.get(2, TimeUnit.SECONDS);
    } catch (Exception e) {
      System.out.println("Stopping tasks...");
    }

    liveRequestQueue.close();
    executorService.shutdownNow();
    System.out.println("Conversation ended.");
  }

  private void captureAndSendMicrophoneAudio(
      LiveRequestQueue liveRequestQueue, AtomicBoolean isRunning) {
    TargetDataLine micLine = null;
    try {
      DataLine.Info info = new DataLine.Info(TargetDataLine.class, MIC_AUDIO_FORMAT);
      if (!AudioSystem.isLineSupported(info)) {
        System.err.println("Microphone line not supported!");
        return;
      }

      micLine = (TargetDataLine) AudioSystem.getLine(info);
      micLine.open(MIC_AUDIO_FORMAT);
      micLine.start();

      System.out.println("Microphone initialized. Start speaking...");

      byte[] buffer = new byte[BUFFER_SIZE];
      int bytesRead;

      while (isRunning.get()) {
        bytesRead = micLine.read(buffer, 0, buffer.length);

        if (bytesRead > 0) {
          byte[] audioChunk = new byte[bytesRead];
          System.arraycopy(buffer, 0, audioChunk, 0, bytesRead);

          Blob audioBlob = Blob.builder().data(audioChunk).mimeType("audio/pcm").build();

          liveRequestQueue.realtime(audioBlob);
        }
      }
    } catch (LineUnavailableException e) {
      System.err.println("Error accessing microphone: " + e.getMessage());
      e.printStackTrace();
    } finally {
      if (micLine != null) {
        micLine.stop();
        micLine.close();
      }
    }
  }

  private void processAudioOutput(
      Flowable<Event> eventStream, AtomicBoolean isRunning, AtomicBoolean conversationEnded) {
    SourceDataLine speakerLine = null;
    try {
      DataLine.Info info = new DataLine.Info(SourceDataLine.class, SPEAKER_AUDIO_FORMAT);
      if (!AudioSystem.isLineSupported(info)) {
        System.err.println("Speaker line not supported!");
        return;
      }

      final SourceDataLine finalSpeakerLine = (SourceDataLine) AudioSystem.getLine(info);
      finalSpeakerLine.open(SPEAKER_AUDIO_FORMAT);
      finalSpeakerLine.start();

      System.out.println("Speaker initialized.");

      for (Event event : eventStream.blockingIterable()) {
        if (!isRunning.get()) {
          break;
        }

        AtomicBoolean audioReceived = new AtomicBoolean(false);
        processEvent(event, audioReceived);

        event.content().ifPresent(content -> content.parts().ifPresent(parts -> parts.forEach(part -> playAudioData(part, finalSpeakerLine))));
      }

      speakerLine = finalSpeakerLine; // Asignar a la variable externa para limpieza en el bloque finally
    } catch (LineUnavailableException e) {
      System.err.println("Error accessing speaker: " + e.getMessage());
      e.printStackTrace();
    } finally {
      if (speakerLine != null) {
        speakerLine.drain();
        speakerLine.stop();
        speakerLine.close();
      }
      conversationEnded.set(true);
    }
  }

  private void playAudioData(Part part, SourceDataLine speakerLine) {
    part.inlineData()
        .ifPresent(
            inlineBlob ->
                inlineBlob
                    .data()
                    .ifPresent(
                        audioBytes -> {
                          if (audioBytes.length > 0) {
                            System.out.printf(
                                "Playing audio (%s): %d bytes%n",
                                inlineBlob.mimeType(),
                                audioBytes.length);
                            speakerLine.write(audioBytes, 0, audioBytes.length);
                          }
                        }));
  }

  private void processEvent(Event event, java.util.concurrent.atomic.AtomicBoolean audioReceived) {
    event
        .content()
        .ifPresent(
            content ->
                content
                    .parts()
                    .ifPresent(parts -> parts.forEach(part -> logReceivedAudioData(part, audioReceived))));
  }

  private void logReceivedAudioData(Part part, AtomicBoolean audioReceived) {
    part.inlineData()
        .ifPresent(
            inlineBlob ->
                inlineBlob
                    .data()
                    .ifPresent(
                        audioBytes -> {
                          if (audioBytes.length > 0) {
                            System.out.printf(
                                "    Audio (%s): received %d bytes.%n",
                                inlineBlob.mimeType(),
                                audioBytes.length);
                            audioReceived.set(true);
                          } else {
                            System.out.printf(
                                "    Audio (%s): received empty audio data.%n",
                                inlineBlob.mimeType());
                          }
                        }));
  }

  public static void main(String[] args) throws Exception {
    LiveAudioRun liveAudioRun = new LiveAudioRun();
    liveAudioRun.runConversation();
    System.out.println("Exiting Live Audio Run.");
  }
}
```

### **Ejecuta la herramienta Live Audio Run**

Para ejecutar la herramienta Live Audio Run, usa el siguiente comando en el directorio `adk-agents`:

```
mvn compile exec:java
```

Entonces deberías ver:

```
$ mvn compile exec:java
...
Initializing microphone input and speaker output...
Conversation started. Press Enter to stop...
Speaker initialized.
Microphone initialized. Start speaking...
```

Con este mensaje, la herramienta está lista para recibir entrada de voz. Habla con el agente con una pregunta como `What's the electron?`.

!!! Caution
    Cuando observes que el agente sigue hablando por sí mismo y no se detiene, intenta usar auriculares para suprimir el eco.

## **Resumen** {#summary}

El Streaming para ADK permite a los desarrolladores crear agentes capaces de comunicación de voz y video bidireccional de baja latencia, mejorando las experiencias interactivas. El artículo demuestra que el streaming de texto es una característica integrada de los Agentes ADK, que no requiere código específico adicional, mientras también muestra cómo implementar conversaciones de audio en vivo para la interacción de voz en tiempo real con un agente. Esto permite una comunicación más natural y dinámica, ya que los usuarios pueden hablar y escuchar al agente sin problemas.