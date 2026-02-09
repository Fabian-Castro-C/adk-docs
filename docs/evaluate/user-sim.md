# Simulación de Usuario

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v1.18.0</span>
</div>

Al evaluar agentes conversacionales, no siempre es práctico utilizar un conjunto
fijo de prompts de usuario, ya que la conversación puede proceder de maneras inesperadas.
Por ejemplo, si el agente necesita que el usuario proporcione dos valores para realizar una tarea,
puede solicitar esos valores uno a la vez o ambos a la vez.
Para resolver este problema, ADK puede generar dinámicamente prompts de usuario utilizando un
modelo de IA generativa.

Para usar esta función, debes especificar un
[`ConversationScenario`](https://github.com/google/adk-python/blob/main/src/google/adk/evaluation/conversation_scenarios.py)
que dicta los objetivos del usuario en su conversación con el agente.
Un escenario de conversación de muestra para el agente
[`hello_world`](https://github.com/google/adk-python/tree/main/contributing/samples/hello_world)
se muestra a continuación:

```json
{
  "starting_prompt": "What can you do for me?",
  "conversation_plan": "Ask the agent to roll a 20-sided die. After you get the result, ask the agent to check if it is prime."
}
```

El `starting_prompt` en un escenario de conversación especifica un prompt inicial
fijo que el usuario debe usar para iniciar la conversación con el agente.
Especificar tales prompts fijos para interacciones posteriores con el agente no es
práctico ya que el agente puede responder de diferentes maneras.
En su lugar, el `conversation_plan` proporciona una guía sobre cómo debe proceder el resto de la
conversación con el agente.
Un LLM utiliza este plan de conversación, junto con el historial de conversación, para
generar dinámicamente prompts de usuario hasta que juzgue que la conversación está
completa.

!!! tip "Pruébalo en Colab"

    Prueba este flujo de trabajo completo tú mismo en un cuaderno interactivo sobre
    [Simulating User Conversations to Dynamically Evaluate ADK Agents](https://github.com/google/adk-samples/blob/main/python/notebooks/evaluation/user_simulation_in_adk_evals.ipynb).
    Definirás un escenario de conversación, ejecutarás una "prueba en seco" para verificar el
    diálogo, y luego realizarás una evaluación completa para calificar las respuestas del agente.

## Ejemplo: Evaluando el agente [`hello_world`](https://github.com/google/adk-python/tree/main/contributing/samples/hello_world) con escenarios de conversación

Para agregar casos de evaluación que contengan escenarios de conversación a un
[`EvalSet`](https://github.com/google/adk-python/blob/main/src/google/adk/evaluation/eval_set.py) nuevo o existente,
primero necesitas crear una lista de escenarios de conversación para probar el agente.

Intenta guardar lo siguiente en
`contributing/samples/hello_world/conversation_scenarios.json`:

```json
{
  "scenarios": [
    {
      "starting_prompt": "What can you do for me?",
      "conversation_plan": "Ask the agent to roll a 20-sided die. After you get the result, ask the agent to check if it is prime."
    },
    {
      "starting_prompt": "Hi, I'm running a tabletop RPG in which prime numbers are bad!",
      "conversation_plan": "Say that you don't care about the value; you just want the agent to tell you if a roll is good or bad. Once the agent agrees, ask it to roll a 6-sided die. Finally, ask the agent to do the same with 2 20-sided dice."
    }
  ]
}
```

También necesitarás un archivo de entrada de sesión que contenga información utilizada durante
la evaluación.
Intenta guardar lo siguiente en
`contributing/samples/hello_world/session_input.json`:

```json
{
  "app_name": "hello_world",
  "user_id": "user"
}
```

Luego, puedes agregar los escenarios de conversación a un `EvalSet`:

```bash
# (opcional) crear un nuevo EvalSet
adk eval_set create \
  contributing/samples/hello_world \
  eval_set_with_scenarios

# agregar escenarios de conversación al EvalSet como nuevos casos de evaluación
adk eval_set add_eval_case \
  contributing/samples/hello_world \
  eval_set_with_scenarios \
  --scenarios_file contributing/samples/hello_world/conversation_scenarios.json \
  --session_input_file contributing/samples/hello_world/session_input.json
```

Por defecto, ADK ejecuta evaluaciones con métricas que requieren que se especifique la
respuesta esperada del agente.
Dado que ese no es el caso para un escenario de conversación dinámico, usaremos un
[`EvalConfig`](https://github.com/google/adk-python/blob/main/src/google/adk/evaluation/eval_config.py)
con algunas métricas alternativas soportadas.

Intenta guardar lo siguiente en
`contributing/samples/hello_world/eval_config.json`:

```json
{
  "criteria": {
    "hallucinations_v1": {
      "threshold": 0.5,
      "evaluate_intermediate_nl_responses": true
    },
    "safety_v1": {
      "threshold": 0.8
    }
  }
}
```

Finalmente, puedes usar el comando `adk eval` para ejecutar la evaluación:

```bash
adk eval \
    contributing/samples/hello_world \
    --config_file_path contributing/samples/hello_world/eval_config.json \
    eval_set_with_scenarios \
    --print_detailed_results
```

## Configuración del simulador de usuario

Puedes anular la configuración predeterminada del simulador de usuario para cambiar el modelo,
el comportamiento interno del modelo y el número máximo de interacciones usuario-agente.
El siguiente `EvalConfig` muestra la configuración predeterminada del simulador de usuario:

```json
{
  "criteria": {
    # igual que antes
  },
  "user_simulator_config": {
    "model": "gemini-2.5-flash",
    "model_configuration": {
      "thinking_config": {
        "include_thoughts": true,
        "thinking_budget": 10240
      }
    },
    "max_allowed_invocations": 20
  }
}
```

* `model`: El modelo que respalda el simulador de usuario.
* `model_configuration`: Un
[`GenerateContentConfig`](https://github.com/googleapis/python-genai/blob/6196b1b4251007e33661bb5d7dc27bafee3feefe/google/genai/types.py#L4295)
que controla el comportamiento del modelo.
* `max_allowed_invocations`: El máximo de interacciones usuario-agente permitidas antes de
que la conversación se termine forzosamente. Esto debe configurarse para ser mayor que
la interacción usuario-agente más larga razonable en tu `EvalSet`.