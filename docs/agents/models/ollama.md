# Alojamiento de modelos Ollama para agentes ADK

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span>
</div>

[Ollama](https://ollama.com/) es una herramienta que te permite alojar y ejecutar
modelos de código abierto localmente. ADK se integra con modelos alojados en Ollama a través de la
biblioteca de conectores de modelos [LiteLLM](/agents/models/litellm/).

## Comenzar

Utiliza el wrapper de LiteLLM para crear agentes con modelos alojados en Ollama. El
siguiente ejemplo de código muestra una implementación básica para usar modelos abiertos
Gemma con tus agentes:

```py
root_agent = Agent(
    model=LiteLlm(model="ollama_chat/gemma3:latest"),
    name="dice_agent",
    description=(
        "hello world agent that can roll a dice of 8 sides and check prime"
        " numbers."
    ),
    instruction="""
      You roll dice and answer questions about the outcome of the dice rolls.
    """,
    tools=[
        roll_die,
        check_prime,
    ],
)
```

!!! warning "Advertencia: Usa la interfaz `ollama_chat`"

    Asegúrate de establecer el proveedor `ollama_chat` en lugar de `ollama`. Usar
    `ollama` puede resultar en comportamientos inesperados como bucles infinitos de llamadas a herramientas
    e ignorar el contexto previo.

!!! tip "Usa la variable de entorno `OLLAMA_API_BASE`"

    Aunque puedes especificar el parámetro `api_base` en LiteLLM para la generación,
    desde la v1.65.5, la biblioteca depende de la variable de entorno para otras llamadas API.
    Por lo tanto, debes establecer la variable de entorno `OLLAMA_API_BASE` para la
    URL de tu servidor Ollama para asegurar que todas las solicitudes se enruten correctamente.

```bash
export OLLAMA_API_BASE="http://localhost:11434"
adk web
```

## Elección del modelo

Si tu agente depende de herramientas, asegúrate de seleccionar un modelo con
soporte de herramientas desde el [sitio web de Ollama](https://ollama.com/search?c=tools).
Para resultados confiables, usa un modelo con soporte de herramientas.
Puedes verificar el soporte de herramientas para el modelo usando el siguiente comando:

```bash
ollama show mistral-small3.1
  Model
    architecture        mistral3
    parameters          24.0B
    context length      131072
    embedding length    5120
    quantization        Q4_K_M

  Capabilities
    completion
    vision
    tools
```

Deberías ver **tools** listado bajo capabilities.
También puedes ver la plantilla que el modelo está usando y ajustarla según tus
necesidades.

```bash
ollama show --modelfile llama3.2 > model_file_to_modify
```

Por ejemplo, la plantilla predeterminada para el modelo anterior sugiere inherentemente que
el modelo debe llamar a una función todo el tiempo. Esto puede resultar en un bucle
infinito de llamadas a funciones.

```
Given the following functions, please respond with a JSON for a function call
with its proper arguments that best answers the given prompt.

Respond in the format {"name": function name, "parameters": dictionary of
argument name and its value}. Do not use variables.
```

Puedes reemplazar tales prompts con uno más descriptivo para prevenir bucles infinitos de
llamadas a herramientas, por ejemplo:

```
Review the user's prompt and the available functions listed below.

First, determine if calling one of these functions is the most appropriate way
to respond. A function call is likely needed if the prompt asks for a specific
action, requires external data lookup, or involves calculations handled by the
functions. If the prompt is a general question or can be answered directly, a
function call is likely NOT needed.

If you determine a function call IS required: Respond ONLY with a JSON object in
the format {"name": "function_name", "parameters": {"argument_name": "value"}}.
Ensure parameter values are concrete, not variables.

If you determine a function call IS NOT required: Respond directly to the user's
prompt in plain text, providing the answer or information requested. Do not
output any JSON.
```

Luego puedes crear un nuevo modelo con el siguiente comando:

```bash
ollama create llama3.2-modified -f model_file_to_modify
```

## Usar el proveedor OpenAI

Alternativamente, puedes usar `openai` como el nombre del proveedor. Este enfoque
requiere establecer las variables de entorno `OPENAI_API_BASE=http://localhost:11434/v1` y
`OPENAI_API_KEY=anything` en lugar de `OLLAMA_API_BASE`.
Ten en cuenta que el valor de `API_BASE` tiene *`/v1`* al final.

```py
root_agent = Agent(
    model=LiteLlm(model="openai/mistral-small3.1"),
    name="dice_agent",
    description=(
        "hello world agent that can roll a dice of 8 sides and check prime"
        " numbers."
    ),
    instruction="""
      You roll dice and answer questions about the outcome of the dice rolls.
    """,
    tools=[
        roll_die,
        check_prime,
    ],
)
```

```bash
export OPENAI_API_BASE=http://localhost:11434/v1
export OPENAI_API_KEY=anything
adk web
```

### Depuración

Puedes ver la solicitud enviada al servidor Ollama agregando lo siguiente en
tu código de agente justo después de los imports.

```py
import litellm
litellm._turn_on_debug()
```

Busca una línea como la siguiente:

```bash
Request Sent from LiteLLM:
curl -X POST \
http://localhost:11434/api/chat \
-d '{'model': 'mistral-small3.1', 'messages': [{'role': 'system', 'content': ...
```