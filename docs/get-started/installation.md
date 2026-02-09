# Instalación de ADK

=== "Python"

    ## Crear y activar entorno virtual

    Recomendamos crear un entorno virtual de Python usando
    [venv](https://docs.python.org/3/library/venv.html):

    ```shell
    python -m venv .venv
    ```

    Ahora, puedes activar el entorno virtual usando el comando apropiado para
    tu sistema operativo y entorno:

    ```
    # Mac / Linux
    source .venv/bin/activate

    # Windows CMD:
    .venv\Scripts\activate.bat

    # Windows PowerShell:
    .venv\Scripts\Activate.ps1
    ```

    ### Instalar ADK

    ```bash
    pip install google-adk
    ```

    (Opcional) Verifica tu instalación:

    ```bash
    pip show google-adk
    ```

=== "TypeScript"

    ### Instalar ADK y ADK DevTools

    ```bash
    npm install @google/adk @google/adk-devtools
    ```

=== "Go"

    ## Crear un nuevo módulo Go

    Si estás comenzando un nuevo proyecto, puedes crear un nuevo módulo Go:

    ```shell
    go mod init example.com/my-agent
    ```

    ## Instalar ADK

    Para agregar el ADK a tu proyecto, ejecuta el siguiente comando:

    ```shell
    go get google.golang.org/adk
    ```

    Esto agregará el ADK como una dependencia a tu archivo `go.mod`.

    (Opcional) Verifica tu instalación revisando tu archivo `go.mod` para la entrada `google.golang.org/adk`.

=== "Java"

    Puedes usar maven o gradle para agregar los paquetes `google-adk` y `google-adk-dev`.

    `google-adk` es la biblioteca principal de Java ADK. Java ADK también viene con un servidor SpringBoot de ejemplo conectable para ejecutar tus agentes sin problemas. Este paquete opcional
    está presente como parte de `google-adk-dev`.

    Si estás usando maven, agrega lo siguiente a tu `pom.xml`:

    ```xml title="pom.xml"
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
                <version>0.5.0</version>
            </dependency>
            <!-- La interfaz web de desarrollo de ADK para depurar tu agente -->
            <dependency>
                <groupId>com.google.adk</groupId>
                <artifactId>google-adk-dev</artifactId>
                <version>0.5.0</version>
            </dependency>
        </dependencies>

    </project>
    ```

    Aquí hay un [pom.xml completo](https://github.com/google/adk-docs/tree/main/examples/java/cloud-run/pom.xml) como referencia.

    Si estás usando gradle, agrega la dependencia a tu build.gradle:

    ```title="build.gradle"
    dependencies {
        implementation 'com.google.adk:google-adk:0.5.0'
        implementation 'com.google.adk:google-adk-dev:0.5.0'
    }
    ```

    También deberías configurar Gradle para pasar `-parameters` a `javac`. (Alternativamente, usa `@Schema(name = "...")`).

## Próximos pasos

* Intenta crear tu primer agente con el [**Inicio rápido**](quickstart.md)