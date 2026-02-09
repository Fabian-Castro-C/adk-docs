# Registro (Logging) en el Agent Development Kit (ADK)

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

El Agent Development Kit (ADK) utiliza el módulo estándar `logging` de Python para proporcionar capacidades de registro flexibles y potentes. Entender cómo configurar e interpretar estos registros es crucial para monitorear el comportamiento del agente y depurar problemas de manera efectiva.

## Filosofía de Registro

El enfoque de ADK para el registro es proporcionar información de diagnóstico detallada sin ser excesivamente verboso por defecto. Está diseñado para ser configurado por el desarrollador de la aplicación, permitiéndote adaptar la salida de registro a tus necesidades específicas, ya sea en un entorno de desarrollo o producción.

- **Biblioteca Estándar:** Utiliza la biblioteca estándar `logging`, por lo que cualquier configuración o manejador que funcione con ella funcionará con ADK.
- **Registradores Jerárquicos:** Los registradores se nombran jerárquicamente según la ruta del módulo (por ejemplo, `google_adk.google.adk.agents.llm_agent`), permitiendo un control fino sobre qué partes del framework producen registros.
- **Configurado por el Usuario:** El framework no configura el registro por sí mismo. Es responsabilidad del desarrollador que usa el framework configurar la configuración de registro deseada en el punto de entrada de su aplicación.

## Cómo Configurar el Registro

Puedes configurar el registro en tu script principal de la aplicación (por ejemplo, `main.py`) antes de inicializar y ejecutar tu agente. La forma más simple es usar `logging.basicConfig`.

### Ejemplo de Configuración

Para habilitar el registro detallado, incluyendo mensajes de nivel `DEBUG`, agrega lo siguiente al principio de tu script:

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(levelname)s - %(name)s - %(message)s'
)

