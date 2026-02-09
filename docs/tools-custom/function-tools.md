# Herramientas de función

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Cuando las herramientas predefinidas de ADK no cumplen con tus requisitos, puedes crear *herramientas de función* personalizadas. Construir herramientas de función te permite crear funcionalidad personalizada, como conectarte a bases de datos propietarias o implementar algoritmos únicos.
Por ejemplo, una herramienta de función, `myfinancetool`, podría ser una función que calcula una métrica financiera específica. ADK también soporta funciones de larga ejecución, por lo que si ese cálculo toma un tiempo, el agente puede continuar trabajando en otras tareas.

ADK ofrece varias formas de crear herramientas de función, cada una adecuada para diferentes niveles de complejidad y control:

*  [Herramientas de Función](#function-tool)
*  [Herramientas de Función de Larga Ejecución](#long-run-tool)
*  [Agentes como Herramienta](#agent-tool)

## Herramientas de Función {#function-tool}

Transformar una función de Python en una herramienta es una forma directa de integrar lógica personalizada en tus agentes. Cuando asignas una función a la lista `tools` de un agente, el framework la envuelve automáticamente como un `FunctionTool`.

### Cómo funciona

El framework ADK inspecciona automáticamente la firma de tu función de Python—incluyendo su nombre, docstring, parámetros, anotaciones de tipo y valores predeterminados—para generar un esquema. Este esquema es lo que el LLM usa para entender el propósito de la herramienta, cuándo usarla y qué argumentos requiere.

### Definir firmas de función

Una firma de función bien definida es crucial para que el LLM use tu herramienta correctamente.

#### Parámetros

##### Parámetros requeridos

=== "Python"
    Un parámetro se considera **requerido** si tiene una anotación de tipo pero **no tiene valor predeterminado**. El LLM debe proporcionar un valor para este argumento cuando llama a la herramienta. La descripción del parámetro se toma del docstring de la función.

    ???+ "Ejemplo: Parámetros requeridos"
        ```python
        def get_weather(city: str, unit: str):
            """
            Retrieves the weather for a city in the specified unit.

            Args:
                city (str): The city name.
                unit (str): The temperature unit, either 'Celsius' or 'Fahrenheit'.
            """
            # ... function logic ...
            return {"status": "success", "report": f"Weather for {city} is sunny."}
        ```
    En este ejemplo, tanto `city` como `unit` son obligatorios. Si el LLM intenta llamar a `get_weather` sin uno de ellos, el ADK devolverá un error al LLM, solicitándole que corrija la llamada.

=== "Go"
    En Go, usas etiquetas de struct para controlar el esquema JSON. Las dos etiquetas principales son `json` y `jsonschema`.

    Un parámetro se considera **requerido** si su campo de struct **no** tiene la opción `omitempty` o `omitzero` en su etiqueta `json`.

    La etiqueta `jsonschema` se usa para proporcionar la descripción del argumento. Esto es crucial para que el LLM entienda para qué sirve el argumento.

    ???+ "Ejemplo: Parámetros requeridos"
        ```go
        // GetWeatherParams defines the arguments for the getWeather tool.
        type GetWeatherParams struct {
            // This field is REQUIRED (no "omitempty").
            // The jsonschema tag provides the description.
            Location string `json:"location" jsonschema:"The city and state, e.g., San Francisco, CA"`

            // This field is also REQUIRED.
            Unit     string `json:"unit" jsonschema:"The temperature unit, either 'celsius' or 'fahrenheit'"`
        }
        ```
    En este ejemplo, tanto `location` como `unit` son obligatorios.

##### Parámetros opcionales

=== "Python"
    Un parámetro se considera **opcional** si proporcionas un **valor predeterminado**. Esta es la forma estándar de Python para definir argumentos opcionales. También puedes marcar un parámetro como opcional usando `typing.Optional[SomeType]` o la sintaxis `| None` (Python 3.10+).

    ???+ "Ejemplo: Parámetros opcionales"
        ```python
        def search_flights(destination: str, departure_date: str, flexible_days: int = 0):
            """
            Searches for flights.

            Args:
                destination (str): The destination city.
                departure_date (str): The desired departure date.
                flexible_days (int, optional): Number of flexible days for the search. Defaults to 0.
            """
            # ... function logic ...
            if flexible_days > 0:
                return {"status": "success", "report": f"Found flexible flights to {destination}."}
            return {"status": "success", "report": f"Found flights to {destination} on {departure_date}."}
        ```
    Aquí, `flexible_days` es opcional. El LLM puede elegir proporcionarlo, pero no es requerido.

=== "Go"
    Un parámetro se considera **opcional** si su campo de struct tiene la opción `omitempty` o `omitzero` en su etiqueta `json`.

    ???+ "Ejemplo: Parámetros opcionales"
        ```go
        // GetWeatherParams defines the arguments for the getWeather tool.
        type GetWeatherParams struct {
            // Location is required.
            Location string `json:"location" jsonschema:"The city and state, e.g., San Francisco, CA"`

            // Unit is optional.
            Unit string `json:"unit,omitempty" jsonschema:"The temperature unit, either 'celsius' or 'fahrenheit'"`

            // Days is optional.
            Days int `json:"days,omitzero" jsonschema:"The number of forecast days to return (defaults to 1)"`
        }
        ```
    Aquí, `unit` y `days` son opcionales. El LLM puede elegir proporcionarlos, pero no son requeridos.

##### Parámetros opcionales con `typing.Optional`
También puedes marcar un parámetro como opcional usando `typing.Optional[SomeType]` o la sintaxis `| None` (Python 3.10+). Esto indica que el parámetro puede ser `None`. Cuando se combina con un valor predeterminado de `None`, se comporta como un parámetro opcional estándar.

???+ "Ejemplo: `typing.Optional`"
    === "Python"
        ```python
        from typing import Optional

        def create_user_profile(username: str, bio: Optional[str] = None):
            """
            Creates a new user profile.

            Args:
                username (str): The user's unique username.
                bio (str, optional): A short biography for the user. Defaults to None.
            """
            # ... function logic ...
            if bio:
                return {"status": "success", "message": f"Profile for {username} created with a bio."}
            return {"status": "success", "message": f"Profile for {username} created."}
        ```

##### Parámetros variádicos (`*args` y `**kwargs`)
Si bien puedes incluir `*args` (argumentos posicionales variables) y `**kwargs` (argumentos de palabra clave variables) en la firma de tu función para otros propósitos, son **ignorados por el framework ADK** al generar el esquema de herramienta para el LLM. El LLM no será consciente de ellos y no puede pasarles argumentos. Es mejor confiar en parámetros definidos explícitamente para todos los datos que esperas del LLM.

#### Tipo de retorno

El tipo de retorno preferido para una Herramienta de Función es un **diccionario** en Python, un **Map** en Java, o un **objeto** en TypeScript. Esto te permite estructurar la respuesta con pares clave-valor, proporcionando contexto y claridad al LLM. Si tu función devuelve un tipo diferente a un diccionario, el framework automáticamente lo envuelve en un diccionario con una única clave llamada **"result"**.

Esfuérzate por hacer tus valores de retorno lo más descriptivos posible. *Por ejemplo,* en lugar de devolver un código de error numérico, devuelve un diccionario con una clave "error_message" que contenga una explicación legible por humanos. **Recuerda que el LLM**, no un fragmento de código, necesita entender el resultado. Como mejor práctica, incluye una clave "status" en tu diccionario de retorno para indicar el resultado general (ej., "success", "error", "pending"), proporcionando al LLM una señal clara sobre el estado de la operación.

#### Docstrings

El docstring de tu función sirve como la **descripción** de la herramienta y se envía al LLM. Por lo tanto, un docstring bien escrito y completo es crucial para que el LLM entienda cómo usar la herramienta efectivamente. Explica claramente el propósito de la función, el significado de sus parámetros y los valores de retorno esperados.

### Pasar datos entre herramientas

Cuando un agente llama a múltiples herramientas en una secuencia, podrías necesitar pasar datos de una herramienta a otra. La forma recomendada de hacer esto es usando el prefijo `temp:` en el estado de sesión.

Una herramienta puede escribir datos en una variable `temp:`, y una herramienta subsecuente puede leerla. Estos datos solo están disponibles para la invocación actual y se descartan después.

!!! note "Contexto de invocación compartido"
    Todas las llamadas a herramientas dentro de un único turno de agente comparten el mismo `InvocationContext`. Esto significa que también comparten el mismo estado temporal (`temp:`), que es cómo se pueden pasar datos entre ellas.

### Ejemplo

??? "Ejemplo"

    === "Python"

        Esta herramienta es una función de python que obtiene el precio de una acción dado un ticker/símbolo de acción.

        <u>Nota</u>: Necesitas instalar la biblioteca `pip install yfinance` antes de usar esta herramienta.

        ```python
        --8<-- "examples/python/snippets/tools/function-tools/func_tool.py"
        ```

        El valor de retorno de esta herramienta será envuelto en un diccionario.

        ```json
        {"result": "$123"}
        ```

    === "Typescript"

        Esta herramienta recupera el valor simulado de un precio de acción.

        ```typescript
        --8<-- "examples/typescript/snippets/tools/function-tools/function-tools-example.ts"
        ```

        El valor de retorno de esta herramienta será un objeto.

        ```json
        For input `GOOG`: {"price": 2800.0, "currency": "USD"}
        ```

    === "Go"

        Esta herramienta recupera el valor simulado de un precio de acción.

        ```go
        import (
            "google.golang.org/adk/agent"
            "google.golang.org/adk/agent/llmagent"
            "google.golang.org/adk/model/gemini"
            "google.golang.org/adk/runner"
            "google.golang.org/adk/session"
            "google.golang.org/adk/tool"
            "google.golang.org/adk/tool/functiontool"
            "google.golang.org/genai"
        )

        --8<-- "examples/go/snippets/tools/function-tools/func_tool.go"
        ```

        El valor de retorno de esta herramienta será una instancia de `getStockPriceResults`.

        ```json
        For input `{"symbol": "GOOG"}`: {"price":300.6,"symbol":"GOOG"}
        ```

    === "Java"

        Esta herramienta recupera el valor simulado de un precio de acción.

        ```java
        --8<-- "examples/java/snippets/src/main/java/tools/StockPriceAgent.java:full_code"
        ```

        El valor de retorno de esta herramienta será envuelto en un Map<String, Object>.

        ```json
        For input `GOOG`: {"symbol": "GOOG", "price": "1.0"}
        ```

### Mejores prácticas

Aunque tienes considerable flexibilidad al definir tu función, recuerda que la simplicidad mejora la usabilidad para el LLM. Considera estas pautas:

* **Menos parámetros es mejor:** Minimiza el número de parámetros para reducir la complejidad.
* **Tipos de datos simples:** Favorece tipos de datos primitivos como `str` e `int` sobre clases personalizadas siempre que sea posible.
* **Nombres significativos:** El nombre de la función y los nombres de los parámetros influyen significativamente en cómo el LLM interpreta y utiliza la herramienta. Elige nombres que reflejen claramente el propósito de la función y el significado de sus entradas. Evita nombres genéricos como `do_stuff()` o `beAgent()`.
* **Construir para ejecución paralela:** Mejora el rendimiento de las llamadas a funciones cuando se ejecutan múltiples herramientas construyendo para operación asíncrona. Para información sobre cómo habilitar la ejecución paralela para herramientas, consulta
[Aumentar el rendimiento de herramientas con ejecución paralela](/adk-docs/tools-custom/performance/).

## Herramientas de Función de Larga Ejecución {#long-run-tool}

Esta herramienta está diseñada para ayudarte a iniciar y gestionar tareas que se manejan fuera de la operación de tu flujo de trabajo de agente, y requieren una cantidad significativa de tiempo de procesamiento, sin bloquear la ejecución del agente. Esta herramienta es una subclase de `FunctionTool`.

Al usar un `LongRunningFunctionTool`, tu función puede iniciar la operación de larga ejecución y opcionalmente devolver un **resultado inicial**, como un id de operación de larga ejecución. Una vez que se invoca una herramienta de función de larga ejecución, el ejecutor del agente pausa la ejecución del agente y permite al cliente del agente decidir si continuar o esperar hasta que la operación de larga ejecución finalice. El cliente del agente puede consultar el progreso de la operación de larga ejecución y enviar de vuelta una respuesta intermedia o final. El agente puede entonces continuar con otras tareas. Un ejemplo es el escenario de humano en el bucle donde el agente necesita aprobación humana antes de proceder con una tarea.

!!! warning "Advertencia: Manejo de ejecución"
    Las Herramientas de Función de Larga Ejecución están diseñadas para ayudarte a iniciar y *gestionar* tareas
    de larga ejecución como parte de tu flujo de trabajo de agente, pero ***no realizar*** la tarea larga real.
    Para tareas que requieren un tiempo significativo para completarse, debes implementar un servidor
    separado para hacer la tarea.

!!! tip "Consejo: Ejecución paralela"
    Dependiendo del tipo de herramienta que estés construyendo, diseñar para operación asíncrona
    puede ser una mejor solución que crear una herramienta de larga ejecución. Para
    más información, consulta
    [Aumentar el rendimiento de herramientas con ejecución paralela](/adk-docs/tools-custom/performance/).

### Cómo funciona

En Python, envuelves una función con `LongRunningFunctionTool`. En Java, pasas un nombre de Método a `LongRunningFunctionTool.create()`. En TypeScript, instancias la clase `LongRunningFunctionTool`.


1. **Iniciación:** Cuando el LLM llama a la herramienta, tu función inicia la operación de larga ejecución.

2. **Actualizaciones iniciales:** Tu función debe opcionalmente devolver un resultado inicial (ej. el id de operación de larga ejecución). El framework ADK toma el resultado y lo envía de vuelta al LLM empaquetado dentro de un `FunctionResponse`. Esto permite al LLM informar al usuario (ej., estado, porcentaje completo, mensajes). Y luego la ejecución del agente termina/se pausa.

3. **Continuar o esperar:** Después de que cada ejecución del agente se complete. El cliente del agente puede consultar el progreso de la operación de larga ejecución y decidir si continuar la ejecución del agente con una respuesta intermedia (para actualizar el progreso) o esperar hasta que se recupere una respuesta final. El cliente del agente debe enviar la respuesta intermedia o final de vuelta al agente para la siguiente ejecución.

4. **Manejo del framework:** El framework ADK gestiona la ejecución. Envía la `FunctionResponse` intermedia o final enviada por el cliente del agente al LLM para generar un mensaje amigable para el usuario.

### Crear la herramienta

Define tu función de herramienta y envuélvela usando la clase `LongRunningFunctionTool`:

=== "Python"

    ```python
    --8<-- "examples/python/snippets/tools/function-tools/human_in_the_loop.py:define_long_running_function"
    ```

=== "TypeScript"

    ```typescript
    --8<-- "examples/typescript/snippets/tools/function-tools/long-running-function-tool-example.ts:define_long_running_function"
    ```

=== "Go"

    ```go
    import (
        "google.golang.org/adk/agent"
        "google.golang.org/adk/agent/llmagent"
        "google.golang.org/adk/model/gemini"
        "google.golang.org/adk/tool"
        "google.golang.org/adk/tool/functiontool"
        "google.golang.org/genai"
    )

    --8<-- "examples/go/snippets/tools/function-tools/long-running-tool/long_running_tool.go:create_long_running_tool"
    ```

=== "Java"

    ```java
    import com.google.adk.agents.LlmAgent;
    import com.google.adk.tools.LongRunningFunctionTool;
    import java.util.HashMap;
    import java.util.Map;

    public class ExampleLongRunningFunction {

      // Define your Long Running function.
      // Ask for approval for the reimbursement.
      public static Map<String, Object> askForApproval(String purpose, double amount) {
        // Simulate creating a ticket and sending a notification
        System.out.println(
            "Simulating ticket creation for purpose: " + purpose + ", amount: " + amount);

        // Send a notification to the approver with the link of the ticket
        Map<String, Object> result = new HashMap<>();
        result.put("status", "pending");
        result.put("approver", "Sean Zhou");
        result.put("purpose", purpose);
        result.put("amount", amount);
        result.put("ticket-id", "approval-ticket-1");
        return result;
      }

      public static void main(String[] args) throws NoSuchMethodException {
        // Pass the method to LongRunningFunctionTool.create
        LongRunningFunctionTool approveTool =
            LongRunningFunctionTool.create(ExampleLongRunningFunction.class, "askForApproval");

        // Include the tool in the agent
        LlmAgent approverAgent =
            LlmAgent.builder()
                // ...
                .tools(approveTool)
                .build();
      }
    }
    ```

### Actualizaciones de resultado intermedio/final

El cliente del agente recibió un evento con llamadas a funciones de larga ejecución y verifica el estado del ticket. Luego, el cliente del agente puede enviar la respuesta intermedia o final de vuelta para actualizar el progreso. El framework empaqueta este valor (incluso si es None) en el contenido del `FunctionResponse` enviado de vuelta al LLM.

!!! note "Nota: Respuesta de función de larga ejecución con funcionalidad de Resume"

    Si tu flujo de trabajo de agente ADK está configurado con la
    funcionalidad [Resume](/adk-docs/runtime/resume/), también debes incluir
    el parámetro de ID de Invocación (`invocation_id`) con la respuesta de función de larga
    ejecución. El ID de Invocación que proporciones debe ser la misma
    invocación que generó la solicitud de función de larga ejecución, de lo contrario
    el sistema inicia una nueva invocación con la respuesta. Si tu
    agente usa la funcionalidad Resume, considera incluir el ID de Invocación
    como parámetro con tu solicitud de función de larga ejecución, para que pueda ser
    incluido con la respuesta. Para más detalles sobre el uso de la funcionalidad Resume, consulta
    [Reanudar agentes detenidos](/adk-docs/runtime/resume/).

??? Tip "Aplica solo a ADK de Java"

    Al pasar `ToolContext` con Herramientas de Función, asegúrate de que una de las siguientes sea verdadera:

    * El Schema se pasa con el parámetro ToolContext en la firma de la función, como:
      ```
      @com.google.adk.tools.Annotations.Schema(name = "toolContext") ToolContext toolContext
      ```
    O

    * La siguiente bandera `-parameters` está configurada para el plugin del compilador mvn

    ```
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.14.0</version> <!-- or newer -->
                <configuration>
                    <compilerArgs>
                        <arg>-parameters</arg>
                    </compilerArgs>
                </configuration>
            </plugin>
        </plugins>
    </build>
    ```
    Esta restricción es temporal y será eliminada.


=== "Python"

    ```python
    --8<-- "examples/python/snippets/tools/function-tools/human_in_the_loop.py:call_reimbursement_tool"
    ```

=== "TypeScript"

    ```typescript
    --8<-- "examples/typescript/snippets/tools/function-tools/long-running-function-tool-example.ts"
    ```

=== "Go"

    El siguiente ejemplo demuestra un flujo de trabajo de múltiples turnos. Primero, el usuario le pide al agente que cree un ticket. El agente llama a la herramienta de larga ejecución y el cliente captura el ID de `FunctionCall`. El cliente luego simula que el trabajo asíncrono se completa enviando mensajes `FunctionResponse` subsecuentes de vuelta al agente para proporcionar el ID del ticket y el estado final.

    ```go
    --8<-- "examples/go/snippets/tools/function-tools/long-running-tool/long_running_tool.go:run_long_running_tool"
    ```

=== "Java"

    ```java
    --8<-- "examples/java/snippets/src/main/java/tools/LongRunningFunctionExample.java:full_code"
    ```


??? "Ejemplo completo de Python: Simulación de procesamiento de archivos"

    ```python
    --8<-- "examples/python/snippets/tools/function-tools/human_in_the_loop.py"
    ```

#### Aspectos clave de este ejemplo

* **`LongRunningFunctionTool`**: Envuelve el método/función proporcionado; el framework maneja el envío de actualizaciones producidas y el valor de retorno final como FunctionResponses secuenciales.

* **Instrucción del agente**: Dirige al LLM a usar la herramienta y entender el flujo de FunctionResponse entrante (progreso vs. finalización) para actualizaciones al usuario.

* **Retorno final**: La función devuelve el diccionario de resultado final, que se envía en el FunctionResponse conclusivo para indicar finalización.

## Agente como Herramienta {#agent-tool}

Esta poderosa característica te permite aprovechar las capacidades de otros agentes dentro de tu sistema llamándolos como herramientas. El Agente como Herramienta te permite invocar otro agente para realizar una tarea específica, efectivamente **delegando responsabilidad**. Esto es conceptualmente similar a crear una función de Python que llama a otro agente y usa la respuesta del agente como el valor de retorno de la función.

### Diferencia clave con los sub-agentes

Es importante distinguir un Agente como Herramienta de un Sub-Agente.

* **Agente como Herramienta:** Cuando el Agente A llama al Agente B como una herramienta (usando Agente como Herramienta), la respuesta del Agente B se **pasa de vuelta** al Agente A, que luego resume la respuesta y genera una respuesta al usuario. El Agente A mantiene el control y continúa manejando futuras entradas del usuario.

* **Sub-agente:** Cuando el Agente A llama al Agente B como un sub-agente, la responsabilidad de responder al usuario se **transfiere completamente** al Agente B. El Agente A queda efectivamente fuera del bucle. Todas las entradas subsecuentes del usuario serán respondidas por el Agente B.

### Uso

Para usar un agente como una herramienta, envuelve el agente con la clase AgentTool.

=== "Python"

    ```python
    tools=[AgentTool(agent=agent_b)]
    ```

=== "TypeScript"

    ```typescript
    tools: [new AgentTool({agent: agentB})]
    ```

=== "Go"

    ```go
    agenttool.New(agent, &agenttool.Config{...})
    ```

=== "Java"

    ```java
    AgentTool.create(agent)
    ```

### Personalización

La clase `AgentTool` proporciona los siguientes atributos para personalizar su comportamiento:

* **skip\_summarization: bool:** Si se establece en True, el framework **omitirá la sumarización basada en LLM** de la respuesta del agente herramienta. Esto puede ser útil cuando la respuesta de la herramienta ya está bien formateada y no requiere procesamiento adicional.

??? "Ejemplo"

    === "Python"

        ```python
        --8<-- "examples/python/snippets/tools/function-tools/summarizer.py"
        ```

    === "TypeScript"

        ```typescript
        --8<-- "examples/typescript/snippets/tools/function-tools/agent-as-a-tool-example.ts"
        ```

    === "Go"

        ```go
        import (
            "google.golang.org/adk/agent"
            "google.golang.org/adk/agent/llmagent"
            "google.golang.org/adk/model/gemini"
            "google.golang.org/adk/tool"
            "google.golang.org/adk/tool/agenttool"
            "google.golang.org/genai"
        )

        --8<-- "examples/go/snippets/tools/function-tools/func_tool.go:agent_tool_example"
        ```

    === "Java"

        ```java
        --8<-- "examples/java/snippets/src/main/java/tools/AgentToolCustomization.java:full_code"
        ```

### Cómo funciona

1. Cuando el `main_agent` recibe el texto largo, su instrucción le dice que use la herramienta 'summarize' para textos largos.
2. El framework reconoce 'summarize' como un `AgentTool` que envuelve al `summary_agent`.
3. Detrás de escena, el `main_agent` llamará al `summary_agent` con el texto largo como entrada.
4. El `summary_agent` procesará el texto de acuerdo a su instrucción y generará un resumen.
5. **La respuesta del `summary_agent` se pasa luego de vuelta al `main_agent`.**
6. El `main_agent` puede entonces tomar el resumen y formular su respuesta final al usuario (ej., "Aquí hay un resumen del texto: ...")