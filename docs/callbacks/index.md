# Callbacks: Observar, Personalizar y Controlar el Comportamiento del Agente

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Los callbacks son una característica fundamental de ADK, proporcionando un mecanismo poderoso para conectarse al proceso de ejecución de un agente. Te permiten observar, personalizar e incluso controlar el comportamiento del agente en puntos específicos predefinidos sin modificar el código del framework principal de ADK.

**¿Qué son?** En esencia, los callbacks son funciones estándar que tú defines. Luego asocias estas funciones con un agente cuando lo creas. El framework de ADK llama automáticamente a tus funciones en etapas clave, permitiéndote observar o intervenir. Piénsalo como puntos de control durante el proceso del agente:

* **Antes de que el agente comience su trabajo principal en una solicitud, y después de que termine:** Cuando le pides a un agente que haga algo (por ejemplo, responder una pregunta), ejecuta su lógica interna para determinar la respuesta.
  * El callback `Before Agent` se ejecuta *justo antes* de que comience este trabajo principal para esa solicitud específica.
  * El callback `After Agent` se ejecuta *justo después* de que el agente haya terminado todos sus pasos para esa solicitud y haya preparado el resultado final, pero justo antes de que se devuelva el resultado.
  * Este "trabajo principal" abarca el proceso *completo* del agente para manejar esa única solicitud. Esto puede involucrar decidir llamar a un LLM, realmente llamar al LLM, decidir usar una herramienta, usar la herramienta, procesar los resultados y finalmente armar la respuesta. Estos callbacks esencialmente envuelven toda la secuencia desde recibir la entrada hasta producir la salida final para esa interacción.
* **Antes de enviar una solicitud a, o después de recibir una respuesta del, Modelo de Lenguaje Grande (LLM):** Estos callbacks (`Before Model`, `After Model`) te permiten inspeccionar o modificar los datos que van hacia y vienen del LLM específicamente.
* **Antes de ejecutar una herramienta (como una función de Python u otro agente) o después de que termine:** De manera similar, los callbacks `Before Tool` y `After Tool` te dan puntos de control específicamente alrededor de la ejecución de herramientas invocadas por el agente.


![intro_components.png](../assets/callback_flow.png)

**¿Por qué usarlos?** Los callbacks desbloquean flexibilidad significativa y habilitan capacidades avanzadas del agente:

* **Observar y Depurar:** Registrar información detallada en pasos críticos para monitoreo y resolución de problemas.
* **Personalizar y Controlar:** Modificar datos que fluyen a través del agente (como solicitudes LLM o resultados de herramientas) o incluso omitir ciertos pasos completamente basándose en tu lógica.
* **Implementar Barreras de Seguridad:** Aplicar reglas de seguridad, validar entradas/salidas o prevenir operaciones no permitidas.
* **Gestionar Estado:** Leer o actualizar dinámicamente el estado de sesión del agente durante la ejecución.
* **Integrar y Mejorar:** Activar acciones externas (llamadas API, notificaciones) o agregar características como caché.