# Tu código de agente ADK sigue...
# from google.adk.agents import LlmAgent
# ...
```

### Configurando el Registro con la CLI de ADK

Al ejecutar agentes usando los servidores web o API integrados de ADK, puedes controlar fácilmente la verbosidad del registro directamente desde la línea de comandos. Los comandos `adk web`, `adk api_server` y `adk deploy cloud_run` aceptan una opción `--log_level`.

Esto proporciona una forma conveniente de establecer el nivel de registro sin modificar el código fuente de tu agente.

> **Nota:** La configuración de línea de comandos siempre tiene precedencia sobre la configuración programática (como `logging.basicConfig`) para los registradores de ADK. Se recomienda usar `INFO` o `WARNING` en producción y habilitar `DEBUG` solo cuando se esté solucionando problemas.

**Ejemplo usando `adk web`:**

Para iniciar el servidor web con registro de nivel `DEBUG`, ejecuta:

```bash
adk web --log_level DEBUG path/to/your/agents_dir
```

Los niveles de registro disponibles para la opción `--log_level` son:

- `DEBUG`
- `INFO` (por defecto)
- `WARNING`
- `ERROR`
- `CRITICAL`

> También puedes usar `-v` o `--verbose` como atajo para `--log_level DEBUG`.
>
> ```bash
> adk web -v path/to/your/agents_dir
> ```

### Niveles de Registro

ADK utiliza niveles de registro estándar para categorizar mensajes. El nivel configurado determina qué información se registra.

| Nivel | Descripción | Tipo de Información Registrada  |
| :--- | :--- | :--- |
| **`DEBUG`** | **Crucial para la depuración.** El nivel más verboso para información de diagnóstico detallada. | <ul><li>**Prompts Completos del LLM:** La solicitud completa enviada al modelo de lenguaje, incluyendo instrucciones del sistema, historial y herramientas.</li><li>Respuestas detalladas de API de servicios.</li><li>Transiciones de estado interno y valores de variables.</li></ul> |
| **`INFO`** | Información general sobre el ciclo de vida del agente. | <ul><li>Inicialización y arranque del agente.</li><li>Eventos de creación y eliminación de sesiones.</li><li>Ejecución de una herramienta, incluyendo su nombre y argumentos.</li></ul> |
| **`WARNING`** | Indica un problema potencial o uso de características obsoletas. El agente continúa funcionando, pero puede requerir atención. | <ul><li>Uso de métodos o parámetros obsoletos.</li><li>Errores no críticos de los que el sistema se recuperó.</li></ul> |
| **`ERROR`** | Un error grave que impidió que una operación se completara. | <ul><li>Llamadas de API fallidas a servicios externos (por ejemplo, LLM, Servicio de Sesión).</li><li>Excepciones no manejadas durante la ejecución del agente.</li><li>Errores de configuración.</li></ul> |

> **Nota:** Se recomienda usar `INFO` o `WARNING` en entornos de producción. Solo habilita `DEBUG` cuando estés solucionando activamente un problema, ya que los registros `DEBUG` pueden ser muy verbosos y pueden contener información sensible.

## Leer y Entender los Registros

La cadena `format` en el ejemplo de `basicConfig` determina la estructura de cada mensaje de registro.

Aquí hay una entrada de registro de ejemplo:

```text
2025-07-08 11:22:33,456 - DEBUG - google_adk.google.adk.models.google_llm - LLM Request: contents { ... }
```

| Segmento de Registro            | Especificador de Formato | Significado                                         |
| ------------------------------- | ---------------- | ---------------------------------------------- |
| `2025-07-08 11:22:33,456`       | `%(asctime)s`    | Marca de tiempo                                      |
| `DEBUG`                         | `%(levelname)s`  | Nivel de severidad                                 |
| `google_adk.models.google_llm`  | `%(name)s`       | Nombre del registrador (el módulo que produjo el registro) |
| `LLM Request: contents { ... }` | `%(message)s`    | El mensaje de registro real                         |

Al leer el nombre del registrador, puedes identificar inmediatamente la fuente del registro y entender su contexto dentro de la arquitectura del agente.

## Depurando con Registros: Un Ejemplo Práctico

**Escenario:** Tu agente no está produciendo la salida esperada, y sospechas que el prompt que se envía al LLM es incorrecto o le falta información.

**Pasos:**

1.  **Habilitar el Registro DEBUG:** En tu `main.py`, establece el nivel de registro a `DEBUG` como se muestra en el ejemplo de configuración.

    ```python
    logging.basicConfig(
        level=logging.DEBUG,
        format='%(asctime)s - %(levelname)s - %(name)s - %(message)s'
    )
    ```

2.  **Ejecutar tu Agente:** Ejecuta la tarea de tu agente como lo harías normalmente.

3.  **Inspeccionar los Registros:** Busca en la salida de la consola un mensaje del registrador `google.adk.models.google_llm` que comience con `LLM Request:`.

    ```log
    ...
    2025-07-10 15:26:13,778 - DEBUG - google_adk.google.adk.models.google_llm - Sending out request, model: gemini-2.0-flash, backend: GoogleLLMVariant.GEMINI_API, stream: False
    2025-07-10 15:26:13,778 - DEBUG - google_adk.google.adk.models.google_llm -
    LLM Request:
    -----------------------------------------------------------
    System Instruction:

          You roll dice and answer questions about the outcome of the dice rolls.
          You can roll dice of different sizes.
          You can use multiple tools in parallel by calling functions in parallel(in one request and in one round).
          It is ok to discuss previous dice roles, and comment on the dice rolls.
          When you are asked to roll a die, you must call the roll_die tool with the number of sides. Be sure to pass in an integer. Do not pass in a string.
          You should never roll a die on your own.
          When checking prime numbers, call the check_prime tool with a list of integers. Be sure to pass in a list of integers. You should never pass in a string.
          You should not check prime numbers before calling the tool.
          When you are asked to roll a die and check prime numbers, you should always make the following two function calls:
          1. You should first call the roll_die tool to get a roll. Wait for the function response before calling the check_prime tool.
          2. After you get the function response from roll_die tool, you should call the check_prime tool with the roll_die result.
            2.1 If user asks you to check primes based on previous rolls, make sure you include the previous rolls in the list.
          3. When you respond, you must include the roll_die result from step 1.
          You should always perform the previous 3 steps when asking for a roll and checking prime numbers.
          You should not rely on the previous history on prime results.


    You are an agent. Your internal name is "hello_world_agent".

    The description about you is "hello world agent that can roll a dice of 8 sides and check prime numbers."
    -----------------------------------------------------------
    Contents:
    {"parts":[{"text":"Roll a 6 sided dice"}],"role":"user"}
    {"parts":[{"function_call":{"args":{"sides":6},"name":"roll_die"}}],"role":"model"}
    {"parts":[{"function_response":{"name":"roll_die","response":{"result":2}}}],"role":"user"}
    -----------------------------------------------------------
    Functions:
    roll_die: {'sides': {'type': <Type.INTEGER: 'INTEGER'>}}
    check_prime: {'nums': {'items': {'type': <Type.INTEGER: 'INTEGER'>}, 'type': <Type.ARRAY: 'ARRAY'>}}
    -----------------------------------------------------------

    2025-07-10 15:26:13,779 - INFO - google_genai.models - AFC is enabled with max remote calls: 10.
    2025-07-10 15:26:14,309 - INFO - google_adk.google.adk.models.google_llm -
    LLM Response:
    -----------------------------------------------------------
    Text:
    I have rolled a 6 sided die, and the result is 2.
    ...
    ```

4.  **Analizar el Prompt:** Al examinar las secciones `System Instruction`, `contents` y `functions` de la solicitud registrada, puedes verificar:
    -   ¿Es correcta la instrucción del sistema?
    -   ¿Es preciso el historial de conversación (turnos `user` y `model`)?
    -   ¿Está incluida la consulta más reciente del usuario?
    -   ¿Se están proporcionando las herramientas correctas al modelo?
    -   ¿Las herramientas están siendo llamadas correctamente por el modelo?
    -   ¿Cuánto tiempo tarda el modelo en responder?

Esta salida detallada te permite diagnosticar una amplia gama de problemas, desde ingeniería de prompts incorrecta hasta problemas con definiciones de herramientas, directamente desde los archivos de registro.