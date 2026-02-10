# Inicio Rápido: Consumir un agente remoto mediante A2A

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-go">Go</span><span class="lst-preview">Experimental</span>
</div>

Este inicio rápido cubre el punto de partida más común para cualquier desarrollador: **"Hay un agente remoto, ¿cómo hago que mi agente ADK lo use mediante A2A?"**. Esto es crucial para construir sistemas complejos multi-agente donde diferentes agentes necesitan colaborar e interactuar.

## Descripción General

Este ejemplo demuestra la arquitectura **Agent-to-Agent (A2A)** en el Agent Development Kit (ADK), mostrando cómo múltiples agentes pueden trabajar juntos para manejar tareas complejas. El ejemplo implementa un agente que puede lanzar dados y verificar si los números son primos.

```text
┌─────────────────┐    ┌──────────────────┐    ┌────────────────────┐
│   Root Agent    │───▶│   Roll Agent     │    │   Remote Prime     │
│  (Local)        │    │   (Local)        │    │   Agent            │
│                 │    │                  │    │  (localhost:8001)  │
│                 │───▶│                  │◀───│                    │
└─────────────────┘    └──────────────────┘    └────────────────────┘
```

El ejemplo A2A Basic consiste en:

- **Root Agent** (`root_agent`): El orquestador principal que delega tareas a sub-agentes especializados
- **Roll Agent** (`roll_agent`): Un sub-agente local que maneja operaciones de lanzamiento de dados
- **Prime Agent** (`prime_agent`): Un agente A2A remoto que verifica si los números son primos, este agente se ejecuta en un servidor A2A separado

## Exponiendo Tu Agente con el Servidor ADK

  En el ejemplo `a2a_basic`, primero necesitarás exponer el `check_prime_agent` mediante un servidor A2A, para que el agente raíz local pueda usarlo.

### 1. Obtener el Código de Ejemplo { #getting-the-sample-code }

Primero, asegúrate de tener Go instalado y tu entorno configurado.

Puedes clonar y navegar al ejemplo [**`a2a_basic`**](https://github.com/google/tree/main/examples/go/a2a_basic) aquí:

```bash
cd examples/go/a2a_basic
```

Como verás, la estructura de carpetas es la siguiente:

```text
a2a_basic/
├── remote_a2a/
│   └── check_prime_agent/
│       └── main.go
├── go.mod
├── go.sum
└── main.go # agente raíz local
```

#### Agente Principal (`a2a_basic/main.go`)

- **`rollDieTool`**: Herramienta de función para lanzar dados
- **`newRollAgent`**: Agente local especializado en lanzamiento de dados
- **`newPrimeAgent`**: Configuración del agente A2A remoto
- **`newRootAgent`**: Orquestador principal con lógica de delegación

#### Agente Prime Remoto (`a2a_basic/remote_a2a/check_prime_agent/main.go`)

- **`checkPrimeTool`**: Algoritmo de verificación de números primos
- **`main`**: Implementación del servicio de verificación de primos y servidor A2A.

### 2. Iniciar el servidor del Agente Prime Remoto { #start-the-remote-prime-agent-server }

Para mostrar cómo tu agente ADK puede consumir un agente remoto mediante A2A, primero necesitarás iniciar un servidor de agente remoto, que alojará el agente prime (bajo `check_prime_agent`).

```bash
# Iniciar el servidor a2a remoto que sirve el check_prime_agent en el puerto 8001
go run remote_a2a/check_prime_agent/main.go
```

Una vez ejecutado, deberías ver algo como:

``` shell
2025/11/06 11:00:19 Starting A2A prime checker server on port 8001
2025/11/06 11:00:19 Starting the web server: &{port:8001}
2025/11/06 11:00:19 
2025/11/06 11:00:19 Web servers starts on http://localhost:8001
2025/11/06 11:00:19        a2a:  you can access A2A using jsonrpc protocol: http://localhost:8001
```
  
### 3. Buscar la tarjeta de agente requerida del agente remoto { #look-out-for-the-required-agent-card-of-the-remote-agent }

El Protocolo A2A requiere que cada agente tenga una tarjeta de agente que describa lo que hace.

En el ADK de Go, la tarjeta de agente se genera dinámicamente cuando expones un agente usando el lanzador A2A. Puedes visitar `http://localhost:8001/.well-known/agent-card.json` para ver la tarjeta generada.

### 4. Ejecutar el Agente Principal (Consumidor) { #run-the-main-consuming-agent }

  ```bash
  # En una terminal separada, ejecutar el agente principal
  go run main.go
  ```

#### Cómo funciona

El agente principal usa `remoteagent.New` para consumir el agente remoto (`prime_agent` en nuestro ejemplo). Como puedes ver a continuación, requiere el `Name`, `Description` y la URL `AgentCardSource`.

```go title="a2a_basic/main.go"
--8<-- "examples/go/a2a_basic/main.go:new-prime-agent"
```

Luego, simplemente puedes usar el agente remoto en tu agente raíz. En este caso, `primeAgent` se usa como uno de los sub-agentes en el `root_agent` a continuación:

```go title="a2a_basic/main.go"
--8<-- "examples/go/a2a_basic/main.go:new-root-agent"
```

## Ejemplos de Interacciones

Una vez que tanto tu agente principal como el remoto estén ejecutándose, puedes interactuar con el agente raíz para ver cómo llama al agente remoto mediante A2A:

**Lanzamiento Simple de Dados:**
Esta interacción usa un agente local, el Roll Agent:

```text
User: Roll a 6-sided die
Bot calls tool: transfer_to_agent with args: map[agent_name:roll_agent]
Bot calls tool: roll_die with args: map[sides:6]
Bot: I rolled a 6-sided die and the result is 6.
```

**Verificación de Números Primos:**

Esta interacción usa un agente remoto mediante A2A, el Prime Agent:

```text
User: Is 7 a prime number?
Bot calls tool: transfer_to_agent with args: map[agent_name:prime_agent]
Bot calls tool: prime_checking with args: map[nums:[7]]
Bot: Yes, 7 is a prime number.
```

**Operaciones Combinadas:**

Esta interacción usa tanto el Roll Agent local como el Prime Agent remoto:

```text
User: roll a die and check if it's a prime
Bot: Okay, I will first roll a die and then check if the result is a prime number.

Bot calls tool: transfer_to_agent with args: map[agent_name:roll_agent]
Bot calls tool: roll_die with args: map[sides:6]
Bot calls tool: transfer_to_agent with args: map[agent_name:prime_agent]
Bot calls tool: prime_checking with args: map[nums:[3]]
Bot: 3 is a prime number.
```

## Próximos Pasos

Ahora que has creado un agente que está usando un agente remoto mediante un servidor A2A, el siguiente paso es aprender cómo exponer tu propio agente.

- [**Inicio Rápido A2A (Exposición)**](./quickstart-exposing-go.md): Aprende cómo exponer tu agente existente para que otros agentes puedan usarlo mediante el Protocolo A2A.