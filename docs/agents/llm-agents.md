# Agente LLM

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span>
  <span class="lst-python">Python v0.1.0</span>
  <span class="lst-typescript">Typescript v0.2.0</span>
  <span class="lst-go">Go v0.1.0</span>
  <span class="lst-java">Java v0.1.0</span>
</div>

El `LlmAgent` (a menudo abreviado simplemente como `Agent`) es un componente central en ADK,
actuando como la parte "pensante" de tu aplicación. Aprovecha el poder de un
Modelo de Lenguaje Grande (LLM) para razonar, comprender lenguaje natural, tomar
decisiones, generar respuestas e interactuar con herramientas.

A diferencia de los [Agentes de Flujo de Trabajo](workflow-agents/index.md) deterministas que siguen
rutas de ejecución predefinidas, el comportamiento del `LlmAgent` es no determinista. Utiliza
el LLM para interpretar instrucciones y contexto, decidiendo dinámicamente cómo
proceder, qué herramientas usar (si las hay), o si transferir el control a otro
agente.

Construir un `LlmAgent` efectivo implica definir su identidad, guiar claramente
su comportamiento a través de instrucciones, y equiparlo con las herramientas y
capacidades necesarias.

## Definiendo la Identidad y Propósito del Agente

Primero, necesitas establecer qué *es* el agente y para qué *sirve*.

* **`name` (Requerido):** Cada agente necesita un identificador de cadena único. Este
  `name` es crucial para operaciones internas, especialmente en sistemas multi-agente
  donde los agentes necesitan referirse o delegar tareas entre sí. Elige un
  nombre descriptivo que refleje la función del agente (por ejemplo,
  `customer_support_router`, `billing_inquiry_agent`). Evita nombres reservados como
  `user`.

