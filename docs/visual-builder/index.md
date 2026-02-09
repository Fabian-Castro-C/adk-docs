# Constructor Visual para agentes

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v1.18.0</span><span class="lst-preview">Experimental</span>
</div>

El Constructor Visual de ADK es una herramienta basada en web que proporciona un entorno
visual de diseño de flujos de trabajo para crear y gestionar agentes ADK. Te permite
diseñar, construir y probar tus agentes en una interfaz gráfica amigable para principiantes,
e incluye un asistente impulsado por IA para ayudarte a construir agentes.

![Visual Agent Builder](../assets/visual-builder.png)

!!! example "Experimental"
    La función Constructor Visual es un lanzamiento experimental. ¡Damos la bienvenida a tus
    [comentarios](https://github.com/google/adk-python/issues/new?template=feature_request.md)!

## Primeros pasos

La interfaz del Constructor Visual es parte de la interfaz de usuario de la herramienta ADK Web.
Asegúrate de tener la biblioteca ADK
[instalada](/adk-docs/get-started/installation/#python)
y luego ejecuta la interfaz de usuario de ADK Web.

```console
adk web --port 8000
```

??? tip "Consejo: Ejecuta desde un directorio de desarrollo de código"

    La herramienta Constructor Visual escribe archivos de proyecto en nuevos subdirectorios ubicados
    en el directorio desde donde ejecutas la herramienta ADK Web. Asegúrate de ejecutar este
    comando desde un directorio de desarrollo donde tengas acceso de escritura.

![Visual Agent Builder start](../assets/visual-builder-start.png)
**Figura 1:** Controles de ADK Web para iniciar la herramienta Constructor Visual.

Para crear un agente con el Constructor Visual:

1.  En la parte superior izquierda de la página, selecciona el **+** (signo más), como se muestra en la *Figura 1*, para comenzar a crear un agente.
1.  Escribe un nombre para tu aplicación de agente y selecciona **Create**.
1.  Edita tu agente haciendo cualquiera de lo siguiente:
    *   En el panel izquierdo, edita los valores de los componentes del agente.
    *   En el panel central, agrega nuevos componentes de agente.
    *   En el panel derecho, usa indicaciones para modificar el agente u obtener ayuda.
1.  En la esquina inferior izquierda, selecciona **Save** para guardar tu agente.
1.  Interactúa con tu nuevo agente para probarlo.
1.  En la parte superior izquierda de la página, selecciona el ícono de lápiz, como se muestra en la *Figura 1*, para continuar editando tu agente.

Aquí hay algunas cosas a tener en cuenta al usar el Constructor Visual:

*   **Crear agente y guardar:** Al crear un agente, asegúrate de seleccionar
    **Save** antes de salir de la interfaz de edición, de lo contrario tu nuevo agente puede
    no ser editable.
*   **Edición de agente:** La edición (ícono de lápiz) para agentes *solo* está disponible para
    agentes creados con el Constructor Visual
*   **Agregar herramientas:** Al agregar Herramientas personalizadas existentes a un agente del Constructor Visual,
    especifica un nombre de función Python completamente calificado.

## Soporte de componentes de flujo de trabajo

La herramienta Constructor Visual proporciona una interfaz de usuario de arrastrar y soltar para construir agentes, así
como un Asistente de desarrollo impulsado por IA que puede responder preguntas y editar tu flujo de trabajo de agente.
La herramienta soporta todos los componentes esenciales para construir un flujo de trabajo de agente ADK, incluyendo:

*   **Agentes**
    *   **Agente Raíz**: El agente controlador principal para un flujo de trabajo. Todos los demás agentes en
        un flujo de trabajo de agente ADK se consideran Sub Agentes.
    *   [**Agente LLM:**](/adk-docs/agents/llm-agents/)
        Un agente impulsado por un modelo de IA generativa.
    *   [**Agente Secuencial:**](/adk-docs/agents/workflow-agents/sequential-agents/)
        Un agente de flujo de trabajo que ejecuta una serie de sub-agentes en una secuencia.
    *   [**Agente de Bucle:**](/adk-docs/agents/workflow-agents/loop-agents/)
        Un agente de flujo de trabajo que ejecuta repetidamente un sub-agente hasta que se cumple cierta condición.
    *   [**Agente Paralelo:**](/adk-docs/agents/workflow-agents/parallel-agents/)
        Un agente de flujo de trabajo que ejecuta múltiples sub-agentes concurrentemente.
*   **Herramientas**
    *   [**Herramientas preconstruidas:**](/adk-docs/tools/built-in-tools/)
        Un conjunto limitado de herramientas proporcionadas por ADK que se pueden agregar a los agentes.
    *   [**Herramientas personalizadas:**](/adk-docs/tools-custom/)
        Puedes construir y agregar herramientas personalizadas a tu flujo de trabajo.
*   **Componentes**
    *   [**Callbacks**](/adk-docs/callbacks/)
        Un componente de control de flujo que te permite modificar el comportamiento de los agentes al inicio
        y al final de los eventos del flujo de trabajo del agente.

Algunas funciones avanzadas de ADK no son soportadas por el Constructor Visual debido a
limitaciones de la función Agent Config. Para más información, consulta las
[Limitaciones conocidas](/adk-docs/agents/config/#known-limitations) de Agent Config.

## Salida de código del proyecto

La herramienta Constructor Visual genera código en el formato [Agent Config](/adk-docs/agents/config/),
usando archivos de configuración `.yaml` para agentes y código Python para herramientas
personalizadas. Estos archivos se generan en una subcarpeta del directorio donde ejecutaste
la interfaz ADK Web. El siguiente listado muestra un diseño de ejemplo para un
proyecto DiceAgent:

```none
DiceAgent/
    root_agent.yaml    # código del agente principal
    sub_agent_1.yaml   # sub agentes (si hay)
    tools/             # directorio de herramientas
        __init__.py
        dice_tool.py   # código de la herramienta
```

!!! note "Editando agentes generados"

    Puedes editar los archivos generados en tu entorno de desarrollo. Sin embargo,
    algunos cambios pueden no ser compatibles con el Constructor Visual.

## Próximos pasos

Usando el Asistente de desarrollo del Constructor Visual, intenta construir un nuevo agente usando
esta indicación:

```none
Help me add a dice roll tool to my current agent.
Use the default model if you need to configure that.
```

Consulta más información sobre el formato de código Agent Config usado por el Constructor Visual
y las opciones disponibles:

*   [Agent Config](/adk-docs/agents/config/)
*   [Esquema YAML de Agent Config](/adk-docs/api-reference/agentconfig/)