# Inicio Rápido de Java para ADK

Esta guía te muestra cómo empezar a trabajar con Agent Development Kit
para Java. Antes de comenzar, asegúrate de tener instalado lo siguiente:

*   Java 17 o posterior
*   Maven 3.9 o posterior

## Crear un proyecto de agente

Crea un proyecto de agente con los siguientes archivos y estructura de directorios:

```none
my_agent/
    src/main/java/com/example/agent/
                        HelloTimeAgent.java # código principal del agente
                        AgentCliRunner.java # interfaz de línea de comandos
    pom.xml                                 # configuración del proyecto
    .env                                    # claves API o IDs de proyecto
```

??? tip "Crear esta estructura de proyecto usando la línea de comandos"

    === "Windows"

        ```console
        mkdir my_agent\src\main\java\com\example\agent
        type nul > my_agent\src\main\java\com\example\agent\HelloTimeAgent.java
        type nul > my_agent\src\main\java\com\example\agent\AgentCliRunner.java
        type nul > my_agent\pom.xml
        type nul > my_agent\.env
        ```

    === "MacOS / Linux"

        ```bash
        mkdir -p my_agent/src/main/java/com/example/agent && \
            touch my_agent/src/main/java/com/example/agent/HelloTimeAgent.java && \
            touch my_agent/src/main/java/com/example/agent/AgentCliRunner.java && \
            touch my_agent/pom.xml my_agent/.env
        ```

### Definir el código del agente

Crea el código para un agente básico, incluyendo una implementación simple de una
[Herramienta de Función](/tools-custom/function-tools/) de ADK, llamada `getCurrentTime()`.
Agrega el siguiente código al archivo `HelloTimeAgent.java` en el directorio de tu proyecto:

```java title="my_agent/src/main/java/com/example/agent/HelloTimeAgent.java"
package com.example.agent;

import com.google.adk.agents.BaseAgent;
import com.google.adk.agents.LlmAgent;
import com.google.adk.tools.Annotations.Schema;
import com.google.adk.tools.FunctionTool;

import java.util.Map;

public class HelloTimeAgent {

    public static BaseAgent ROOT_AGENT = initAgent();

    private static BaseAgent initAgent() {
        return LlmAgent.builder()
            .name("hello-time-agent")
            .description("Tells the current time in a specified city")
            .instruction("""
                You are a helpful assistant that tells the current time in a city.
                Use the 'getCurrentTime' tool for this purpose.
                """)
            .model("gemini-2.5-flash")
            .tools(FunctionTool.create(HelloTimeAgent.class, "getCurrentTime"))
            .build();
    }

    /** Implementación simulada de herramienta */
    @Schema(description = "Get the current time for a given city")
    public static Map<String, String> getCurrentTime(
        @Schema(name = "city", description = "Name of the city to get the time for") String city) {
        return Map.of(
            "city", city,
            "forecast", "The time is 10:30am."
        );
    }
}
```

