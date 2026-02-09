# Inicio Rápido: Exposición de un agente remoto vía A2A

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span><span class="lst-preview">Experimental</span>
</div>

Este inicio rápido cubre el punto de partida más común para cualquier desarrollador: **"Tengo un agente. ¿Cómo lo expongo para que otros agentes puedan usar mi agente vía A2A?"**. Esto es crucial para construir sistemas complejos multi-agente donde diferentes agentes necesitan colaborar e interactuar.

## Resumen General

Este ejemplo demuestra cómo puedes exponer fácilmente un agente ADK para que luego pueda ser consumido por otro agente usando el Protocolo A2A.

Hay dos formas principales de exponer un agente ADK vía A2A.

* **usando la función `to_a2a(root_agent)`**: usa esta función si solo quieres convertir un agente existente para que funcione con A2A, y poder exponerlo vía un servidor a través de `uvicorn`, en lugar de `adk deploy api_server`. Esto significa que tienes un control más estricto sobre lo que quieres exponer vía `uvicorn` cuando quieres productivizar tu agente. Además, la función `to_a2a()` auto-genera una tarjeta de agente basada en tu código de agente.
* **creando tu propia tarjeta de agente (`agent.json`) y hospedándola usando `adk api_server --a2a`**: Hay dos beneficios principales al usar este enfoque. Primero, `adk api_server --a2a` funciona con `adk web`, haciendo que sea fácil de usar, depurar y probar tu agente. Segundo, con `adk api_server`, puedes especificar una carpeta padre con múltiples agentes separados. Aquellos agentes que tengan una tarjeta de agente (`agent.json`), automáticamente serán usables vía A2A por otros agentes a través del mismo servidor. Sin embargo, necesitarás crear tus propias tarjetas de agente. Para crear una tarjeta de agente, puedes seguir el [tutorial de Python de A2A](https://a2a-protocol.org/latest/tutorials/python/1-introduction/).

Este inicio rápido se enfocará en `to_a2a()`, ya que es la forma más fácil de exponer tu agente y también autogenerará la tarjeta de agente detrás de escenas. Si te gustaría usar el enfoque de `adk api_server`, puedes verlo siendo usado en la [documentación de Inicio Rápido de A2A (Consumo)](quickstart-consuming.md).

```text
Before:
                                                ┌────────────────────┐
                                                │ Hello World Agent  │
                                                │  (Python Object)   │
                                                | without agent card │
                                                └────────────────────┘

                                                          │
                                                          │ to_a2a()
                                                          ▼

After:
┌────────────────┐                             ┌───────────────────────────────┐
│   Root Agent   │       A2A Protocol          │ A2A-Exposed Hello World Agent │
│(RemoteA2aAgent)│────────────────────────────▶│      (localhost: 8001)         │
│(localhost:8000)│                             └───────────────────────────────┘
└────────────────┘
```

El ejemplo consiste en:

- **Agente Remoto Hello World** (`remote_a2a/hello_world/agent.py`): Este es el agente que quieres exponer para que otros agentes puedan usarlo vía A2A. Es un agente que maneja el lanzamiento de dados y la verificación de números primos. Se expone usando la función `to_a2a()` y se sirve usando `uvicorn`.
- **Agente Raíz** (`agent.py`): Un agente simple que solo está llamando al agente remoto Hello World.

## Exponiendo el Agente Remoto con la función `to_a2a(root_agent)`

Puedes tomar un agente existente construido usando ADK y hacerlo compatible con A2A simplemente envolviéndolo usando la función `to_a2a()`. Por ejemplo, si tienes un agente como el siguiente definido en `root_agent`:

```python
# Tu código de agente aquí
root_agent = Agent(
    model='gemini-2.0-flash',
    name='hello_world_agent',
    
    <...your agent code...>
)
```

Entonces puedes hacerlo compatible con A2A simplemente usando `to_a2a(root_agent)`:

```python
from google.adk.a2a.utils.agent_to_a2a import to_a2a

# Haz tu agente compatible con A2A
a2a_app = to_a2a(root_agent, port=8001)
```

La función `to_a2a()` incluso auto-generará una tarjeta de agente en memoria detrás de escenas [extrayendo habilidades, capacidades y metadatos del agente ADK](https://github.com/google/adk-python/blob/main/src/google/adk/a2a/utils/agent_card_builder.py), de modo que la conocida tarjeta de agente esté disponible cuando el endpoint del agente sea servido usando `uvicorn`.

También puedes proporcionar tu propia tarjeta de agente usando el parámetro `agent_card`. El valor puede ser un objeto `AgentCard` o una ruta a un archivo JSON de tarjeta de agente.

**Ejemplo con un objeto `AgentCard`:**
```python
from google.adk.a2a.utils.agent_to_a2a import to_a2a
from a2a.types import AgentCard

# Define la tarjeta de agente A2A
my_agent_card = AgentCard(
    name="file_agent",
    url="http://example.com",
    description="Test agent from file",
    version="1.0.0",
    capabilities={},
    skills=[],
    defaultInputModes=["text/plain"],
    defaultOutputModes=["text/plain"],
    supportsAuthenticatedExtendedCard=False,
)
a2a_app = to_a2a(root_agent, port=8001, agent_card=my_agent_card)
```

**Ejemplo con una ruta a un archivo JSON:**
```python
from google.adk.a2a.utils.agent_to_a2a import to_a2a

# Carga la tarjeta de agente A2A desde un archivo
a2a_app = to_a2a(root_agent, port=8001, agent_card="/path/to/your/agent-card.json")
```

Ahora profundicemos en el código de ejemplo.

### 1. Obtener el Código de Ejemplo { #getting-the-sample-code }

Primero, asegúrate de tener las dependencias necesarias instaladas:

```bash
pip install google-adk[a2a]
```

Puedes clonar y navegar al ejemplo [**a2a_root**](https://github.com/google/adk-python/tree/main/contributing/samples/a2a_root) aquí:

```bash
git clone https://github.com/google/adk-python.git
```

Como verás, la estructura de carpetas es la siguiente:

```text
a2a_root/
├── remote_a2a/
│   └── hello_world/    
│       ├── __init__.py
│       └── agent.py    # Agente Remoto Hello World
├── README.md
└── agent.py            # Agente raíz
```

#### Agente Raíz (`a2a_root/agent.py`)

- **`root_agent`**: Un `RemoteA2aAgent` que se conecta al servicio A2A remoto
- **URL de Tarjeta de Agente**: Apunta al endpoint de tarjeta de agente conocido en el servidor remoto

#### Agente Remoto Hello World (`a2a_root/remote_a2a/hello_world/agent.py`)

- **`roll_die(sides: int)`**: Herramienta de función para lanzar dados con gestión de estado
- **`check_prime(nums: list[int])`**: Función asíncrona para verificación de números primos
- **`root_agent`**: El agente principal con instrucciones completas
- **`a2a_app`**: La aplicación A2A creada usando la utilidad `to_a2a()`

### 2. Iniciar el servidor del Agente A2A Remoto { #start-the-remote-a2a-agent-server }

Ahora puedes iniciar el servidor del agente remoto, que hospedará la `a2a_app` dentro del agente hello_world:

```bash
# Asegúrate de que el directorio de trabajo actual sea adk-python/
# Inicia el agente remoto usando uvicorn
uvicorn contributing.samples.a2a_root.remote_a2a.hello_world.agent:a2a_app --host localhost --port 8001
```

??? note "¿Por qué usar el puerto 8001?"
    En este inicio rápido, al probar localmente, tus agentes estarán usando localhost, por lo que el `port` para el servidor A2A del agente expuesto (el agente remoto de primos) debe ser diferente del puerto del agente consumidor. El puerto predeterminado para `adk web` donde interactuarás con el agente consumidor es `8000`, por lo que el servidor A2A se crea usando un puerto separado, `8001`.

Una vez ejecutado, deberías ver algo como:

```shell
INFO:     Started server process [10615]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://localhost:8001 (Press CTRL+C to quit)
```

### 3. Verificar que tu agente remoto está ejecutándose { #check-that-your-remote-agent-is-running }

Puedes verificar que tu agente está activo y ejecutándose visitando la tarjeta de agente que fue auto-generada anteriormente como parte de tu función `to_a2a()` en `a2a_root/remote_a2a/hello_world/agent.py`:

[http://localhost:8001/.well-known/agent-card.json](http://localhost:8001/.well-known/agent-card.json)

Deberías ver el contenido de la tarjeta de agente, que debería verse así:

```json
{"capabilities":{},"defaultInputModes":["text/plain"],"defaultOutputModes":["text/plain"],"description":"hello world agent that can roll a dice of 8 sides and check prime numbers.","name":"hello_world_agent","protocolVersion":"0.2.6","skills":[{"description":"hello world agent that can roll a dice of 8 sides and check prime numbers. \n      I roll dice and answer questions about the outcome of the dice rolls.\n      I can roll dice of different sizes.\n      I can use multiple tools in parallel by calling functions in parallel(in one request and in one round).\n      It is ok to discuss previous dice roles, and comment on the dice rolls.\n      When I are asked to roll a die, I must call the roll_die tool with the number of sides. Be sure to pass in an integer. Do not pass in a string.\n      I should never roll a die on my own.\n      When checking prime numbers, call the check_prime tool with a list of integers. Be sure to pass in a list of integers. I should never pass in a string.\n      I should not check prime numbers before calling the tool.\n      When I are asked to roll a die and check prime numbers, I should always make the following two function calls:\n      1. I should first call the roll_die tool to get a roll. Wait for the function response before calling the check_prime tool.\n      2. After I get the function response from roll_die tool, I should call the check_prime tool with the roll_die result.\n        2.1 If user asks I to check primes based on previous rolls, make sure I include the previous rolls in the list.\n      3. When I respond, I must include the roll_die result from step 1.\n      I should always perform the previous 3 steps when asking for a roll and checking prime numbers.\n      I should not rely on the previous history on prime results.\n    ","id":"hello_world_agent","name":"model","tags":["llm"]},{"description":"Roll a die and return the rolled result.\n\nArgs:\n  sides: The integer number of sides the die has.\n  tool_context: the tool context\nReturns:\n  An integer of the result of rolling the die.","id":"hello_world_agent-roll_die","name":"roll_die","tags":["llm","tools"]},{"description":"Check if a given list of numbers are prime.\n\nArgs:\n  nums: The list of numbers to check.\n\nReturns:\n  A str indicating which number is prime.","id":"hello_world_agent-check_prime","name":"check_prime","tags":["llm","tools"]}],"supportsAuthenticatedExtendedCard":false,"url":"http://localhost:8001","version":"0.0.1"}
```

### 4. Ejecutar el Agente Principal (Consumidor) { #run-the-main-consuming-agent }

Ahora que tu agente remoto está ejecutándose, puedes lanzar la interfaz de desarrollo y seleccionar "a2a_root" como tu agente.

```bash
# En un terminal separado, ejecuta el servidor adk web
adk web contributing/samples/
```

Para abrir el servidor adk web, ve a: [http://localhost:8000](http://localhost:8000).

## Ejemplos de Interacciones

Una vez que ambos servicios estén ejecutándose, puedes interactuar con el agente raíz para ver cómo llama al agente remoto vía A2A:

**Lanzamiento Simple de Dados:**
Esta interacción usa un agente local, el Agente de Dados:

```text
User: Roll a 6-sided die
Bot: I rolled a 4 for you.
```

**Verificación de Números Primos:**

Esta interacción usa un agente remoto vía A2A, el Agente de Primos:

```text
User: Is 7 a prime number?
Bot: Yes, 7 is a prime number.
```

**Operaciones Combinadas:**

Esta interacción usa tanto el Agente de Dados local como el Agente de Primos remoto:

```text
User: Roll a 10-sided die and check if it's prime
Bot: I rolled an 8 for you.
Bot: 8 is not a prime number.
```

## Próximos Pasos

Ahora que has creado un agente que está exponiendo un agente remoto vía un servidor A2A, el siguiente paso es aprender cómo consumirlo desde otro agente.

- [**Inicio Rápido de A2A (Consumo)**](./quickstart-consuming.md): Aprende cómo tu agente puede usar otros agentes usando el Protocolo A2A.