# Seguridad y Protección para Agentes de IA

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span><span class="lst-typescript">TypeScript</span><span class="lst-go">Go</span><span class="lst-java">Java</span>
</div>

A medida que los agentes de IA crecen en capacidad, garantizar que operen de manera segura, protegida y alineada con los valores de tu marca es primordial. Los agentes sin control pueden presentar riesgos, incluyendo la ejecución de acciones desalineadas o dañinas, como la exfiltración de datos, y la generación de contenido inapropiado que puede impactar la reputación de tu marca. **Las fuentes de riesgo incluyen instrucciones vagas, alucinaciones del modelo, jailbreaks e inyecciones de prompts por parte de usuarios adversarios, e inyecciones indirectas de prompts mediante el uso de herramientas.**

[Google Cloud Vertex AI](https://cloud.google.com/vertex-ai/generative-ai/docs/overview) proporciona un enfoque de múltiples capas para mitigar estos riesgos, permitiéndote construir agentes potentes *y* confiables. Ofrece varios mecanismos para establecer límites estrictos, asegurando que los agentes solo realicen acciones que hayas permitido explícitamente:

1. **Identidad y Autorización**: Controla quién **actúa como** el agente definiendo la autenticación del agente y del usuario.
2. **Barreras de protección para filtrar entradas y salidas:** Controla tu modelo y las llamadas a herramientas con precisión.

    * *Barreras de protección dentro de las herramientas:* Diseña herramientas de manera defensiva, usando contexto de herramienta establecido por el desarrollador para aplicar políticas (p. ej., permitir consultas solo en tablas específicas).
    * *Funcionalidades de seguridad integradas de Gemini:* Si usas modelos Gemini, benefíciate de los filtros de contenido para bloquear salidas dañinas e Instrucciones del sistema para guiar el comportamiento del modelo y las directrices de seguridad
    * *Callbacks y Plugins:* Valida las llamadas al modelo y a las herramientas antes o después de la ejecución, verificando los parámetros contra el estado del agente o políticas externas.
    * *Usar Gemini como barrera de protección de seguridad:* Implementa una capa de seguridad adicional usando un modelo económico y rápido (como Gemini Flash Lite) configurado mediante callbacks para filtrar entradas y salidas.

3. **Ejecución de código aislada:** Previene que el código generado por el modelo cause problemas de seguridad aislando el entorno
4. **Evaluación y rastreo**: Usa herramientas de evaluación para evaluar la calidad, relevancia y corrección de la salida final del agente. Usa el rastreo para obtener visibilidad de las acciones del agente para analizar los pasos que un agente toma para llegar a una solución, incluyendo su elección de herramientas, estrategias y la eficiencia de su enfoque.
5. **Controles de red y VPC-SC:** Confina la actividad del agente dentro de perímetros seguros (como VPC Service Controls) para prevenir la exfiltración de datos y limitar el radio de impacto potencial.

## Riesgos de Seguridad y Protección

Antes de implementar medidas de seguridad, realiza una evaluación exhaustiva de riesgos específica para las capacidades de tu agente, dominio y contexto de implementación.

***Las fuentes*** **de riesgo** incluyen:

* Instrucciones ambiguas del agente
* Intentos de inyección de prompts y jailbreak por parte de usuarios adversarios
* Inyecciones indirectas de prompts mediante el uso de herramientas

**Las categorías de riesgo** incluyen:

* **Desalineación y corrupción de objetivos**
    * Perseguir objetivos no intencionados o proxy que llevan a resultados dañinos ("reward hacking")
    * Malinterpretar instrucciones complejas o ambiguas
* **Generación de contenido dañino, incluyendo seguridad de marca**
    * Generar contenido tóxico, odioso, sesgado, sexualmente explícito, discriminatorio o ilegal
    * Riesgos de seguridad de marca como usar lenguaje que va en contra de los valores de la marca o conversaciones fuera de tema
* **Acciones inseguras**
    * Ejecutar comandos que dañen sistemas
    * Realizar compras no autorizadas o transacciones financieras.
    * Filtrar datos personales sensibles (PII)
    * Exfiltración de datos

## Mejores prácticas

### Identidad y Autorización

La identidad que una *herramienta* usa para realizar acciones en sistemas externos es una consideración de diseño crucial desde una perspectiva de seguridad. Diferentes herramientas en el mismo agente pueden ser configuradas con diferentes estrategias, por lo que se necesita cuidado al hablar sobre las configuraciones del agente.

#### Autenticación del Agente

La **herramienta interactúa con sistemas externos usando la propia identidad del agente** (p. ej., una cuenta de servicio). La identidad del agente debe ser explícitamente autorizada en las políticas de acceso del sistema externo, como agregar una cuenta de servicio del agente a la política IAM de una base de datos para acceso de lectura. Tales políticas restringen al agente a solo realizar acciones que el desarrollador intentó como posibles: al dar permisos de solo lectura a un recurso, sin importar lo que el modelo decida, la herramienta será prohibida de realizar acciones de escritura.

Este enfoque es simple de implementar, y es **apropiado para agentes donde todos los usuarios comparten el mismo nivel de acceso.** Si no todos los usuarios tienen el mismo nivel de acceso, tal enfoque por sí solo no proporciona suficiente protección y debe ser complementado con otras técnicas a continuación. En la implementación de herramientas, asegúrate de que se creen registros para mantener la atribución de acciones a los usuarios, ya que todas las acciones de los agentes aparecerán como provenientes del agente.

#### Autenticación del Usuario

La herramienta interactúa con un sistema externo usando la **identidad del "usuario controlador"** (p. ej., el humano interactuando con el frontend en una aplicación web). En ADK, esto se implementa típicamente usando OAuth: el agente interactúa con el frontend para adquirir un token OAuth, y luego la herramienta usa el token al realizar acciones externas: el sistema externo autoriza la acción si el usuario controlador está autorizado para realizarla por sí mismo.

La autenticación del usuario tiene la ventaja de que los agentes solo realizan acciones que el usuario podría haber realizado por sí mismo. Esto reduce enormemente el riesgo de que un usuario malicioso pueda abusar del agente para obtener acceso a datos adicionales. Sin embargo, las implementaciones más comunes de delegación tienen un conjunto fijo de permisos para delegar (es decir, ámbitos de OAuth). A menudo, tales ámbitos son más amplios que el acceso que el agente realmente requiere, y se requieren las técnicas a continuación para restringir aún más las acciones del agente.

### Barreras de protección para filtrar entradas y salidas

#### Barreras de protección dentro de las herramientas

Las herramientas pueden ser diseñadas con la seguridad en mente: podemos crear herramientas que expongan las acciones que queremos que el modelo tome y nada más. Al limitar el rango de acciones que proporcionamos a los agentes, podemos eliminar determinísticamente clases de acciones deshonestas que nunca queremos que el agente tome.

Las barreras de protección dentro de las herramientas es un enfoque para crear herramientas comunes y reutilizables que expongan controles determinísticos que pueden ser usados por los desarrolladores para establecer límites en cada instanciación de herramienta.

Este enfoque se basa en el hecho de que las herramientas reciben dos tipos de entrada: argumentos, que son establecidos por el modelo, y [**`Tool Context`**](../tools-custom/index.md#tool-context), que puede ser establecido determinísticamente por el desarrollador del agente. Podemos confiar en la información establecida determinísticamente para validar que el modelo está comportándose como se espera.

Por ejemplo, una herramienta de consulta puede ser diseñada para esperar que una política sea leída desde el Tool Context.

=== "Python"

    ```py
    # Ejemplo conceptual: Establecer datos de política destinados al contexto de herramienta
    # En una aplicación ADK real, esto podría establecerse en InvocationContext.session.state
    # o pasarse durante la inicialización de la herramienta, luego recuperarse a través de ToolContext.

    policy = {} # Asumiendo que policy es un diccionario
    policy['select_only'] = True
    policy['tables'] = ['mytable1', 'mytable2']

    # Conceptual: Almacenar la política donde la herramienta pueda acceder a ella a través de ToolContext más tarde.
    # Esta línea específica podría verse diferente en la práctica.
    # Por ejemplo, almacenar en el estado de sesión:
    invocation_context.session.state["query_tool_policy"] = policy

    # O tal vez pasar durante la inicialización de la herramienta:
    query_tool = QueryTool(policy=policy)
    # Para este ejemplo, asumiremos que se almacena en algún lugar accesible.
    ```

=== "TypeScript"

    ```typescript
    // Ejemplo conceptual: Establecer datos de política destinados al contexto de herramienta
    // En una aplicación ADK real, esto podría establecerse en InvocationContext.session.state
    // o pasarse durante la inicialización de la herramienta, luego recuperarse a través de ToolContext.

    const policy: {[key: string]: any} = {}; // Asumiendo que policy es un objeto
    policy['select_only'] = true;
    policy['tables'] = ['mytable1', 'mytable2'];

    // Conceptual: Almacenar la política donde la herramienta pueda acceder a ella a través de ToolContext más tarde.
    // Esta línea específica podría verse diferente en la práctica.
    // Por ejemplo, almacenar en el estado de sesión:
    invocationContext.session.state["query_tool_policy"] = policy;

    // O tal vez pasar durante la inicialización de la herramienta:
    const queryTool = new QueryTool({policy: policy});
    // Para este ejemplo, asumiremos que se almacena en algún lugar accesible.
    ```

=== "Go"

    ```go
    // Ejemplo conceptual: Establecer datos de política destinados al contexto de herramienta
    // En una aplicación ADK real, esto podría establecerse usando el servicio de estado de sesión.
    // `ctx` es un `agent.Context` disponible en callbacks o agentes personalizados.

    policy := map[string]interface{}{
    	"select_only": true,
    	"tables":      []string{"mytable1", "mytable2"},
    }

    // Conceptual: Almacenar la política donde la herramienta pueda acceder a ella a través de ToolContext más tarde.
    // Esta línea específica podría verse diferente en la práctica.
    // Por ejemplo, almacenar en el estado de sesión:
    if err := ctx.Session().State().Set("query_tool_policy", policy); err != nil {
        // Manejar error, p. ej., registrarlo.
    }

    // O tal vez pasar durante la inicialización de la herramienta:
    // queryTool := NewQueryTool(policy)
    // Para este ejemplo, asumiremos que se almacena en algún lugar accesible.
    ```

=== "Java"

    ```java
    // Ejemplo conceptual: Establecer datos de política destinados al contexto de herramienta
    // En una aplicación ADK real, esto podría establecerse en InvocationContext.session.state
    // o pasarse durante la inicialización de la herramienta, luego recuperarse a través de ToolContext.

    policy = new HashMap<String, Object>(); // Asumiendo que policy es un Map
    policy.put("select_only", true);
    policy.put("tables", new ArrayList<>("mytable1", "mytable2"));

    // Conceptual: Almacenar la política donde la herramienta pueda acceder a ella a través de ToolContext más tarde.
    // Esta línea específica podría verse diferente en la práctica.
    // Por ejemplo, almacenar en el estado de sesión:
    invocationContext.session().state().put("query_tool_policy", policy);

    // O tal vez pasar durante la inicialización de la herramienta:
    query_tool = QueryTool(policy);
    // Para este ejemplo, asumiremos que se almacena en algún lugar accesible.
    ```

Durante la ejecución de la herramienta, [**`Tool Context`**](../tools-custom/index.md#tool-context) será pasado a la herramienta:

=== "Python"

    ```py
    def query(query: str, tool_context: ToolContext) -> str | dict:
      # Asumir que 'policy' se recupera del contexto, p. ej., a través del estado de sesión:
      # policy = tool_context.invocation_context.session.state.get('query_tool_policy', {})

      # --- Aplicación de Política de Marcador de Posición ---
      policy = tool_context.invocation_context.session.state.get('query_tool_policy', {}) # Ejemplo de recuperación
      actual_tables = explainQuery(query) # Llamada hipotética a función

      if not set(actual_tables).issubset(set(policy.get('tables', []))):
        # Devolver un mensaje de error para el modelo
        allowed = ", ".join(policy.get('tables', ['(None defined)']))
        return f"Error: Query targets unauthorized tables. Allowed: {allowed}"

      if policy.get('select_only', False):
           if not query.strip().upper().startswith("SELECT"):
               return "Error: Policy restricts queries to SELECT statements only."
      # --- Fin de Aplicación de Política ---

      print(f"Executing validated query (hypothetical): {query}")
      return {"status": "success", "results": [...]} # Ejemplo de retorno exitoso
    ```

=== "TypeScript"

    ```typescript
    function query(query: string, toolContext: ToolContext): string | object {
        // Asumir que 'policy' se recupera del contexto, p. ej., a través del estado de sesión:
        const policy = toolContext.state.get('query_tool_policy', {}) as {[key: string]: any};

        // --- Aplicación de Política de Marcador de Posición ---
        const actual_tables = explainQuery(query); // Llamada hipotética a función

        const policyTables = new Set(policy['tables'] || []);
        const isSubset = actual_tables.every(table => policyTables.has(table));

        if (!isSubset) {
            // Devolver un mensaje de error para el modelo
            const allowed = (policy['tables'] || ['(None defined)']).join(', ');
            return `Error: Query targets unauthorized tables. Allowed: ${allowed}`;
        }

        if (policy['select_only']) {
            if (!query.trim().toUpperCase().startsWith("SELECT")) {
                return "Error: Policy restricts queries to SELECT statements only.";
            }
        }
        // --- Fin de Aplicación de Política ---

        console.log(`Executing validated query (hypothetical): ${query}`);
        return { "status": "success", "results": [] }; // Ejemplo de retorno exitoso
    }
    ```

=== "Go"

    ```go
    import (
    	"fmt"
    	"strings"

    	"google.golang.org/adk/tool"
    )

    func query(query string, toolContext *tool.Context) (any, error) {
    	// Asumir que 'policy' se recupera del contexto, p. ej., a través del estado de sesión:
    	policyAny, err := toolContext.State().Get("query_tool_policy")
    	if err != nil {
    		return nil, fmt.Errorf("could not retrieve policy: %w", err)
    	}    	policy, _ := policyAny.(map[string]interface{})
    	actualTables := explainQuery(query) // Llamada hipotética a función

    	// --- Aplicación de Política de Marcador de Posición ---
    	if tables, ok := policy["tables"].([]string); ok {
    		if !isSubset(actualTables, tables) {
    			// Devolver un error para señalar falla
    			allowed := strings.Join(tables, ", ")
    			if allowed == "" {
    				allowed = "(None defined)"
    			}
    			return nil, fmt.Errorf("query targets unauthorized tables. Allowed: %s", allowed)
    		}
    	}

    	if selectOnly, _ := policy["select_only"].(bool); selectOnly {
    		if !strings.HasPrefix(strings.ToUpper(strings.TrimSpace(query)), "SELECT") {
    			return nil, fmt.Errorf("policy restricts queries to SELECT statements only")
    		}
    	}
    	// --- Fin de Aplicación de Política ---

    	fmt.Printf("Executing validated query (hypothetical): %s\n", query)
    	return map[string]interface{}{"status": "success", "results": []string{"..."}}, nil
    }

    // Función auxiliar para verificar si a es un subconjunto de b
    func isSubset(a, b []string) bool {
    	set := make(map[string]bool)
    	for _, item := range b {
    		set[item] = true
    	}
    	for _, item := range a {
    		if _, found := set[item]; !found {
    			return false
    		}
    	}
    	return true
    }
    ```

=== "Java"

    ```java

    import com.google.adk.tools.ToolContext;
    import java.util.*;

    class ToolContextQuery {

      public Object query(String query, ToolContext toolContext) {

        // Asumir que 'policy' se recupera del contexto, p. ej., a través del estado de sesión:
        Map<String, Object> queryToolPolicy =
            toolContext.invocationContext.session().state().getOrDefault("query_tool_policy", null);
        List<String> actualTables = explainQuery(query);

        // --- Aplicación de Política de Marcador de Posición ---
        if (!queryToolPolicy.get("tables").containsAll(actualTables)) {
          List<String> allowedPolicyTables =
              (List<String>) queryToolPolicy.getOrDefault("tables", new ArrayList<String>());

          String allowedTablesString =
              allowedPolicyTables.isEmpty() ? "(None defined)" : String.join(", ", allowedPolicyTables);

          return String.format(
              "Error: Query targets unauthorized tables. Allowed: %s", allowedTablesString);
        }

        if (!queryToolPolicy.get("select_only")) {
          if (!query.trim().toUpperCase().startswith("SELECT")) {
            return "Error: Policy restricts queries to SELECT statements only.";
          }
        }
        // --- Fin de Aplicación de Política ---

        System.out.printf("Executing validated query (hypothetical) %s:", query);
        Map<String, Object> successResult = new HashMap<>();
        successResult.put("status", "success");
        successResult.put("results", Arrays.asList("result_item1", "result_item2"));
        return successResult;
      }
    }
    ```

#### Funcionalidades de seguridad integradas de Gemini

Los modelos Gemini vienen con mecanismos de seguridad incorporados que pueden aprovecharse para mejorar el contenido y la seguridad de marca.

* **Filtros de seguridad de contenido**: Los [filtros de contenido](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/configure-safety-attributes) pueden ayudar a bloquear la salida de contenido dañino. Funcionan independientemente de los modelos Gemini como parte de una defensa en capas contra actores de amenazas que intentan hacer jailbreak al modelo. Los modelos Gemini en Vertex AI usan dos tipos de filtros de contenido:
* **Filtros de seguridad no configurables** bloquean automáticamente salidas que contienen contenido prohibido, como material de abuso sexual infantil (CSAM) e información personalmente identificable (PII).
* **Filtros de contenido configurables** te permiten definir umbrales de bloqueo en cuatro categorías de daño (discurso de odio, acoso, sexualmente explícito y contenido peligroso), basados en puntuaciones de probabilidad y gravedad. Estos filtros están desactivados por defecto pero puedes configurarlos según tus necesidades.
* **Instrucciones del sistema para seguridad**: Las [instrucciones del sistema](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/safety-system-instructions) para modelos Gemini en Vertex AI proporcionan orientación directa al modelo sobre cómo comportarse y qué tipo de contenido generar. Al proporcionar instrucciones específicas, puedes dirigir proactivamente al modelo lejos de generar contenido indeseable para satisfacer las necesidades únicas de tu organización. Puedes elaborar instrucciones del sistema para definir directrices de seguridad de contenido, como temas prohibidos y sensibles, y lenguaje de descargo de responsabilidad, así como directrices de seguridad de marca para garantizar que las salidas del modelo se alineen con la voz, tono, valores y audiencia objetivo de tu marca.

Si bien estas medidas son robustas contra la seguridad de contenido, necesitas verificaciones adicionales para reducir la desalineación del agente, las acciones inseguras y los riesgos de seguridad de marca.

#### Callbacks y Plugins para Barreras de Protección de Seguridad

Los callbacks proporcionan un método simple y específico del agente para agregar pre-validación a las E/S de herramientas y modelos, mientras que los plugins ofrecen una solución reutilizable para implementar políticas de seguridad generales en múltiples agentes.

Cuando las modificaciones a las herramientas para agregar barreras de protección no son posibles, la función [**`Before Tool Callback`**](../callbacks/types-of-callbacks.md#before-tool-callback) puede usarse para agregar pre-validación de llamadas. El callback tiene acceso al estado del agente, la herramienta solicitada y los parámetros. Este enfoque es muy general y puede incluso crearse para crear una biblioteca común de políticas de herramientas reutilizables. Sin embargo, podría no ser aplicable para todas las herramientas si la información para aplicar las barreras de protección no es directamente visible en los parámetros.

=== "Python"

    ```py
    # Función callback hipotética
    def validate_tool_params(
        callback_context: CallbackContext, # Tipo de contexto correcto
        tool: BaseTool,
        args: Dict[str, Any],
        tool_context: ToolContext
        ) -> Optional[Dict]: # Tipo de retorno correcto para before_tool_callback

      print(f"Callback triggered for tool: {tool.name}, args: {args}")

      # Ejemplo de validación: Verificar si un ID de usuario requerido del estado coincide con un arg
      expected_user_id = callback_context.state.get("session_user_id")
      actual_user_id_in_args = args.get("user_id_param") # Asumiendo que la herramienta toma 'user_id_param'

      if actual_user_id_in_args != expected_user_id:
          print("Validation Failed: User ID mismatch!")
          # Devolver un diccionario para prevenir la ejecución de la herramienta y proporcionar retroalimentación
          return {"error": f"Tool call blocked: User ID mismatch."}

      # Devolver None para permitir que la llamada a la herramienta continúe si la validación pasa
      print("Callback validation passed.")
      return None

    # Configuración hipotética del Agente
    root_agent = LlmAgent( # Usar tipo de agente específico
        model='gemini-2.0-flash',
        name='root_agent',
        instruction="...",
        before_tool_callback=validate_tool_params, # Asignar el callback
        tools = [
          # ... lista de funciones de herramienta o instancias de Tool ...
          # p. ej., query_tool_instance
        ]
    )
    ```

=== "TypeScript"

    ```typescript
    // Función callback hipotética
    function validateToolParams(
        {tool, args, context}: {
            tool: BaseTool,
            args: {[key: string]: any},
            context: ToolContext
        }
    ): {[key: string]: any} | undefined {
        console.log(`Callback triggered for tool: ${tool.name}, args: ${JSON.stringify(args)}`);

        // Ejemplo de validación: Verificar si un ID de usuario requerido del estado coincide con un arg
        const expectedUserId = context.state.get("session_user_id");
        const actualUserIdInArgs = args["user_id_param"]; // Asumiendo que la herramienta toma 'user_id_param'

        if (actualUserIdInArgs !== expectedUserId) {
            console.log("Validation Failed: User ID mismatch!");
            // Devolver un diccionario para prevenir la ejecución de la herramienta y proporcionar retroalimentación
            return {"error": `Tool call blocked: User ID mismatch.`};
        }

        // Devolver undefined para permitir que la llamada a la herramienta continúe si la validación pasa
        console.log("Callback validation passed.");
        return undefined;
    }

    // Configuración hipotética del Agente
    const rootAgent = new LlmAgent({
        model: 'gemini-2.5-flash',
        name: 'root_agent',
        instruction: "...",
        beforeToolCallback: validateToolParams, // Asignar el callback
        tools: [
          // ... lista de funciones de herramienta o instancias de Tool ...
          // p. ej., queryToolInstance
        ]
    });
    ```

=== "Go"

    ```go
    import (
    	"fmt"
    	"reflect"

    	"google.golang.org/adk/agent/llmagent"
    	"google.golang.org/adk/tool"
    )

    // Función callback hipotética
    func validateToolParams(
    	ctx tool.Context,
    	t tool.Tool,
    	args map[string]any,
    ) (map[string]any, error) {
    	fmt.Printf("Callback triggered for tool: %s, args: %v\n", t.Name(), args)

    	// Ejemplo de validación: Verificar si un ID de usuario requerido del estado coincide con un arg
    	expectedUserID, err := ctx.State().Get("session_user_id")
    	if err != nil {
    		// Este es un fallo inesperado, devolver un error.
    		return nil, fmt.Errorf("internal error: session_user_id not found in state: %w", err)
    	}
    	    	expectedUserID, ok := expectedUserIDVal.(string)
    	if !ok {
    		return nil, fmt.Errorf("internal error: session_user_id in state is not a string, got %T", expectedUserIDVal)
    	}

    	actualUserIDInArgs, exists := args["user_id_param"]
    	if !exists {
    		// Manejar caso donde user_id_param no está en args
    		fmt.Println("Validation Failed: user_id_param missing from arguments!")
    		return map[string]any{"error": "Tool call blocked: user_id_param missing from arguments."}, nil
    	}

    	actualUserID, ok := actualUserIDInArgs.(string)
    	if !ok {
    		// Manejar caso donde user_id_param no es una cadena
    		fmt.Println("Validation Failed: user_id_param is not a string!")
    		return map[string]any{"error": "Tool call blocked: user_id_param is not a string."}, nil
    	}

    	if actualUserID != expectedUserID {
    		fmt.Println("Validation Failed: User ID mismatch!")
    		// Devolver un mapa para prevenir la ejecución de la herramienta y proporcionar retroalimentación al modelo.
    		// Esto no es un error de Go, sino un mensaje para el agente.
    		return map[string]any{"error": "Tool call blocked: User ID mismatch."}, nil
    	}
    	// Devolver nil, nil para permitir que la llamada a la herramienta continúe si la validación pasa
    	fmt.Println("Callback validation passed.")
    	return nil, nil
    }

    // Configuración hipotética del Agente
    // rootAgent, err := llmagent.New(llmagent.Config{
    // 	Model: "gemini-2.0-flash",
    // 	Name: "root_agent",
    // 	Instruction: "...",
    // 	BeforeToolCallbacks: []llmagent.BeforeToolCallback{validateToolParams},
    // 	Tools: []tool.Tool{queryToolInstance},
    // })
    ```

=== "Java"

    ```java
    // Función callback hipotética
    public Optional<Map<String, Object>> validateToolParams(
      CallbackContext callbackContext,
      Tool baseTool,
      Map<String, Object> input,
      ToolContext toolContext) {

    System.out.printf("Callback triggered for tool: %s, Args: %s", baseTool.name(), input);

    // Ejemplo de validación: Verificar si un ID de usuario requerido del estado coincide con un parámetro de entrada
    Object expectedUserId = callbackContext.state().get("session_user_id");
    Object actualUserIdInput = input.get("user_id_param"); // Asumiendo que la herramienta toma 'user_id_param'

    if (!actualUserIdInput.equals(expectedUserId)) {
      System.out.println("Validation Failed: User ID mismatch!");
      // Devolver para prevenir la ejecución de la herramienta y proporcionar retroalimentación
      return Optional.of(Map.of("error", "Tool call blocked: User ID mismatch."));
    }

    // Devolver para permitir que la llamada a la herramienta continúe si la validación pasa
    System.out.println("Callback validation passed.");
    return Optional.empty();
    }

    // Configuración hipotética del Agente
    public void runAgent() {
    LlmAgent agent =
        LlmAgent.builder()
            .model("gemini-2.0-flash")
            .name("AgentWithBeforeToolCallback")
            .instruction("...")
            .beforeToolCallback(this::validateToolParams) // Asignar el callback
            .tools(anyToolToUse) // Definir la herramienta a usar
            .build();
    }
    ```

Sin embargo, al agregar barreras de protección de seguridad a tus aplicaciones de agentes, los plugins son el enfoque recomendado para implementar políticas que no son específicas de un solo agente. Los plugins están diseñados para ser autocontenidos y modulares, permitiéndote crear plugins individuales para políticas de seguridad específicas, y aplicarlos globalmente a nivel del runner. Esto significa que un plugin de seguridad puede ser configurado una vez y aplicado a cada agente que usa el runner, asegurando barreras de protección de seguridad consistentes en toda tu aplicación sin código repetitivo.

Algunos ejemplos incluyen:

* **Plugin Gemini como Juez**: Este plugin usa Gemini Flash Lite para evaluar las entradas del usuario, las entradas y salidas de herramientas, y la respuesta del agente para detectar inyección de prompts, jailbreak y adecuación. El plugin configura Gemini para actuar como un filtro de seguridad para mitigar contra la seguridad de contenido, seguridad de marca y desalineación del agente. El plugin está configurado para pasar la entrada del usuario, las entradas y salidas de herramientas, y la salida del modelo a Gemini Flash Lite, quien decide si la entrada al agente es segura o no. Si Gemini decide que la entrada no es segura, el agente devuelve una respuesta predeterminada: "Sorry I cannot help with that. Can I help you with something else?".

* **Plugin Model Armor**: Un plugin que consulta la API de model armor para verificar posibles violaciones de seguridad de contenido en puntos especificados de la ejecución del agente. Similar al plugin _Gemini como Juez_, si Model Armor encuentra coincidencias de contenido dañino, devuelve una respuesta predeterminada al usuario.

* **Plugin de Redacción de PII**: Un plugin especializado con diseño para el [Before Tool Callback](/adk-docs/plugins/#tool-callbacks) y creado específicamente para redactar información personalmente identificable antes de que sea procesada por una herramienta o enviada a un servicio externo.

### Ejecución de código aislada

La ejecución de código es una herramienta especial que tiene implicaciones de seguridad extra: el aislamiento debe usarse para prevenir que el código generado por el modelo comprometa el entorno local, potencialmente creando problemas de seguridad.

Google y el ADK proporcionan varias opciones para la ejecución segura de código. La [función de ejecución de código de la API Vertex Gemini Enterprise](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/code-execution-api) permite a los agentes aprovechar la ejecución de código aislada del lado del servidor habilitando la herramienta tool_execution. Para código que realiza análisis de datos, puedes usar la herramienta [Code Executor](/adk-docs/tools/gemini-api/code-execution/) en ADK para llamar a la [Extensión de Intérprete de Código de Vertex](https://cloud.google.com/vertex-ai/generative-ai/docs/extensions/code-interpreter).

Si ninguna de estas opciones satisface tus requisitos, puedes construir tu propio ejecutor de código usando los bloques de construcción proporcionados por el ADK. Recomendamos crear entornos de ejecución que sean herméticos: sin conexiones de red y llamadas API permitidas para evitar la exfiltración de datos sin control; y limpieza completa de datos entre ejecuciones para no crear preocupaciones de exfiltración entre usuarios.

### Evaluaciones

Consulta [Evaluar Agentes](../evaluate/index.md).

### Perímetros VPC-SC y Controles de Red

Si estás ejecutando tu agente dentro de un perímetro VPC-SC, eso garantizará que todas las llamadas API solo manipularán recursos dentro del perímetro, reduciendo la posibilidad de exfiltración de datos.

Sin embargo, la identidad y los perímetros solo proporcionan controles gruesos alrededor de las acciones del agente. Las barreras de protección de uso de herramientas mitigan tales limitaciones, y dan más poder a los desarrolladores de agentes para controlar finamente qué acciones permitir.

### Otros riesgos de seguridad

#### Siempre escapar el contenido generado por el modelo en las IU

Se debe tener cuidado cuando la salida del agente se visualiza en un navegador: si el contenido HTML o JS no se escapa adecuadamente en la IU, el texto devuelto por el modelo podría ejecutarse, llevando a la exfiltración de datos. Por ejemplo, una inyección indirecta de prompt puede engañar a un modelo para incluir una etiqueta img engañando al navegador para enviar el contenido de la sesión a un sitio de terceros; o construir URLs que, si se hacen clic, envíen datos a sitios externos. El escape adecuado de tal contenido debe asegurar que el texto generado por el modelo no sea interpretado como código por los navegadores.