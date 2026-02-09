# Agentes de Flujo de Trabajo

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span><span class="lst-typescript">TypeScript</span><span class="lst-go">Go</span><span class="lst-java">Java</span>
</div>

Esta sección introduce los "*agentes de flujo de trabajo*" - **agentes especializados que controlan el flujo de ejecución de sus sub-agentes**.

Los agentes de flujo de trabajo son componentes especializados en ADK diseñados puramente para **orquestar el flujo de ejecución de los sub-agentes**. Su rol principal es gestionar *cómo* y *cuándo* se ejecutan otros agentes, definiendo el flujo de control de un proceso.

A diferencia de los [Agentes LLM](../llm-agents.md), que utilizan Modelos de Lenguaje Grandes para razonamiento dinámico y toma de decisiones, los Agentes de Flujo de Trabajo operan basándose en **lógica predefinida**. Determinan la secuencia de ejecución según su tipo (por ejemplo, secuencial, paralelo, bucle) sin consultar un LLM para la orquestación en sí. Esto resulta en **patrones de ejecución determinísticos y predecibles**.

ADK proporciona tres tipos principales de agentes de flujo de trabajo, cada uno implementando un patrón de ejecución distinto:

<div class="grid cards" markdown>

- :material-console-line: **Agentes Secuenciales**

    ---

    Ejecuta sub-agentes uno tras otro, en **secuencia**.

    [:octicons-arrow-right-24: Aprender más](sequential-agents.md)

- :material-console-line: **Agentes de Bucle**

    ---

    Ejecuta **repetidamente** sus sub-agentes hasta que se cumple una condición de terminación específica.

    [:octicons-arrow-right-24: Aprender más](loop-agents.md)

- :material-console-line: **Agentes Paralelos**

    ---

    Ejecuta múltiples sub-agentes en **paralelo**.

    [:octicons-arrow-right-24: Aprender más](parallel-agents.md)

</div>

## ¿Por Qué Usar Agentes de Flujo de Trabajo?

Los agentes de flujo de trabajo son esenciales cuando necesitas control explícito sobre cómo se ejecuta una serie de tareas o agentes. Proporcionan:

* **Predictibilidad:** El flujo de ejecución está garantizado según el tipo de agente y la configuración.
* **Confiabilidad:** Asegura que las tareas se ejecuten en el orden o patrón requerido de manera consistente.
* **Estructura:** Te permite construir procesos complejos componiendo agentes dentro de estructuras de control claras.

Mientras que el agente de flujo de trabajo gestiona el flujo de control de manera determinística, los sub-agentes que orquesta pueden ser ellos mismos de cualquier tipo de agente, incluidas instancias inteligentes de Agente LLM. Esto te permite combinar control de proceso estructurado con ejecución de tareas flexible impulsada por LLM.