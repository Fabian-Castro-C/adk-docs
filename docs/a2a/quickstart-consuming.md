# Inicio Rápido: Consumiendo un agente remoto vía A2A

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span><span class="lst-preview">Experimental</span>
</div>

Este inicio rápido cubre el punto de partida más común para cualquier desarrollador: **"Hay un agente remoto, ¿cómo puedo hacer que mi agente ADK lo use vía A2A?"**. Esto es crucial para construir sistemas multi-agente complejos donde diferentes agentes necesitan colaborar e interactuar.

## Descripción General

Esta muestra demuestra la arquitectura **Agent2Agent (A2A)** en el Agent Development Kit (ADK), mostrando cómo múltiples agentes pueden trabajar juntos para manejar tareas complejas. La muestra implementa un agente que puede lanzar dados y verificar si los números son primos.

```text
┌─────────────────┐    ┌──────────────────┐    ┌────────────────────┐
│   Root Agent    │───▶│   Roll Agent     │    │   Remote Prime     │
│  (Local)        │    │   (Local)        │    │   Agent            │
│                 │    │                  │    │  (localhost:8001)  │
│                 │───▶│                  │◀───│                    │
└─────────────────┘    └──────────────────┘    └────────────────────┘
```

La muestra A2A Basic consiste en:

- **Root Agent** (`root_agent`): El orquestador principal que delega tareas a sub-agentes especializados
- **Roll Agent** (`roll_agent`): Un sub-agente local que maneja operaciones de lanzamiento de dados
- **Prime Agent** (`prime_agent`): Un agente A2A remoto que verifica si los números son primos, este agente se ejecuta en un servidor A2A separado

## Exponiendo Tu Agente con el Servidor ADK

  El ADK viene con un comando CLI integrado, `adk api_server --a2a` para exponer tu agente usando el protocolo A2A.

  En el ejemplo `a2a_basic`, primero necesitarás exponer el `check_prime_agent` vía un servidor A2A, para que el agente raíz local pueda usarlo.

### 1. Obteniendo el Código de Muestra { #getting-the-sample-code }

Primero, asegúrate de tener las dependencias necesarias instaladas:

```bash
pip install google-adk[a2a]
```

