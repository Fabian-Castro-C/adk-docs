# Desplegar en Vertex AI Agent Engine

<div class="language-support-tag" title="Vertex AI Agent Engine currently supports only Python.">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span>
</div>

Google Cloud Vertex AI
[Agent Engine](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/overview)
es un conjunto de servicios modulares que ayudan a los desarrolladores a escalar y gobernar agentes en
producción. El runtime de Agent Engine te permite desplegar agentes en producción
con infraestructura administrada de extremo a extremo para que puedas enfocarte en crear agentes
inteligentes e impactantes. Cuando despliegas un agente ADK en Agent Engine, tu código
se ejecuta en el entorno *runtime de Agent Engine*, que es parte del conjunto más amplio de
servicios de agentes proporcionados por el producto Agent Engine.

Esta guía incluye las siguientes rutas de despliegue, que sirven para diferentes
propósitos:

*   **[Despliegue estándar](/deploy/agent-engine/deploy/)**: Sigue
    esta ruta de despliegue estándar si tienes un proyecto existente de Google Cloud
    y si deseas gestionar cuidadosamente el despliegue de un agente ADK en el
    runtime de Agent Engine. Esta ruta de despliegue utiliza Cloud Console, la interfaz de línea de
    comandos de ADK, y proporciona instrucciones paso a paso. Esta ruta es recomendada
    para usuarios que ya están familiarizados con la configuración de proyectos de Google Cloud,
    y usuarios que se preparan para despliegues en producción.

*   **[Despliegue con Agent Starter Pack](/deploy/agent-engine/asp/)**:
    Sigue esta ruta de despliegue acelerada si no tienes un proyecto existente de
    Google Cloud y estás creando un proyecto específicamente para desarrollo
    y pruebas. El Agent Starter Pack (ASP) te ayuda a desplegar proyectos ADK
    rápidamente y configura servicios de Google Cloud que no son estrictamente
    necesarios para ejecutar un agente ADK con el runtime de Agent Engine.

!!! note "Servicio Agent Engine en Google Cloud"

    Agent Engine es un servicio de pago y puedes incurrir en costos si superas
    el nivel de acceso sin costo. Más información se puede encontrar en la
    [página de precios de Agent Engine](https://cloud.google.com/vertex-ai/pricing#vertex-ai-agent-engine).

## Carga útil de despliegue {#payload}

Cuando despliegas tu proyecto de agente ADK en Agent Engine, el siguiente contenido se
sube al servicio:

- Tu código de agente ADK
- Cualquier dependencia declarada en tu código de agente ADK

El despliegue *no* incluye el servidor API de ADK o las bibliotecas de la interfaz de usuario
web de ADK. El servicio Agent Engine proporciona las bibliotecas para la funcionalidad del
servidor API de ADK.