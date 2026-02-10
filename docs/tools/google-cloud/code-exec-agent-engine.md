---
catalog_title: Code Execution Tool with Agent Engine
catalog_description: Run AI-generated code in a secure and scalable GKE environment
catalog_icon: /assets/tools-vertex-ai.png
---

# Herramienta de Ejecución de Código con Agent Engine

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v1.17.0</span><span class="lst-preview">Preview</span>
</div>

La Herramienta ADK de Ejecución de Código de Agent Engine proporciona un método de baja latencia y alta eficiencia para ejecutar código generado por IA utilizando el servicio [Google Cloud Agent Engine](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/overview). Esta herramienta está diseñada para una ejecución rápida, adaptada a flujos de trabajo agénticos, y utiliza entornos aislados para mejorar la seguridad. La herramienta de Ejecución de Código permite que el código y los datos persistan a lo largo de múltiples solicitudes, permitiendo tareas de codificación complejas y de múltiples pasos, incluyendo:

-   **Desarrollo y depuración de código:** Crea tareas de agente que prueban e iteran sobre versiones de código a lo largo de múltiples solicitudes.
-   **Código con análisis de datos:** Carga archivos de datos de hasta 100MB, y ejecuta múltiples análisis basados en código sin necesidad de recargar datos para cada ejecución de código.

Esta herramienta de ejecución de código es parte de la suite de Agent Engine, sin embargo no es necesario desplegar tu agente en Agent Engine para usarla. Puedes ejecutar tu agente localmente o con otros servicios y usar esta herramienta. Para más información sobre la característica de Ejecución de Código en Agent Engine, consulta la documentación de [Agent Engine Code Execution](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/code-execution/overview).