* **`description` (Opcional, Recomendado para Multi-Agente):** Proporciona un resumen
  conciso de las capacidades del agente. Esta descripción es utilizada principalmente por
  *otros* agentes LLM para determinar si deben dirigir una tarea a este agente.
  Hazla lo suficientemente específica para diferenciarlo de sus pares (por ejemplo, "Maneja
  consultas sobre estados de facturación actuales", no solo "Agente de facturación").

* **`model` (Requerido):** Especifica el LLM subyacente que impulsará el
  razonamiento de este agente. Este es un identificador de cadena como `"gemini-2.5-flash"`. La
  elección del modelo impacta las capacidades del agente, el costo y el rendimiento. Consulta
  la página de [Modelos](/agents/models/) para opciones disponibles y consideraciones.

=== "Python"

    ```python
    # Ejemplo: Definiendo la identidad básica
    capital_agent = LlmAgent(
        model="gemini-2.5-flash",
        name="capital_agent",
        description="Responde preguntas de usuarios sobre la ciudad capital de un país dado."
        # instruction y tools se agregarán a continuación
    )
    ```

=== "Typescript"

    ```typescript
    // Ejemplo: Definiendo la identidad básica
    const capitalAgent = new LlmAgent({
        model: 'gemini-2.5-flash',
        name: 'capital_agent',
        description: 'Responde preguntas de usuarios sobre la ciudad capital de un país dado.',
        // instruction y tools se agregarán a continuación
    });
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/agents/llm-agents/snippets/main.go:identity"
    ```

=== "Java"

    ```java
    // Ejemplo: Definiendo la identidad básica
    LlmAgent capitalAgent =
        LlmAgent.builder()
            .model("gemini-2.5-flash")
            .name("capital_agent")
            .description("Responde preguntas de usuarios sobre la ciudad capital de un país dado.")
            // instruction y tools se agregarán a continuación
            .build();
    ```

## Guiando al Agente: Instrucciones (`instruction`)

El parámetro `instruction` es probablemente el más crítico para moldear el
comportamiento de un `LlmAgent`. Es una cadena (o una función que devuelve una cadena) que
le dice al agente:

* Su tarea u objetivo principal.
* Su personalidad o persona (por ejemplo, "Eres un asistente útil", "Eres un pirata ingenioso").
* Restricciones en su comportamiento (por ejemplo, "Solo responde preguntas sobre X", "Nunca reveles Y").
* Cómo y cuándo usar sus `tools`. Debes explicar el propósito de cada herramienta y las circunstancias bajo las cuales debe ser llamada, complementando cualquier descripción dentro de la herramienta misma.
* El formato deseado para su salida (por ejemplo, "Responde en JSON", "Proporciona una lista con viñetas").

**Consejos para Instrucciones Efectivas:**

* **Sé Claro y Específico:** Evita la ambigüedad. Declara claramente las acciones y resultados deseados.
* **Usa Markdown:** Mejora la legibilidad para instrucciones complejas usando encabezados, listas, etc.
* **Proporciona Ejemplos (Few-Shot):** Para tareas complejas o formatos de salida específicos, incluye ejemplos directamente en la instrucción.
* **Guía el Uso de Herramientas:** No solo listes herramientas; explica *cuándo* y *por qué* el agente debe usarlas.

**Estado:**

* La instrucción es una plantilla de cadena, puedes usar la sintaxis `{var}` para insertar valores dinámicos en la instrucción.
* `{var}` se usa para insertar el valor de la variable de estado llamada var.
* `{artifact.var}` se usa para insertar el contenido de texto del artefacto llamado var.
* Si la variable de estado o artefacto no existe, el agente generará un error. Si quieres ignorar el error, puedes agregar un `?` al nombre de la variable como en `{var?}`.

=== "Python"

    ```python
    # Ejemplo: Agregando instrucciones
    capital_agent = LlmAgent(
        model="gemini-2.5-flash",
        name="capital_agent",
        description="Responde preguntas de usuarios sobre la ciudad capital de un país dado.",
        instruction="""Eres un agente que proporciona la ciudad capital de un país.
    Cuando un usuario pregunte por la capital de un país:
    1. Identifica el nombre del país de la consulta del usuario.
    2. Usa la herramienta `get_capital_city` para encontrar la capital.
    3. Responde claramente al usuario, indicando la ciudad capital.
    Ejemplo de Consulta: "¿Cuál es la capital de {country}?"
    Ejemplo de Respuesta: "La capital de Francia es París."
    """,
        # tools se agregarán a continuación
    )
    ```

=== "Typescript"

    ```typescript
    // Ejemplo: Agregando instrucciones
    const capitalAgent = new LlmAgent({
        model: 'gemini-2.5-flash',
        name: 'capital_agent',
        description: 'Responde preguntas de usuarios sobre la ciudad capital de un país dado.',
        instruction: `Eres un agente que proporciona la ciudad capital de un país.
            Cuando un usuario pregunte por la capital de un país:
            1. Identifica el nombre del país de la consulta del usuario.
            2. Usa la herramienta \`getCapitalCity\` para encontrar la capital.
            3. Responde claramente al usuario, indicando la ciudad capital.
            Ejemplo de Consulta: "¿Cuál es la capital de {country}?"
            Ejemplo de Respuesta: "La capital de Francia es París."
            `,
        // tools se agregarán a continuación
    });
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/agents/llm-agents/snippets/main.go:instruction"
    ```

=== "Java"

    ```java
    // Ejemplo: Agregando instrucciones
    LlmAgent capitalAgent =
        LlmAgent.builder()
            .model("gemini-2.5-flash")
            .name("capital_agent")
            .description("Responde preguntas de usuarios sobre la ciudad capital de un país dado.")
            .instruction(
                """
                Eres un agente que proporciona la ciudad capital de un país.
                Cuando un usuario pregunte por la capital de un país:
                1. Identifica el nombre del país de la consulta del usuario.
                2. Usa la herramienta `get_capital_city` para encontrar la capital.
                3. Responde claramente al usuario, indicando la ciudad capital.
                Ejemplo de Consulta: "¿Cuál es la capital de {country}?"
                Ejemplo de Respuesta: "La capital de Francia es París."
                """)
            // tools se agregarán a continuación
            .build();
    ```

*(Nota: Para instrucciones que se aplican a *todos* los agentes en un sistema, considera usar
`global_instruction` en el agente raíz, detallado más adelante en la
sección [Multi-Agentes](multi-agents.md).)*

## Equipando al Agente: Herramientas (`tools`)

Las herramientas le dan a tu `LlmAgent` capacidades más allá del conocimiento integrado del LLM o
razonamiento. Permiten al agente interactuar con el mundo exterior, realizar
cálculos, obtener datos en tiempo real o ejecutar acciones específicas.

* **`tools` (Opcional):** Proporciona una lista de herramientas que el agente puede usar. Cada elemento en la lista puede ser:
    * Una función o método nativo (envuelto como un `FunctionTool`). Python ADK automáticamente envuelve la función nativa en un `FunctionTool` mientras que, debes envolver explícitamente tus métodos Java usando `FunctionTool.create(...)`
    * Una instancia de una clase que hereda de `BaseTool`.
    * Una instancia de otro agente (`AgentTool`, habilitando delegación agente-a-agente - ver [Multi-Agentes](multi-agents.md)).

El LLM usa los nombres de función/herramienta, descripciones (de docstrings o el
campo `description`), y esquemas de parámetros para decidir qué herramienta llamar basándose
en la conversación y sus instrucciones.

=== "Python"

    ```python
    # Define una función de herramienta
    def get_capital_city(country: str) -> str:
      """Recupera la ciudad capital para un país dado."""
      # Reemplaza con lógica real (ej., llamada API, búsqueda en base de datos)
      capitals = {"france": "Paris", "japan": "Tokyo", "canada": "Ottawa"}
      return capitals.get(country.lower(), f"Lo siento, no conozco la capital de {country}.")

    # Agrega la herramienta al agente
    capital_agent = LlmAgent(
        model="gemini-2.5-flash",
        name="capital_agent",
        description="Responde preguntas de usuarios sobre la ciudad capital de un país dado.",
        instruction="""Eres un agente que proporciona la ciudad capital de un país... (texto de instrucción anterior)""",
        tools=[get_capital_city] # Proporciona la función directamente
    )
    ```

=== "Typescript"

    ```typescript
    import {z} from 'zod';
    import { LlmAgent, FunctionTool } from '@google/adk';

    // Define el esquema para los parámetros de entrada de la herramienta
    const getCapitalCityParamsSchema = z.object({
        country: z.string().describe('El país para el cual obtener la capital.'),
    });

    // Define la función de herramienta en sí
    async function getCapitalCity(params: z.infer<typeof getCapitalCityParamsSchema>): Promise<{ capitalCity: string }> {
    const capitals: Record<string, string> = {
        'france': 'Paris',
        'japan': 'Tokyo',
        'canada': 'Ottawa',
    };
    const result = capitals[params.country.toLowerCase()] ??
        `Lo siento, no conozco la capital de ${params.country}.`;
    return {capitalCity: result}; // Las herramientas deben devolver un objeto
    }

    // Crea una instancia del FunctionTool
    const getCapitalCityTool = new FunctionTool({
        name: 'getCapitalCity',
        description: 'Recupera la ciudad capital para un país dado.',
        parameters: getCapitalCityParamsSchema,
        execute: getCapitalCity,
    });

    // Agrega la herramienta al agente
    const capitalAgent = new LlmAgent({
        model: 'gemini-2.5-flash',
        name: 'capitalAgent',
        description: 'Responde preguntas de usuarios sobre la ciudad capital de un país dado.',
        instruction: 'Eres un agente que proporciona la ciudad capital de un país...', // Nota: la instrucción completa se omite por brevedad
        tools: [getCapitalCityTool], // Proporciona la instancia de FunctionTool en un array
    });
    ```

=== "Go"

    ```go
    --8<-- "examples/go/snippets/agents/llm-agents/snippets/main.go:tool_example"
    ```

=== "Java"

    ```java

    // Define una función de herramienta
    // Recupera la ciudad capital de un país dado.
    public static Map<String, Object> getCapitalCity(
            @Schema(name = "country", description = "El país para el cual obtener la capital")
            String country) {
      // Reemplaza con lógica real (ej., llamada API, búsqueda en base de datos)
      Map<String, String> countryCapitals = new HashMap<>();
      countryCapitals.put("canada", "Ottawa");
      countryCapitals.put("france", "Paris");
      countryCapitals.put("japan", "Tokyo");

      String result =
              countryCapitals.getOrDefault(
                      country.toLowerCase(), "Lo siento, no pude encontrar la capital de " + country + ".");
      return Map.of("result", result); // Las herramientas deben devolver un Map
    }

    // Agrega la herramienta al agente
    FunctionTool capitalTool = FunctionTool.create(experiment.getClass(), "getCapitalCity");
    LlmAgent capitalAgent =
        LlmAgent.builder()
            .model("gemini-2.5-flash")
            .name("capital_agent")
            .description("Responde preguntas de usuarios sobre la ciudad capital de un país dado.")
            .instruction("Eres un agente que proporciona la ciudad capital de un país... (texto de instrucción anterior)")
            .tools(capitalTool) // Proporciona la función envuelta como un FunctionTool
            .build();
    ```

Aprende más sobre Herramientas en la sección [Herramientas](../tools/index.md).

## Configuración y Control Avanzados

Más allá de los parámetros centrales, `LlmAgent` ofrece varias opciones para un control más fino:

### Ajustando la Generación del LLM (`generate_content_config`)

Puedes ajustar cómo el LLM subyacente genera respuestas usando `generate_content_config`.

* **`generate_content_config` (Opcional):** Pasa una instancia de [`google.genai.types.GenerateContentConfig`](https://googleapis.github.io/python-genai/genai.html#genai.types.GenerateContentConfig) para controlar parámetros como `temperature` (aleatoriedad), `max_output_tokens` (longitud de respuesta), `top_p`, `top_k`, y configuraciones de seguridad.

=== "Python"

    ```python
    from google.genai import types

    agent = LlmAgent(
        # ... otros parámetros
        generate_content_config=types.GenerateContentConfig(
            temperature=0.2, # Salida más determinista
            max_output_tokens=250,
            safety_settings=[
                types.SafetySetting(
                    category=types.HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT,
                    threshold=types.HarmBlockThreshold.BLOCK_LOW_AND_ABOVE,
                )
            ]
        )
    )
    ```

=== "Typescript"

    ```typescript
    import { GenerateContentConfig } from '@google/genai';

    const generateContentConfig: GenerateContentConfig = {
        temperature: 0.2, // Salida más determinista
        maxOutputTokens: 250,
    };

    const agent = new LlmAgent({
        // ... otros parámetros
        generateContentConfig,
    });
    ```

=== "Go"

    ```go
    import "google.golang.org/genai"

    --8<-- "examples/go/snippets/agents/llm-agents/snippets/main.go:gen_config"
    ```

=== "Java"

    ```java
    import com.google.genai.types.GenerateContentConfig;

    LlmAgent agent =
        LlmAgent.builder()
            // ... otros parámetros
            .generateContentConfig(GenerateContentConfig.builder()
                .temperature(0.2F) // Salida más determinista
                .maxOutputTokens(250)
                .build())
            .build();
    ```

### Estructurando Datos (`input_schema`, `output_schema`, `output_key`)

Para escenarios que requieren intercambio de datos estructurados con un `LLM Agent`, el ADK proporciona mecanismos para definir formatos de entrada esperados y salida deseados usando definiciones de esquema.

* **`input_schema` (Opcional):** Define un esquema que representa la estructura de entrada esperada. Si se establece, el contenido del mensaje del usuario pasado a este agente *debe* ser una cadena JSON que cumpla con este esquema. Tus instrucciones deben guiar al usuario o agente precedente en consecuencia.

* **`output_schema` (Opcional):** Define un esquema que representa la estructura de salida deseada. Si se establece, la respuesta final del agente *debe* ser una cadena JSON que cumpla con este esquema.

* **`output_key` (Opcional):** Proporciona una clave de cadena. Si se establece, el contenido de texto de la respuesta *final* del agente se guardará automáticamente en el diccionario de estado de la sesión bajo esta clave. Esto es útil para pasar resultados entre agentes o pasos en un flujo de trabajo.
    * En Python, esto podría verse como: `session.state[output_key] = agent_response_text`
    * En Java: `session.state().put(outputKey, agentResponseText)`
    * En Golang, dentro de un manejador de callback: `ctx.State().Set(output_key, agentResponseText)`

=== "Python"

    El esquema de entrada y salida es típicamente un `Pydantic` BaseModel.

    ```python
    from pydantic import BaseModel, Field

    class CapitalOutput(BaseModel):
        capital: str = Field(description="La capital del país.")

    structured_capital_agent = LlmAgent(
        # ... name, model, description
        instruction="""Eres un Agente de Información de Capitales. Dado un país, responde SOLO con un objeto JSON que contenga la capital. Formato: {"capital": "nombre_capital"}""",
        output_schema=CapitalOutput, # Forzar salida JSON
        output_key="found_capital"  # Almacenar resultado en state['found_capital']
        # No se puede usar tools=[get_capital_city] efectivamente aquí
    )
    ```

=== "Typescript"

    ```typescript
    import {z} from 'zod';
    import { Schema, Type } from '@google/genai';

    // Define el esquema para la salida
    const CapitalOutputSchema: Schema = {
        type: Type.OBJECT,
        properties: {
            capital: {
                type: Type.STRING,
                description: 'La capital del país.',
            },
        },
        required: ['capital'],
    };

    // Crea la instancia de LlmAgent
    const structuredCapitalAgent = new LlmAgent({
        // ... name, model, description
        instruction: `Eres un Agente de Información de Capitales. Dado un país, responde SOLO con un objeto JSON que contenga la capital. Formato: {"capital": "nombre_capital"}`,
        outputSchema: CapitalOutputSchema, // Forzar salida JSON
        outputKey: 'found_capital', // Almacenar resultado en state['found_capital']
        // No se pueden usar tools efectivamente aquí
    });
    ```

=== "Go"

    El esquema de entrada y salida es un objeto `google.genai.types.Schema`.

    ```go
    --8<-- "examples/go/snippets/agents/llm-agents/snippets/main.go:schema_example"
    ```

=== "Java"

     El esquema de entrada y salida es un objeto `google.genai.types.Schema`.

    ```java
    private static final Schema CAPITAL_OUTPUT =
        Schema.builder()
            .type("OBJECT")
            .description("Esquema para información de ciudad capital.")
            .properties(
                Map.of(
                    "capital",
                    Schema.builder()
                        .type("STRING")
                        .description("La ciudad capital del país.")
                        .build()))
            .build();

    LlmAgent structuredCapitalAgent =
        LlmAgent.builder()
            // ... name, model, description
            .instruction(
                    "Eres un Agente de Información de Capitales. Dado un país, responde SOLO con un objeto JSON que contenga la capital. Formato: {\"capital\": \"nombre_capital\"}")
            .outputSchema(capitalOutput) // Forzar salida JSON
            .outputKey("found_capital") // Almacenar resultado en state.get("found_capital")
            // No se puede usar tools(getCapitalCity) efectivamente aquí
            .build();
    ```

### Gestionando Contexto (`include_contents`)

Controla si el agente recibe el historial de conversación anterior.

* **`include_contents` (Opcional, Por defecto: `'default'`):** Determina si los `contents` (historial) se envían al LLM.
    * `'default'`: El agente recibe el historial de conversación relevante.
    * `'none'`: El agente no recibe `contents` anteriores. Opera basándose únicamente en su instrucción actual y cualquier entrada proporcionada en el turno *actual* (útil para tareas sin estado o para aplicar contextos específicos).

=== "Python"

    ```python
    stateless_agent = LlmAgent(
        # ... otros parámetros
        include_contents='none'
    )
    ```

=== "Typescript"

    ```typescript
    const statelessAgent = new LlmAgent({
        // ... otros parámetros
        includeContents: 'none',
    });
    ```

=== "Go"

    ```go
    import "google.golang.org/adk/agent/llmagent"

    --8<-- "examples/go/snippets/agents/llm-agents/snippets/main.go:include_contents"
    ```

=== "Java"

    ```java
    import com.google.adk.agents.LlmAgent.IncludeContents;

    LlmAgent statelessAgent =
        LlmAgent.builder()
            // ... otros parámetros
            .includeContents(IncludeContents.NONE)
            .build();
    ```

### Planificador

<div class="language-support-tag" title="">
   <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span>
</div>

**`planner` (Opcional):** Asigna una instancia de `BasePlanner` para habilitar razonamiento y planificación de múltiples pasos antes de la ejecución. Hay dos planificadores principales:

* **`BuiltInPlanner`:** Aprovecha las capacidades de planificación integradas del modelo (por ejemplo, la función de pensamiento de Gemini). Ver [Gemini Thinking](https://ai.google.dev/gemini-api/docs/thinking) para detalles y ejemplos.

    Aquí, el parámetro `thinking_budget` guía al modelo sobre el número de tokens de pensamiento a usar al generar una respuesta. El parámetro `include_thoughts` controla si el modelo debe incluir sus pensamientos crudos y proceso de razonamiento interno en la respuesta.

    ```python
    from google.adk import Agent
    from google.adk.planners import BuiltInPlanner
    from google.genai import types

    my_agent = Agent(
        model="gemini-2.5-flash",
        planner=BuiltInPlanner(
            thinking_config=types.ThinkingConfig(
                include_thoughts=True,
                thinking_budget=1024,
            )
        ),
        # ... tus herramientas aquí
    )
    ```

* **`PlanReActPlanner`:** Este planificador instruye al modelo para seguir una estructura específica en su salida: primero crear un plan, luego ejecutar acciones (como llamar herramientas), y proporcionar razonamiento para sus pasos. *Es particularmente útil para modelos que no tienen una función de "pensamiento" integrada*.

    ```python
    from google.adk import Agent
    from google.adk.planners import PlanReActPlanner

    my_agent = Agent(
        model="gemini-2.5-flash",
        planner=PlanReActPlanner(),
        # ... tus herramientas aquí
    )
    ```

    La respuesta del agente seguirá un formato estructurado:

    ```
    [user]: ai news
    [google_search_agent]: /*PLANNING*/
    1. Realizar una búsqueda en Google de "últimas noticias de IA" para obtener actualizaciones actuales y titulares relacionados con inteligencia artificial.
    2. Sintetizar la información de los resultados de búsqueda para proporcionar un resumen de noticias recientes de IA.

    /*ACTION*/
    /*REASONING*/
    Los resultados de búsqueda proporcionan una visión general completa de noticias recientes de IA, cubriendo varios aspectos como desarrollos de empresas, avances en investigación y aplicaciones. Tengo suficiente información para responder a la solicitud del usuario.

    /*FINAL_ANSWER*/
    Aquí hay un resumen de noticias recientes de IA:
    ....
    ```

Ejemplo para usar built-in-planner:

```python
from dotenv import load_dotenv


