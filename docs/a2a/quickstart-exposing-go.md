# Inicio rápido: Exponiendo un agente remoto mediante A2A

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-go">Go</span><span class="lst-preview">Experimental</span>
</div>

Este inicio rápido cubre el punto de partida más común para cualquier desarrollador: **"Tengo un agente. ¿Cómo lo expongo para que otros agentes puedan usar mi agente mediante A2A?"**. Esto es crucial para construir sistemas multi-agente complejos donde diferentes agentes necesitan colaborar e interactuar.

## Descripción general

Este ejemplo demuestra cómo puedes exponer fácilmente un agente ADK para que luego pueda ser consumido por otro agente usando el Protocolo A2A.

En Go, expones un agente usando el lanzador A2A, que genera dinámicamente una tarjeta de agente para ti.

```text
┌─────────────────┐                             ┌───────────────────────────────┐
│   Root Agent    │       A2A Protocol          │ A2A-Exposed Check Prime Agent │
│                 │────────────────────────────▶│      (localhost: 8001)        │
└─────────────────┘                             └───────────────────────────────┘
```

El ejemplo consiste en:

- **Agente Primo Remoto** (`remote_a2a/check_prime_agent/main.go`): Este es el agente que deseas exponer para que otros agentes puedan usarlo mediante A2A. Es un agente que maneja la verificación de números primos. Se expone usando el lanzador A2A.
- **Agente Raíz** (`main.go`): Un agente simple que simplemente llama al agente primo remoto.

## Exponiendo el Agente Remoto con el Lanzador A2A

Puedes tomar un agente existente construido usando el ADK de Go y hacerlo compatible con A2A usando el lanzador A2A.

### 1. Obteniendo el Código de Ejemplo { #getting-the-sample-code }

Primero, asegúrate de tener Go instalado y tu entorno configurado.

Puedes clonar y navegar al ejemplo [**`a2a_basic`**](https://github.com/google/adk-docs/tree/main/examples/go/a2a_basic) aquí:

```bash
cd examples/go/a2a_basic
```

Como verás, la estructura de carpetas es la siguiente:

```text
a2a_basic/
├── remote_a2a/
│   └── check_prime_agent/
│       └── main.go    # Agente Primo Remoto
├── go.mod
├── go.sum
└── main.go            # Agente raíz
```

#### Agente Raíz (`a2a_basic/main.go`)

- **`newRootAgent`**: Un agente local que se conecta al servicio A2A remoto.

#### Agente Primo Remoto (`a2a_basic/remote_a2a/check_prime_agent/main.go`)

- **`checkPrimeTool`**: Función para verificación de números primos.
- **`main`**: La función principal que crea el agente e inicia el servidor A2A.

### 2. Iniciar el servidor del Agente A2A Remoto { #start-the-remote-a2a-agent-server }

Ahora puedes iniciar el servidor del agente remoto, que alojará el `check_prime_agent`:

```bash
# Iniciar el agente remoto
go run remote_a2a/check_prime_agent/main.go
```

Una vez ejecutado, deberías ver algo como:

```shell
2025/11/06 11:00:19 Starting A2A prime checker server on port 8001
2025/11/06 11:00:19 Starting the web server: &{port:8001}
2025/11/06 11:00:19 
2025/11/06 11:00:19 Web servers starts on http://localhost:8001
2025/11/06 11:00:19        a2a:  you can access A2A using jsonrpc protocol: http://localhost:8001
```

### 3. Verificar que tu agente remoto está en ejecución { #check-that-your-remote-agent-is-running }

Puedes verificar que tu agente está activo y en ejecución visitando la tarjeta de agente que fue generada automáticamente por el lanzador A2A:

[http://localhost:8001/.well-known/agent-card.json](http://localhost:8001/.well-known/agent-card.json)

Deberías ver el contenido de la tarjeta del agente.

### 4. Ejecutar el Agente Principal (Consumidor) { #run-the-main-consuming-agent }

Ahora que tu agente remoto está en ejecución, puedes ejecutar el agente principal.

```bash
# En una terminal separada, ejecuta el agente principal
go run main.go
```

#### Cómo funciona

El agente remoto se expone usando el lanzador A2A en la función `main`. El lanzador se encarga de iniciar el servidor y generar la tarjeta del agente.

```go title="remote_a2a/check_prime_agent/main.go"
--8<-- "examples/go/a2a_basic/remote_a2a/check_prime_agent/main.go:a2a-launcher"
```

## Ejemplos de Interacciones

Una vez que ambos servicios estén en ejecución, puedes interactuar con el agente raíz para ver cómo llama al agente remoto mediante A2A:

**Verificación de Números Primos:**

Esta interacción usa un agente remoto mediante A2A, el Agente Primo:

```text
User: roll a die and check if it's a prime
Bot: Okay, I will first roll a die and then check if the result is a prime number.

Bot calls tool: transfer_to_agent with args: map[agent_name:roll_agent]
Bot calls tool: roll_die with args: map[sides:6]
Bot calls tool: transfer_to_agent with args: map[agent_name:prime_agent]
Bot calls tool: prime_checking with args: map[nums:[3]]
Bot: 3 is a prime number.
...
```

## Próximos Pasos

Ahora que has creado un agente que expone un agente remoto mediante un servidor A2A, el siguiente paso es aprender cómo consumirlo desde otro agente.

- [**Inicio rápido de A2A (Consumo)**](./quickstart-consuming-go.md): Aprende cómo tu agente puede usar otros agentes usando el Protocolo A2A.