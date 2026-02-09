# Observabilidad de Agentes con AgentOps

**Con solo dos líneas de código**, [AgentOps](https://www.agentops.ai) proporciona repeticiones de sesión, métricas y monitoreo para agentes.

## ¿Por qué AgentOps para ADK?

La observabilidad es un aspecto clave en el desarrollo e implementación de agentes de IA conversacional. Permite a los desarrolladores comprender cómo se están desempeñando sus agentes, cómo están interactuando con los usuarios y cómo utilizan herramientas y APIs externas.

Al integrar AgentOps, los desarrolladores pueden obtener información profunda sobre el comportamiento de sus agentes ADK, las interacciones con LLM y el uso de herramientas.

Google ADK incluye su propio sistema de rastreo basado en OpenTelemetry, dirigido principalmente a proporcionar a los desarrolladores una forma de rastrear el flujo básico de ejecución dentro de sus agentes. AgentOps mejora esto al ofrecer una plataforma de observabilidad dedicada y más completa con:

*   **Rastreo Unificado y Análisis de Repetición:** Consolida rastros de ADK y otros componentes de tu stack de IA.
*   **Visualización Enriquecida:** Paneles intuitivos para visualizar el flujo de ejecución del agente, llamadas LLM y rendimiento de herramientas.
*   **Depuración Detallada:** Profundiza en spans específicos, visualiza prompts, completaciones, conteo de tokens y errores.
*   **Seguimiento de Costos y Latencia de LLM:** Rastrea latencias, costos (a través del uso de tokens) e identifica cuellos de botella.
*   **Configuración Simplificada:** Comienza con solo unas pocas líneas de código.

![AgentOps Agent Observability Dashboard](https://raw.githubusercontent.com/AgentOps-AI/agentops/refs/heads/main/docs/images/external/app_screenshots/overview.png)

![AgentOps Dashboard showing an ADK trace with nested agent, LLM, and tool spans.](../assets/agentops-adk-trace-example.jpg)

*Panel de AgentOps mostrando un rastro de una ejecución de aplicación ADK de múltiples pasos. Puedes ver la estructura jerárquica de los spans, incluyendo el flujo de trabajo del agente principal, sub-agentes individuales, llamadas LLM y ejecuciones de herramientas. Nota la jerarquía clara: el span del agente de flujo de trabajo principal contiene spans hijos para varias operaciones de sub-agentes, llamadas LLM y ejecuciones de herramientas.*

## Comenzando con AgentOps y ADK

Integrar AgentOps en tu aplicación ADK es sencillo:

1.  **Instala AgentOps:**
    ```bash
    pip install -U agentops
    ```

2. **Crea una Clave API**
    Crea una clave API de usuario aquí: [Create API Key](https://app.agentops.ai/settings/projects) y configura tu entorno:

    Agrega tu clave API a tus variables de entorno:
    ```
    AGENTOPS_API_KEY=<YOUR_AGENTOPS_API_KEY>
    ```

3.  **Inicializa AgentOps:**
    Agrega las siguientes líneas al inicio de tu script de aplicación ADK (ej., tu archivo Python principal que ejecuta el `Runner` de ADK):

    ```python
    import agentops
    agentops.init()
    ```

    Esto iniciará una sesión de AgentOps y rastreará automáticamente los agentes ADK.

    Ejemplo detallado:

    ```python
    import agentops
    import os
    from dotenv import load_dotenv

    # Carga variables de entorno (opcional, si usas un archivo .env para claves API)
    load_dotenv()

    agentops.init(
        api_key=os.getenv("AGENTOPS_API_KEY"), # Tu Clave API de AgentOps
        trace_name="my-adk-app-trace"  # Opcional: Un nombre para tu rastro
        # auto_start_session=True es el valor predeterminado.
        # Establece en False si quieres controlar manualmente el inicio/fin de sesión.
    )
    ```

    > 🚨 🔑 Puedes encontrar tu clave API de AgentOps en tu [AgentOps Dashboard](https://app.agentops.ai/) después de registrarte. Se recomienda establecerla como una variable de entorno (`AGENTOPS_API_KEY`).

Una vez inicializado, AgentOps comenzará automáticamente a instrumentar tu agente ADK.

**Esto es todo lo que necesitas para capturar todos los datos de telemetría de tu agente ADK**

## Cómo AgentOps Instrumenta ADK

AgentOps emplea una estrategia sofisticada para proporcionar observabilidad sin problemas sin entrar en conflicto con la telemetría nativa de ADK:

1.  **Neutralizando la Telemetría Nativa de ADK:**
    AgentOps detecta ADK y parchea inteligentemente el rastreador interno de OpenTelemetry de ADK (típicamente `trace.get_tracer('gcp.vertex.agent')`). Lo reemplaza con un `NoOpTracer`, asegurando que los propios intentos de ADK de crear spans de telemetría sean efectivamente silenciados. Esto previene rastros duplicados y permite que AgentOps sea la fuente autorizada para los datos de observabilidad.

2.  **Creación de Span Controlada por AgentOps:**
    AgentOps toma el control envolviendo métodos clave de ADK para crear una jerarquía lógica de spans:

    *   **Spans de Ejecución de Agente (ej., `adk.agent.MySequentialAgent`):**
        Cuando un agente ADK (como `BaseAgent`, `SequentialAgent` o `LlmAgent`) inicia su método `run_async`, AgentOps inicia un span padre para la ejecución de ese agente.

    *   **Spans de Interacción con LLM (ej., `adk.llm.gemini-pro`):**
        Para llamadas realizadas por un agente a un LLM (a través de `BaseLlmFlow._call_llm_async` de ADK), AgentOps crea un span hijo dedicado, típicamente nombrado según el modelo LLM. Este span captura detalles de la solicitud (prompts, parámetros del modelo) y, al completarse (a través de `_finalize_model_response_event` de ADK), registra detalles de respuesta como completaciones, uso de tokens y razones de finalización.

    *   **Spans de Uso de Herramientas (ej., `adk.tool.MyCustomTool`):**
        Cuando un agente usa una herramienta (a través de `functions.__call_tool_async` de ADK), AgentOps crea un único span hijo completo nombrado según la herramienta. Este span incluye los parámetros de entrada de la herramienta y el resultado que devuelve.

3.  **Recopilación de Atributos Enriquecidos:**
    AgentOps reutiliza la lógica de extracción de datos interna de ADK. Parchea las funciones específicas de telemetría de ADK (ej., `google.adk.telemetry.trace_tool_call`, `trace_call_llm`). Los envoltorios de AgentOps para estas funciones toman la información detallada que ADK recopila y la adjuntan como atributos al *span de AgentOps actualmente activo*.

## Visualizando tu Agente ADK en AgentOps

Cuando instrumentas tu aplicación ADK con AgentOps, obtienes una vista clara y jerárquica de la ejecución de tu agente en el panel de AgentOps.

1.  **Inicialización:**
    Cuando se llama a `agentops.init()` (ej., `agentops.init(trace_name="my_adk_application")`), se crea un span padre inicial si el parámetro init `auto_start_session=True` (verdadero por defecto). Este span, a menudo nombrado de forma similar a `my_adk_application.session`, será la raíz para todas las operaciones dentro de ese rastro.

2.  **Ejecución del Runner de ADK:**
    Cuando un `Runner` de ADK ejecuta un agente de nivel superior (ej., un `SequentialAgent` orquestando un flujo de trabajo), AgentOps crea un span de agente correspondiente bajo el rastro de sesión. Este span reflejará el nombre de tu agente ADK de nivel superior (ej., `adk.agent.YourMainWorkflowAgent`).

3.  **Llamadas a Sub-Agentes y LLM/Herramientas:**
    A medida que este agente principal ejecuta su lógica, incluyendo llamadas a sub-agentes, LLMs o herramientas:
    *   Cada **ejecución de sub-agente** aparecerá como un span hijo anidado bajo su agente padre.
    *   Las llamadas a **Modelos de Lenguaje Grandes** generarán más spans hijos anidados (ej., `adk.llm.<model_name>`), capturando detalles del prompt, respuestas y uso de tokens.
    *   Las **invocaciones de herramientas** también resultarán en spans hijos distintos (ej., `adk.tool.<your_tool_name>`), mostrando sus parámetros y resultados.

Esto crea una cascada de spans, permitiéndote ver la secuencia, duración y detalles de cada paso en tu aplicación ADK. Todos los atributos relevantes, como prompts LLM, completaciones, conteo de tokens, entradas/salidas de herramientas y nombres de agentes, son capturados y mostrados.

Para una demostración práctica, puedes explorar un cuaderno Jupyter de ejemplo que ilustra un flujo de trabajo de aprobación humana usando Google ADK y AgentOps:
[Google ADK Human Approval Example on GitHub](https://github.com/AgentOps-AI/agentops/blob/main/examples/google_adk/human_approval.ipynb).

Este ejemplo muestra cómo se visualiza en AgentOps un proceso de agente de múltiples pasos con uso de herramientas.

## Beneficios

*   **Configuración sin Esfuerzo:** Cambios mínimos de código para un rastreo completo de ADK.
*   **Visibilidad Profunda:** Comprende el funcionamiento interno de flujos de agentes ADK complejos.
*   **Depuración Más Rápida:** Identifica rápidamente problemas con datos de rastro detallados.
*   **Optimización de Rendimiento:** Analiza latencias y uso de tokens.

Al integrar AgentOps, los desarrolladores de ADK pueden mejorar significativamente su capacidad para construir, depurar y mantener agentes de IA robustos.

## Información Adicional

Para comenzar, [crea una cuenta de AgentOps](http://app.agentops.ai). Para solicitudes de características o reportes de errores, por favor contacta al equipo de AgentOps en el [AgentOps Repo](https://github.com/AgentOps-AI/agentops).

### Enlaces Extra
🐦 [Twitter](http://x.com/agentopsai)   •   📢 [Discord](http://x.com/agentopsai)   •   🖇️ [AgentOps Dashboard](http://app.agentops.ai)   •   📙 [Documentation](http://docs.agentops.ai)