import asyncio
import os

from google.genai import types
from google.adk.agents.llm_agent import LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.artifacts.in_memory_artifact_service import InMemoryArtifactService # Opcional
from google.adk.planners import BasePlanner, BuiltInPlanner, PlanReActPlanner
from google.adk.models import LlmRequest

from google.genai.types import ThinkingConfig
from google.genai.types import GenerateContentConfig

import datetime
from zoneinfo import ZoneInfo

APP_NAME = "weather_app"
USER_ID = "1234"
SESSION_ID = "session1234"

def get_weather(city: str) -> dict:
    """Recupera el reporte de clima actual para una ciudad especificada.

    Args:
        city (str): El nombre de la ciudad para la cual recuperar el reporte de clima.

    Returns:
        dict: status y result o msg de error.
    """
    if city.lower() == "new york":
        return {
            "status": "success",
            "report": (
                "El clima en Nueva York es soleado con una temperatura de 25 grados"
                " Celsius (77 grados Fahrenheit)."
            ),
        }
    else:
        return {
            "status": "error",
            "error_message": f"La información del clima para '{city}' no está disponible.",
        }


def get_current_time(city: str) -> dict:
    """Devuelve la hora actual en una ciudad especificada.

    Args:
        city (str): El nombre de la ciudad para la cual recuperar la hora actual.

    Returns:
        dict: status y result o msg de error.
    """

    if city.lower() == "new york":
        tz_identifier = "America/New_York"
    else:
        return {
            "status": "error",
            "error_message": (
                f"Lo siento, no tengo información de zona horaria para {city}."
            ),
        }

    tz = ZoneInfo(tz_identifier)
    now = datetime.datetime.now(tz)
    report = (
        f'La hora actual en {city} es {now.strftime("%Y-%m-%d %H:%M:%S %Z%z")}'
    )
    return {"status": "success", "report": report}