!!! tip
    Al implementar barreras de seguridad y políticas, usa Plugins de ADK para
    mejor modularidad y flexibilidad que los Callbacks. Para más detalles, consulta
    [Callbacks and Plugins for Security Guardrails](/safety/#callbacks-and-plugins-for-security-guardrails).

**Cómo se agregan:**

??? "Code"
    === "Python"

        ```python
        --8<-- "examples/python/snippets/callbacks/callback_basic.py:callback_basic"
        ```

    === "Typescript"

        ```typescript
        --8<-- "examples/typescript/snippets/callbacks/callback_basic.ts:callback_basic"
        ```
    
    === "Go"

        ```go
        --8<-- "examples/go/snippets/callbacks/main.go:imports"


        --8<-- "examples/go/snippets/callbacks/main.go:callback_basic"
        ```

    === "Java"

        ```java
        --8<-- "examples/java/snippets/src/main/java/callbacks/AgentWithBeforeModelCallback.java:init"
        ```

## El Mecanismo de Callback: Intercepción y Control

Cuando el framework de ADK encuentra un punto donde un callback puede ejecutarse (por ejemplo, justo antes de llamar al LLM), verifica si proporcionaste una función de callback correspondiente para ese agente. Si lo hiciste, el framework ejecuta tu función.

**El Contexto es Clave:** Tu función de callback no se llama de forma aislada. El framework proporciona **objetos de contexto** especiales (`CallbackContext` o `ToolContext`) como argumentos. Estos objetos contienen información vital sobre el estado actual de la ejecución del agente, incluyendo los detalles de invocación, el estado de sesión y potencialmente referencias a servicios como artifacts o memoria. Usas estos objetos de contexto para entender la situación e interactuar con el framework. (Consulta la sección dedicada "Objetos de Contexto" para detalles completos).

**Controlando el Flujo (El Mecanismo Principal):** El aspecto más poderoso de los callbacks radica en cómo su **valor de retorno** influye en las acciones subsecuentes del agente. Así es como interceptas y controlas el flujo de ejecución:

1. **`return None` (Permitir Comportamiento Predeterminado):**

    * El tipo de retorno específico puede variar dependiendo del lenguaje. En Java, el tipo de retorno equivalente es `Optional.empty()`. Consulta la documentación de la API para orientación específica del lenguaje.
    * Esta es la forma estándar de señalar que tu callback ha terminado su trabajo (por ejemplo, registro, inspección, modificaciones menores a argumentos de entrada *mutables* como `llm_request`) y que el agente de ADK debería **proceder con su operación normal**.
    * Para callbacks `before_*` (`before_agent`, `before_model`, `before_tool`), retornar `None` significa que el siguiente paso en la secuencia (ejecutar la lógica del agente, llamar al LLM, ejecutar la herramienta) ocurrirá.
    * Para callbacks `after_*` (`after_agent`, `after_model`, `after_tool`), retornar `None` significa que el resultado recién producido por el paso precedente (la salida del agente, la respuesta del LLM, el resultado de la herramienta) se usará tal cual.

2. **`return <Objeto Específico>` (Anular Comportamiento Predeterminado):**

    * Retornar un *tipo específico de objeto* (en lugar de `None`) es cómo **anulas** el comportamiento predeterminado del agente de ADK. El framework usará el objeto que retornas y *omitirá* el paso que normalmente seguiría o *reemplazará* el resultado que acaba de ser generado.
    * **`before_agent_callback` → `types.Content`**: Omite la lógica de ejecución principal del agente (`_run_async_impl` / `_run_live_impl`). El objeto `Content` retornado se trata inmediatamente como la salida final del agente para este turno. Útil para manejar solicitudes simples directamente o aplicar control de acceso.
    * **`before_model_callback` → `LlmResponse`**: Omite la llamada al Modelo de Lenguaje Grande externo. El objeto `LlmResponse` retornado se procesa como si fuera la respuesta real del LLM. Ideal para implementar barreras de seguridad de entrada, validación de prompts o servir respuestas en caché.
    * **`before_tool_callback` → `dict` o `Map`**: Omite la ejecución de la función de herramienta real (o sub-agente). El `dict` retornado se usa como el resultado de la llamada a la herramienta, que luego típicamente se pasa de vuelta al LLM. Perfecto para validar argumentos de herramientas, aplicar restricciones de políticas o retornar resultados de herramientas simulados/en caché.
    * **`after_agent_callback` → `types.Content`**: *Reemplaza* el `Content` que la lógica de ejecución del agente acaba de producir.
    * **`after_model_callback` → `LlmResponse`**: *Reemplaza* el `LlmResponse` recibido del LLM. Útil para sanitizar salidas, agregar avisos estándar o modificar la estructura de respuesta del LLM.
    * **`after_tool_callback` → `dict` o `Map`**: *Reemplaza* el resultado `dict` retornado por la herramienta. Permite post-procesamiento o estandarización de salidas de herramientas antes de que se envíen de vuelta al LLM.

**Ejemplo de Código Conceptual (Barrera de Seguridad):**

Este ejemplo demuestra el patrón común para una barrera de seguridad usando `before_model_callback`.

??? "Code"
    === "Python"

        ```python
        --8<-- "examples/python/snippets/callbacks/before_model_callback.py"
        ```

    === "Typescript"
        ```typescript
        --8<-- "examples/typescript/snippets/callbacks/before_model_callback.ts"
        ```

    === "Go"
        ```go
        --8<-- "examples/go/snippets/callbacks/main.go:imports"


        --8<-- "examples/go/snippets/callbacks/main.go:guardrail_init"
        ```

    === "Java"
        ```java
        --8<-- "examples/java/snippets/src/main/java/callbacks/BeforeModelGuardrailExample.java:init"
        ```

Al entender este mecanismo de retornar `None` versus retornar objetos específicos, puedes controlar con precisión la ruta de ejecución del agente, haciendo de los callbacks una herramienta esencial para construir agentes sofisticados y confiables con ADK.