!!! example "Preview release"
    La característica de Ejecución de Código de Agent Engine es una versión Preview. Para más información, consulta las [descripciones de etapa de lanzamiento](https://cloud.google.com/products#product-launch-stages).

## Usar la Herramienta

El uso de la herramienta de Ejecución de Código de Agent Engine requiere que crees un entorno sandbox con Google Cloud Agent Engine antes de usar la herramienta con un agente ADK.

Para usar la herramienta de Ejecución de Código con tu agente ADK:

1.  Sigue las instrucciones en el [Code Execution quickstart](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/code-execution/quickstart) de Agent Engine para crear un entorno sandbox de ejecución de código.
1.  Crea un agente ADK con configuraciones para acceder al proyecto de Google Cloud donde creaste el entorno sandbox.
1.  El siguiente ejemplo de código muestra un agente configurado para usar la herramienta Code Executor. Reemplaza `SANDBOX_RESOURCE_NAME` con el nombre de recurso del entorno sandbox que creaste.

```python
from google.adk.agents.llm_agent import Agent
from google.adk.code_executors.agent_engine_sandbox_code_executor import AgentEngineSandboxCodeExecutor

root_agent = Agent(
    model="gemini-2.5-flash",
    name="agent_engine_code_execution_agent",
    instruction="You are a helpful agent that can write and execute code to answer questions and solve problems.",
    code_executor=AgentEngineSandboxCodeExecutor(
        sandbox_resource_name="SANDBOX_RESOURCE_NAME",
    ),
)
```

Para detalles sobre el formato esperado del valor `sandbox_resource_name`, y el parámetro alternativo `agent_engine_resource_name`, consulta [Parámetros de configuración](#config-parameters). Para un ejemplo más avanzado, incluyendo instrucciones de sistema recomendadas para la herramienta, consulta el [Ejemplo avanzado](#advanced-example) o el [ejemplo de código de agente](https://github.com/google/adk-python/tree/main/contributing/samples/agent_engine_code_execution) completo.

## Cómo funciona

La Herramienta `AgentEngineCodeExecutor` mantiene un único sandbox durante toda la tarea de un agente, lo que significa que el estado del sandbox persiste a través de todas las operaciones dentro de una sesión de flujo de trabajo ADK.

1.  **Creación de sandbox:** Para tareas de múltiples pasos que requieren ejecución de código, Agent Engine crea un sandbox con configuraciones de lenguaje y máquina especificadas, aislando el entorno de ejecución de código. Si no se ha creado ningún sandbox previamente, la herramienta de ejecución de código creará uno automáticamente usando configuraciones predeterminadas.
1.  **Ejecución de código con persistencia:** El código generado por IA para una llamada de herramienta se transmite al sandbox y luego se ejecuta dentro del entorno aislado. Después de la ejecución, el sandbox *permanece activo* para llamadas de herramienta subsecuentes dentro de la misma sesión, preservando variables, módulos importados y estado de archivos para la siguiente llamada de herramienta del mismo agente.
1.  **Recuperación de resultados:** La salida estándar y cualquier flujo de error capturado se recopilan y se devuelven al agente que los llamó.
1.  **Limpieza de sandbox:** Una vez que la tarea del agente o conversación concluye, el agente puede eliminar explícitamente el sandbox, o depender de la característica TTL del sandbox especificada al crear el sandbox.

## Beneficios clave

-   **Estado persistente:** Resuelve tareas complejas donde la manipulación de datos o el contexto de variables debe mantenerse entre múltiples llamadas de herramienta.
-   **Aislamiento dirigido:** Proporciona aislamiento robusto a nivel de proceso, asegurando que la ejecución de código de herramienta sea segura mientras permanece ligera.
-   **Integración con Agent Engine:** Estrechamente integrada en la capa de orquestación y uso de herramientas de Agent Engine.
-   **Rendimiento de baja latencia:** Diseñada para velocidad, permitiendo que los agentes ejecuten flujos de trabajo complejos de uso de herramientas eficientemente sin sobrecarga significativa.
-   **Configuraciones de cómputo flexibles:** Crea sandboxes con lenguaje de programación, poder de procesamiento y configuraciones de memoria específicas.

## Requisitos del sistema¶

Los siguientes requisitos deben cumplirse para usar exitosamente la herramienta de Ejecución de Código de Agent Engine con tus agentes ADK:

-   Proyecto de Google Cloud con la API de Vertex habilitada
-   La cuenta de servicio del agente requiere el rol **roles/aiplatform.user**, que le permite:
    -   Crear, obtener, listar y eliminar sandboxes de ejecución de código
    -   Ejecutar sandbox de ejecución de código

## Parámetros de configuración {#config-parameters}

La herramienta de Ejecución de Código de Agent Engine tiene los siguientes parámetros. Debes configurar uno de los siguientes parámetros de recurso:

-   **`sandbox_resource_name`** : Una ruta de recurso de sandbox a un entorno sandbox existente que usa para cada llamada de herramienta. El formato de cadena esperado es el siguiente:
    ```
    projects/{$PROJECT_ID}/locations/{$LOCATION_ID}/reasoningEngines/{$REASONING_ENGINE_ID}/sandboxEnvironments/{$SANDBOX_ENVIRONMENT_ID}
    
    # Ejemplo:
    projects/my-vertex-agent-project/locations/us-central1/reasoningEngines/6842888880301111172/sandboxEnvironments/6545148888889161728
    ```
-   **`agent_engine_resource_name`**: Nombre de recurso de Agent Engine donde la herramienta crea un entorno sandbox. El formato de cadena esperado es el siguiente:
    ```
    projects/{$PROJECT_ID}/locations/{$LOCATION_ID}/reasoningEngines/{$REASONING_ENGINE_ID}
    
    # Ejemplo:
    projects/my-vertex-agent-project/locations/us-central1/reasoningEngines/6842888880301111172
    ```

Puedes usar la API de Google Cloud Agent Engine para configurar entornos sandbox de Agent Engine por separado usando una conexión de cliente de Google Cloud, incluyendo las siguientes configuraciones:

-   **Lenguajes de programación,** incluyendo Python y JavaScript
-   **Entorno de cómputo**, incluyendo tamaños de CPU y memoria

Para más información sobre cómo conectarse a Google Cloud Agent Engine y configurar entornos sandbox, consulta el [Code Execution quickstart](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/code-execution/quickstart#create_a_sandbox) de Agent Engine.

## Ejemplo avanzado {#advanced-example}

El siguiente ejemplo de código muestra cómo implementar el uso de la herramienta Code Executor en un agente ADK. Este ejemplo incluye una cláusula `base_system_instruction` para establecer las pautas operativas para la ejecución de código. Esta cláusula de instrucción es opcional, pero fuertemente recomendada para obtener los mejores resultados de esta herramienta.

```python
from google.adk.agents.llm_agent import Agent
from google.adk.code_executors.agent_engine_sandbox_code_executor import AgentEngineSandboxCodeExecutor

def base_system_instruction():
  """Devuelve: instrucción de sistema del agente de ciencia de datos."""

  return """
  # Pautas

  **Objetivo:** Ayudar al usuario a alcanzar sus objetivos de análisis de datos, **con énfasis en evitar suposiciones y garantizar precisión.** Alcanzar ese objetivo puede involucrar múltiples pasos. Cuando necesites generar código, **no** necesitas resolver el objetivo de una sola vez. Solo genera el siguiente paso a la vez.

  **Ejecución de Código:** Todos los fragmentos de código proporcionados se ejecutarán dentro del entorno sandbox.

  **Estado:** Todos los fragmentos de código se ejecutan y las variables permanecen en el entorno. NUNCA necesitas reinicializar variables. NUNCA necesitas recargar archivos. NUNCA necesitas reimportar bibliotecas.

  **Visibilidad de Salida:** Siempre imprime la salida de la ejecución de código para visualizar resultados, especialmente para exploración y análisis de datos. Por ejemplo:
    - Para ver la forma de un pandas.DataFrame haz:
      ```tool_code
      print(df.shape)
      ```
      La salida se te presentará como:
      ```tool_outputs
      (49, 7)

      ```
    - Para mostrar el resultado de un cálculo numérico:
      ```tool_code
      x = 10 ** 9 - 12 ** 5
      print(f'{{x=}}')
      ```
      La salida se te presentará como:
      ```tool_outputs
      x=999751168

      ```
    - Tú **nunca** generas ```tool_outputs tú mismo.
    - Luego puedes usar esta salida para decidir los siguientes pasos.
    - Imprime solo variables (ej., `print(f'{{variable=}}')`.

  **Sin Suposiciones:** **Crucialmente, evita hacer suposiciones sobre la naturaleza de los datos o nombres de columnas.** Basa los hallazgos únicamente en los datos mismos. Siempre usa la información obtenida de `explore_df` para guiar tu análisis.

  **Archivos disponibles:** Solo usa los archivos que estén disponibles según lo especificado en la lista de archivos disponibles.

  **Datos en el prompt:** Algunas consultas contienen los datos de entrada directamente en el prompt. Tienes que analizar esos datos en un pandas DataFrame. SIEMPRE analiza todos los datos. NUNCA edites los datos que se te dan.

  **Respondibilidad:** Algunas consultas pueden no ser respondibles con los datos disponibles. En esos casos, informa al usuario por qué no puedes procesar su consulta y sugiere qué tipo de datos se necesitarían para cumplir su solicitud.

  """

root_agent = Agent(
    model="gemini-2.5-flash",
    name="agent_engine_code_execution_agent",
    instruction=base_system_instruction() + """


Necesitas ayudar al usuario con sus consultas mirando los datos y el contexto en la conversación.
Tu respuesta final debe resumir el código y la ejecución de código relevante a la consulta del usuario.

Debes incluir todas las piezas de datos para responder la consulta del usuario, como la tabla de los resultados de la ejecución de código.
Si no puedes responder la pregunta directamente, debes seguir las pautas anteriores para generar el siguiente paso.
Si la pregunta puede ser respondida directamente sin escribir ningún código, debes hacer eso.
Si no tienes suficientes datos para responder la pregunta, debes pedir aclaración al usuario.

NUNCA debes instalar ningún paquete por tu cuenta como `pip install ...`.
Al graficar tendencias, debes asegurarte de ordenar y organizar los datos por el eje x.


""",
    code_executor=AgentEngineSandboxCodeExecutor(
        # Reemplaza con el nombre de recurso de tu sandbox si ya tienes uno.
        sandbox_resource_name="SANDBOX_RESOURCE_NAME",
        # Reemplaza con el nombre de recurso de agent engine usado para crear sandbox si
        # sandbox_resource_name no está configurado:
        # agent_engine_resource_name="AGENT_ENGINE_RESOURCE_NAME",
    ),
)
```

Para una versión completa de un agente ADK usando este código de ejemplo, consulta el [ejemplo de agent_engine_code_execution](https://github.com/google/adk-python/tree/main/contributing/samples/agent_engine_code_execution).