# Paso 1: Crear un ThinkingConfig
thinking_config = ThinkingConfig(
    include_thoughts=True,   # Pedir al modelo que incluya sus pensamientos en la respuesta
    thinking_budget=256      # Limitar el 'pensamiento' a 256 tokens (ajustar según sea necesario)
)
print("ThinkingConfig:", thinking_config)

# Paso 2: Instanciar BuiltInPlanner
planner = BuiltInPlanner(
    thinking_config=thinking_config
)
print("BuiltInPlanner creado.")

# Paso 3: Envolver el planificador en un LlmAgent
agent = LlmAgent(
    model="gemini-2.5-pro-preview-03-25",  # Establece el nombre de tu modelo
    name="weather_and_time_agent",
    instruction="Eres un agente que devuelve hora y clima",
    planner=planner,
    tools=[get_weather, get_current_time]
)

# Session y Runner
session_service = InMemorySessionService()
session = session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
runner = Runner(agent=agent, app_name=APP_NAME, session_service=session_service)

# Interacción con el Agente
def call_agent(query):
    content = types.Content(role='user', parts=[types.Part(text=query)])
    events = runner.run(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    for event in events:
        print(f"\nDEBUG EVENT: {event}\n")
        if event.is_final_response() and event.content:
            final_answer = event.content.parts[0].text.strip()
            print("\n🟢 FINAL ANSWER\n", final_answer, "\n")

call_agent("Si está lloviendo en Nueva York ahora mismo, ¿cuál es la temperatura actual?")

```

### Ejecución de Código

<div class="language-support-tag">
   <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

- **`code_executor` (Opcional):** Proporciona una instancia de `BaseCodeExecutor` para permitir
  al agente ejecutar bloques de código encontrados en la respuesta del LLM. Para más
  información, ver [Ejecución de Código con Gemini API](/tools/gemini-api/code-execution/).

=== "Python"

    ```python
    --8<-- "examples/python/snippets/tools/built-in-tools/code_execution.py"
    ```

=== "Java"

    ```java
    --8<-- "examples/java/snippets/src/main/java/tools/CodeExecutionAgentApp.java:full_code"
    ```

## Juntando Todo: Ejemplo

??? "Code"
    Aquí está el `capital_agent` básico completo:

    === "Python"

        ```python
        --8<-- "examples/python/snippets/agents/llm-agent/capital_agent.py"
        ```

    === "Typescript"

        ```javascript
        --8<-- "examples/typescript/snippets/agents/llm-agent/capital_agent.ts"
        ```

    === "Go"

        ```go
        --8<-- "examples/go/snippets/agents/llm-agents/main.go:full_code"
        ```

    === "Java"

        ```java
        --8<-- "examples/java/snippets/src/main/java/agents/LlmAgentExample.java:full_code"
        ```

_(Este ejemplo demuestra los conceptos centrales. Agentes más complejos podrían incorporar esquemas, control de contexto, planificación, etc.)_

## Conceptos Relacionados (Temas Diferidos)

Mientras esta página cubre la configuración central de `LlmAgent`, varios conceptos relacionados proporcionan control más avanzado y se detallan en otros lugares:

* **Callbacks:** Interceptar puntos de ejecución (antes/después de llamadas al modelo, antes/después de llamadas a herramientas) usando `before_model_callback`, `after_model_callback`, etc. Ver [Callbacks](../callbacks/types-of-callbacks.md).
* **Control Multi-Agente:** Estrategias avanzadas para interacción de agentes, incluyendo planificación (`planner`), control de transferencia de agentes (`disallow_transfer_to_parent`, `disallow_transfer_to_peers`), e instrucciones de todo el sistema (`global_instruction`). Ver [Multi-Agentes](multi-agents.md).