# Usa la interfaz web

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

La interfaz web del ADK te permite probar tus agentes directamente en el navegador. Esta
herramienta proporciona una forma simple de desarrollar y depurar tus agentes de manera interactiva.

![ADK Web Interface](../assets/adk-web-dev-ui-chat.png)

!!! warning "Precaución: ADK Web solo para desarrollo"

    ADK Web ***no está destinado para uso en despliegues de producción***. Deberías
    usar ADK Web únicamente para propósitos de desarrollo y depuración.

## Inicia la interfaz web

Usa el siguiente comando para ejecutar tu agente en la interfaz web del ADK:

=== "Python"

    ```shell
    adk web
    ```

=== "TypeScript"

    ```shell
    npx adk web
    ```

=== "Go"

    ```shell
    go run agent.go web api webui
    ```

=== "Java"

    Asegúrate de actualizar el número de puerto.
    === "Maven"
        Con Maven, compila y ejecuta el servidor web del ADK:
        ```console
        mvn compile exec:java \
         -Dexec.args="--adk.agents.source-dir=src/main/java/agents --server.port=8080"
        ```
    === "Gradle"
        Con Gradle, el archivo de construcción `build.gradle` o `build.gradle.kts` debe tener el siguiente plugin de Java en su sección de plugins:

        ```groovy
        plugins {
            id('java')
            // otros plugins
        }
        ```
        Luego, en otra parte del archivo de construcción, en el nivel superior, crea una nueva tarea:

        ```groovy
        tasks.register('runADKWebServer', JavaExec) {
            dependsOn classes
            classpath = sourceSets.main.runtimeClasspath
            mainClass = 'com.google.adk.web.AdkWebServer'
            args '--adk.agents.source-dir=src/main/java/agents', '--server.port=8080'
        }
        ```

        Finalmente, en la línea de comandos, ejecuta el siguiente comando:
        ```console
        gradle runADKWebServer
        ```


    En Java, la interfaz web y el servidor API están empaquetados juntos.

El servidor inicia en `http://localhost:8000` por defecto:

```shell
+-----------------------------------------------------------------------------+
| ADK Web Server started                                                      |
|                                                                             |
| For local testing, access at http://localhost:8000.                         |
+-----------------------------------------------------------------------------+
```

## Características

Las características clave de la interfaz web del ADK incluyen:

- **Interfaz de chat**: Envía mensajes a tus agentes y visualiza respuestas en tiempo real
- **Gestión de sesiones**: Crea y cambia entre sesiones
- **Inspección de estado**: Visualiza y modifica el estado de la sesión durante el desarrollo
- **Historial de eventos**: Inspecciona todos los eventos generados durante la ejecución del agente

## Opciones comunes

| Opción | Descripción | Por defecto |
|--------|-------------|---------|
| `--port` | Puerto en el que se ejecutará el servidor | `8000` |
| `--host` | Dirección de enlace del host | `127.0.0.1` |
| `--session_service_uri` | URI de almacenamiento de sesión personalizado | En memoria |
| `--artifact_service_uri` | URI de almacenamiento de artefactos personalizado | Local `.adk/artifacts` |
| `--reload/--no-reload` | Habilitar recarga automática en cambios de código | `true` |

### Ejemplo con opciones

```shell
adk web --port 3000 --session_service_uri "sqlite:///sessions.db"
```