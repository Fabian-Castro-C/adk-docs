# Usa la Línea de Comandos

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

ADK proporciona una interfaz de terminal interactiva para probar tus agentes. Esto es
útil para pruebas rápidas, interacciones con scripts y pipelines de CI/CD.

![ADK Run](../assets/adk-run.png)

## Ejecutar un agente

Usa el siguiente comando para ejecutar tu agente en la interfaz de línea de comandos de ADK:

=== "Python"

    ```shell
    adk run my_agent
    ```

=== "TypeScript"

    ```shell
    npx @google/adk-devtools run agent.ts
    ```

=== "Go"

    ```shell
    go run agent.go
    ```

=== "Java"

    Crea una clase `AgentCliRunner` (ver [Inicio Rápido Java](../get-started/java.md)) y ejecuta:

    ```shell
    mvn compile exec:java -Dexec.mainClass="com.example.agent.AgentCliRunner"
    ```

Esto inicia una sesión interactiva donde puedes escribir consultas y ver las
respuestas del agente directamente en tu terminal:

```shell
Running agent my_agent, type exit to exit.
[user]: What's the weather in New York?
[my_agent]: The weather in New York is sunny with a temperature of 25°C.
[user]: exit
```

## Opciones de sesión

El comando `adk run` incluye opciones para guardar, reanudar y reproducir
sesiones.

### Guardar sesiones

Para guardar la sesión cuando salgas:

```shell
adk run --save_session path/to/my_agent
```

Se te pedirá que ingreses un ID de sesión, y la sesión se guardará en
`path/to/my_agent/<session_id>.session.json`.

También puedes especificar el ID de sesión por adelantado:

```shell
adk run --save_session --session_id my_session path/to/my_agent
```

### Reanudar sesiones

Para continuar una sesión previamente guardada:

```shell
adk run --resume path/to/my_agent/my_session.session.json path/to/my_agent
```

Esto carga el estado de sesión previo y el historial de eventos, lo muestra, y te permite
continuar la conversación.

### Reproducir sesiones

Para reproducir un archivo de sesión sin entrada interactiva:

```shell
adk run --replay path/to/input.json path/to/my_agent
```

El archivo de entrada debe contener el estado inicial y las consultas:

```json
{
  "state": {"key": "value"},
  "queries": ["What is 2 + 2?", "What is the capital of France?"]
}
```

## Opciones de almacenamiento

| Opción | Descripción | Por defecto |
|--------|-------------|---------|
| `--session_service_uri` | URI personalizado de almacenamiento de sesiones | SQLite bajo `.adk/session.db` |
| `--artifact_service_uri` | URI personalizado de almacenamiento de artefactos | Local `.adk/artifacts` |

### Ejemplo con opciones de almacenamiento

```shell
adk run --session_service_uri "sqlite:///my_sessions.db" path/to/my_agent
```

## Todas las opciones

| Opción | Descripción |
|--------|-------------|
| `--save_session` | Guardar la sesión en un archivo JSON al salir |
| `--session_id` | ID de sesión a usar al guardar |
| `--resume` | Ruta a un archivo de sesión guardado para reanudar |
| `--replay` | Ruta a un archivo de entrada para reproducción no interactiva |
| `--session_service_uri` | URI personalizado de almacenamiento de sesiones |
| `--artifact_service_uri` | URI personalizado de almacenamiento de artefactos |