Puedes clonar y navegar a la muestra [**`a2a_basic`**](https://github.com/google/adk-python/tree/main/contributing/samples/a2a_basic) aquí:

```bash
git clone https://github.com/google/adk-python.git
```

Como verás, la estructura de carpetas es la siguiente:

```text
a2a_basic/
├── remote_a2a/
│   └── check_prime_agent/
│       ├── __init__.py
│       ├── agent.json
│       └── agent.py
├── README.md
├── __init__.py
└── agent.py # agente raíz local
```

#### Agente Principal (`a2a_basic/agent.py`)

- **`roll_die(sides: int)`**: Herramienta de función para lanzar dados
- **`roll_agent`**: Agente local especializado en lanzamiento de dados
- **`prime_agent`**: Configuración de agente A2A remoto
- **`root_agent`**: Orquestador principal con lógica de delegación

#### Agente Prime Remoto (`a2a_basic/remote_a2a/check_prime_agent/`)

- **`agent.py`**: Implementación del servicio de verificación de primos
- **`agent.json`**: Tarjeta del agente A2A
- **`check_prime(nums: list[int])`**: Algoritmo de verificación de números primos

### 2. Iniciar el servidor del Agente Prime Remoto { #start-the-remote-prime-agent-server }

Para mostrar cómo tu agente ADK puede consumir un agente remoto vía A2A, primero necesitarás iniciar un servidor de agente remoto, que alojará el agente prime (bajo `check_prime_agent`).

```bash
# Inicia el servidor a2a remoto que sirve el check_prime_agent en el puerto 8001
adk api_server --a2a --port 8001 contributing/samples/a2a_basic/remote_a2a
```

??? note "Agregando logging para depuración con `--log_level debug`"
    Para habilitar el logging a nivel debug, puedes agregar `--log_level debug` a tu `adk api_server`, como en:
    ```bash
    adk api_server --a2a --port 8001 contributing/samples/a2a_basic/remote_a2a --log_level debug
    ```
    Esto proporcionará logs más completos para que puedas inspeccionar al probar tus agentes.

??? note "¿Por qué usar el puerto 8001?"
    En este inicio rápido, cuando pruebas localmente, tus agentes estarán usando localhost, por lo que el `port` para el servidor A2A del agente expuesto (el agente prime remoto) debe ser diferente del puerto del agente consumidor. El puerto predeterminado para `adk web` donde interactuarás con el agente consumidor es `8000`, razón por la cual el servidor A2A se crea usando un puerto separado, `8001`.

Una vez ejecutado, deberías ver algo como:

``` shell
INFO:     Started server process [56558]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8001 (Press CTRL+C to quit)
```
  
### 3. Busca la tarjeta de agente requerida (`agent-card.json`) del agente remoto { #look-out-for-the-required-agent-card-agent-json-of-the-remote-agent }

El Protocolo A2A requiere que cada agente debe tener una tarjeta de agente que describa lo que hace.

Si alguien más ya ha construido el agente A2A remoto que estás buscando consumir en tu agente, entonces debes confirmar que tienen una tarjeta de agente (`agent-card.json`).

En la muestra, el `check_prime_agent` ya tiene una tarjeta de agente proporcionada:

```json title="a2a_basic/remote_a2a/check_prime_agent/agent-card.json"

{
  "capabilities": {},
  "defaultInputModes": ["text/plain"],
  "defaultOutputModes": ["application/json"],
  "description": "An agent specialized in checking whether numbers are prime. It can efficiently determine the primality of individual numbers or lists of numbers.",
  "name": "check_prime_agent",
  "skills": [
    {
      "id": "prime_checking",
      "name": "Prime Number Checking",
      "description": "Check if numbers in a list are prime using efficient mathematical algorithms",
      "tags": ["mathematical", "computation", "prime", "numbers"]
    }
  ],
  "url": "http://localhost:8001/a2a/check_prime_agent",
  "version": "1.0.0"
}
```

??? note "Más información sobre tarjetas de agente en ADK"

    En ADK, puedes usar un wrapper `to_a2a(root_agent)` que genera automáticamente una tarjeta de agente para ti. Si estás interesado en aprender más sobre cómo exponer tu agente existente para que otros puedan usarlo, entonces por favor consulta el tutorial [Inicio Rápido A2A (Exponiendo)](quickstart-exposing.md). 

### 4. Ejecutar el Agente Principal (Consumidor) { #run-the-main-consuming-agent }

  ```bash
  # En una terminal separada, ejecuta el servidor adk web
  adk web contributing/samples/
  ```

#### Cómo funciona

El agente principal usa la función `RemoteA2aAgent()` para consumir el agente remoto (`prime_agent` en nuestro ejemplo). Como puedes ver a continuación, `RemoteA2aAgent()` requiere el `name`, `description`, y la URL de la `agent_card`.

```python title="a2a_basic/agent.py"
<...code truncated...>

from google.adk.agents.remote_a2a_agent import AGENT_CARD_WELL_KNOWN_PATH
from google.adk.agents.remote_a2a_agent import RemoteA2aAgent

prime_agent = RemoteA2aAgent(
    name="prime_agent",
    description="Agent that handles checking if numbers are prime.",
    agent_card=(
        f"http://localhost:8001/a2a/check_prime_agent{AGENT_CARD_WELL_KNOWN_PATH}"
    ),
)

<...code truncated>
```

Luego, simplemente puedes usar el `RemoteA2aAgent` en tu agente. En este caso, `prime_agent` se usa como uno de los sub-agentes en el `root_agent` a continuación:

```python title="a2a_basic/agent.py"
from google.adk.agents.llm_agent import Agent
from google.genai import types

root_agent = Agent(
    model="gemini-2.0-flash",
    name="root_agent",
    instruction="""
      <You are a helpful assistant that can roll dice and check if numbers are prime.
      You delegate rolling dice tasks to the roll_agent and prime checking tasks to the prime_agent.
      Follow these steps:
      1. If the user asks to roll a die, delegate to the roll_agent.
      2. If the user asks to check primes, delegate to the prime_agent.
      3. If the user asks to roll a die and then check if the result is prime, call roll_agent first, then pass the result to prime_agent.
      Always clarify the results before proceeding.>
    """,
    global_instruction=(
        "You are DicePrimeBot, ready to roll dice and check prime numbers."
    ),
    sub_agents=[roll_agent, prime_agent],
    tools=[example_tool],
    generate_content_config=types.GenerateContentConfig(
        safety_settings=[
            types.SafetySetting(  # evita falsa alarma sobre lanzamiento de dados.
                category=types.HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT,
                threshold=types.HarmBlockThreshold.OFF,
            ),
        ]
    ),
)
```

## Ejemplos de Interacciones

Una vez que tanto tu agente principal como los remotos estén ejecutándose, puedes interactuar con el agente raíz para ver cómo llama al agente remoto vía A2A:

**Lanzamiento Simple de Dados:**
Esta interacción usa un agente local, el Roll Agent:

```text
User: Roll a 6-sided die
Bot: I rolled a 4 for you.
```

**Verificación de Números Primos:**

Esta interacción usa un agente remoto vía A2A, el Prime Agent:

```text
User: Is 7 a prime number?
Bot: Yes, 7 is a prime number.
```

**Operaciones Combinadas:**

Esta interacción usa tanto el Roll Agent local como el Prime Agent remoto:

```text
User: Roll a 10-sided die and check if it's prime
Bot: I rolled an 8 for you.
Bot: 8 is not a prime number.
```

## Próximos Pasos

Ahora que has creado un agente que está usando un agente remoto vía un servidor A2A, el siguiente paso es aprender cómo conectarse a él desde otro agente.

- [**Inicio Rápido A2A (Exponiendo)**](./quickstart-exposing.md): Aprende cómo exponer tu agente existente para que otros agentes puedan usarlo vía el Protocolo A2A.