!!! warning "Precaución: Compatibilidad con Gemini 3"

    ADK Java v0.3.0 e inferior no es compatible con
    [Gemini 3 Pro Preview](https://ai.google.dev/gemini-api/docs/models#gemini-3-pro)
    debido a cambios en la firma de pensamiento para el llamado de funciones. Usa Gemini 2.5
    o modelos inferiores en su lugar.

### Configurar proyecto y dependencias

Un proyecto de agente ADK requiere esta dependencia en tu
archivo de proyecto `pom.xml`:

```xml title="my_agent/pom.xml (partial)"
<dependencies>
    <dependency>
        <groupId>com.google.adk</groupId>
        <artifactId>adk-core</artifactId>
        <version>0.3.0</version>
    </dependency>
</dependencies>
```

Actualiza el archivo de proyecto `pom.xml` para incluir esta dependencia y
configuraciones adicionales con el siguiente código de configuración:

??? info "Configuración completa de `pom.xml` para el proyecto"
    El siguiente código muestra una configuración completa de `pom.xml` para
    este proyecto:

    ```xml title="my_agent/pom.xml"
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>

        <groupId>com.example.agent</groupId>
        <artifactId>adk-agents</artifactId>
        <version>1.0-SNAPSHOT</version>

        <!-- Especifica la versión de Java que usarás -->
        <properties>
            <maven.compiler.source>17</maven.compiler.source>
            <maven.compiler.target>17</maven.compiler.target>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        </properties>

        <dependencies>
            <!-- La dependencia principal de ADK -->
            <dependency>
                <groupId>com.google.adk</groupId>
                <artifactId>google-adk</artifactId>
                <version>0.3.0</version>
            </dependency>
            <!-- La interfaz web de desarrollo de ADK para depurar tu agente -->
            <dependency>
                <groupId>com.google.adk</groupId>
                <artifactId>google-adk-dev</artifactId>
                <version>0.3.0</version>
            </dependency>
        </dependencies>

    </project>
    ```

### Configurar tu clave API

Este proyecto usa la API de Gemini, que requiere una clave API. Si no
tienes ya una clave API de Gemini, crea una clave en Google AI Studio en la
página de [Claves API](https://aistudio.google.com/app/apikey).

En una ventana de terminal, escribe tu clave API en el archivo `.env` de tu proyecto
para establecer las variables de entorno:

=== "MacOS / Linux"

    ```bash title="Update: my_agent/.env"
    echo 'export GOOGLE_API_KEY="YOUR_API_KEY"' > .env
    ```

=== "Windows"

    ```console title="Update: my_agent/env.bat"
    echo 'set GOOGLE_API_KEY="YOUR_API_KEY"' > env.bat
    ```

??? tip "Usar otros modelos de IA con ADK"
    ADK soporta el uso de muchos modelos de IA generativa. Para más
    información sobre cómo configurar otros modelos en agentes ADK, consulta
    [Modelos y Autenticación](/agents/models).

### Crear una interfaz de línea de comandos para el agente

Crea una clase `AgentCliRunner.java` para permitirte ejecutar e interactuar con
`HelloTimeAgent` desde la línea de comandos. Este código muestra cómo crear un
objeto `RunConfig` para ejecutar el agente y un objeto `Session` para interactuar con el
agente en ejecución.

```java title="my_agent/src/main/java/com/example/agent/AgentCliRunner.java"
package com.example.agent;

import com.google.adk.agents.RunConfig;
import com.google.adk.events.Event;
import com.google.adk.runner.InMemoryRunner;
import com.google.adk.sessions.Session;
import com.google.genai.types.Content;
import com.google.genai.types.Part;
import io.reactivex.rxjava3.core.Flowable;
import java.util.Scanner;

import static java.nio.charset.StandardCharsets.UTF_8;

public class AgentCliRunner {

    public static void main(String[] args) {
        RunConfig runConfig = RunConfig.builder().build();
        InMemoryRunner runner = new InMemoryRunner(HelloTimeAgent.ROOT_AGENT);

        Session session = runner
                .sessionService()
                .createSession(runner.appName(), "user1234")
                .blockingGet();

        try (Scanner scanner = new Scanner(System.in, UTF_8)) {
            while (true) {
                System.out.print("\nYou > ");
                String userInput = scanner.nextLine();
                if ("quit".equalsIgnoreCase(userInput)) {
                    break;
                }

                Content userMsg = Content.fromParts(Part.fromText(userInput));
                Flowable<Event> events = runner.runAsync(session.userId(), session.id(), userMsg, runConfig);

                System.out.print("\nAgent > ");
                events.blockingForEach(event -> {
                    if (event.finalResponse()) {
                        System.out.println(event.stringifyContent());
                    }
                });
            }
        }
    }
}
```

## Ejecutar tu agente

Puedes ejecutar tu agente ADK usando la interfaz de línea de comandos interactiva
de la clase `AgentCliRunner` que definiste o la interfaz web de usuario de ADK proporcionada por
ADK usando la clase `AdkWebServer`. Ambas opciones te permiten probar e
interactuar con tu agente.

### Ejecutar con interfaz de línea de comandos

Ejecuta tu agente con la interfaz de línea de comandos de la clase `AgentCliRunner`
usando el siguiente comando Maven:

```console
# Recuerda cargar las claves y configuraciones: source .env OR env.bat
mvn compile exec:java -Dexec.mainClass="com.example.agent.AgentCliRunner"
```

![adk-run.png](/assets/adk-run.png)

### Ejecutar con interfaz web

Ejecuta tu agente con la interfaz web de ADK usando el siguiente comando Maven:

```console
# Recuerda cargar las claves y configuraciones: source .env OR env.bat
mvn compile exec:java \
    -Dexec.mainClass="com.google.adk.web.AdkWebServer" \
    -Dexec.args="--adk.agents.source-dir=target --server.port=8000"
```

Este comando inicia un servidor web con una interfaz de chat para tu agente. Puedes
acceder a la interfaz web en (http://localhost:8000). Selecciona tu agente en la
esquina superior izquierda y escribe una solicitud.

![adk-web-dev-ui-chat.png](/assets/adk-web-dev-ui-chat.png)

!!! warning "Precaución: ADK Web solo para desarrollo"

    ADK Web ***no está destinado para uso en implementaciones de producción***. Debes
    usar ADK Web solo para propósitos de desarrollo y depuración.

## Siguiente: Construye tu agente

Ahora que tienes ADK instalado y tu primer agente ejecutándose, intenta construir
tu propio agente con nuestras guías de construcción:

*  [Construye tu agente](/tutorials/)