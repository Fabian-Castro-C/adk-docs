# Bidi-streaming (en vivo) en ADK

<div class="language-support-tag">
    <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.5.0</span><span class="lst-preview">Experimental</span>
</div>
  
El streaming bidireccional (Bidi) (en vivo) en ADK añade la capacidad de interacción de voz y video bidireccional de baja latencia de la [API de Gemini Live](https://ai.google.dev/gemini-api/docs/live) a los agentes de IA.

Con el modo bidi-streaming, o en vivo, puedes proporcionar a los usuarios finales la experiencia de conversaciones de voz naturales y similares a las humanas, incluyendo la capacidad de que el usuario interrumpa las respuestas del agente con comandos de voz. Los agentes con streaming pueden procesar entradas de texto, audio y video, y pueden proporcionar salidas de texto y audio.

<div class="video-grid">
  <div class="video-item">
    <div class="video-container">
      <iframe src="https://www.youtube-nocookie.com/embed/vLUkAGeLR1k" title="ADK Bidi-streaming in 5 minutes" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
  </div>
  <div class="video-item">
    <div class="video-container">
      <iframe src="https://www.youtube-nocookie.com/embed/Hwx94smxT_0" title="Shopper's Concierge 2 Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
  </div>
</div>



<div class="grid cards" markdown>

-   :material-console-line: **Inicio rápido (Bidi-streaming)**

    ---

    En este inicio rápido, construirás un agente simple y usarás streaming en ADK para implementar comunicación de voz y video bidireccional de baja latencia.

    - [Inicio rápido (Bidi-streaming)](../get-started/streaming/quickstart-streaming.md)

-   :material-console-line: **Aplicación Demo de Bidi-streaming**

    ---

    Una implementación de referencia lista para producción que muestra el streaming bidireccional de ADK con soporte multimodal (texto, audio, imagen). Esta demo basada en FastAPI demuestra comunicación WebSocket en tiempo real, transcripción automática, llamadas a herramientas con Google Search y gestión completa del ciclo de vida del streaming. Esta demo se referencia ampliamente a lo largo de la serie de guías de desarrollo.

    - [Demo de ADK Bidi-streaming](https://github.com/google/adk-samples/tree/main/python/agents/bidi-demo)

-   :material-console-line: **Artículo del blog: Guía Visual de ADK Bidi-streaming**

    ---

    Una guía visual para el desarrollo de agentes de IA multimodal en tiempo real con ADK Bidi-streaming. Este artículo proporciona diagramas e ilustraciones intuitivas para ayudarte a entender cómo funciona Bidi-streaming y cómo construir agentes de IA interactivos.

    - [Artículo del blog: Guía Visual de ADK Bidi-streaming](https://medium.com/google-cloud/adk-bidi-streaming-a-visual-guide-to-real-time-multimodal-ai-agent-development-62dd08c81399)

-   :material-console-line: **Serie de guías de desarrollo de Bidi-streaming**

    ---

    Una serie de artículos para profundizar en el desarrollo de Bidi-streaming con ADK. Puedes aprender conceptos básicos y casos de uso, la API principal y el diseño de aplicaciones de extremo a extremo.

    - [Parte 1: Introducción a ADK Bidi-streaming](dev-guide/part1.md) - Fundamentos de Bidi-streaming, tecnología de Live API, componentes de arquitectura de ADK y ciclo de vida completo de la aplicación con ejemplos de FastAPI
    - [Parte 2: Envío de mensajes con LiveRequestQueue](dev-guide/part2.md) - Flujo de mensajes ascendente, envío de texto/audio/video, señales de actividad y patrones de concurrencia
    - [Parte 3: Manejo de eventos con run_live()](dev-guide/part3.md) - Procesamiento de eventos, manejo de texto/audio/transcripciones, ejecución automática de herramientas y flujos de trabajo multi-agente
    - [Parte 4: Entendiendo RunConfig](dev-guide/part4.md) - Modalidades de respuesta, modos de streaming, gestión de sesiones, reanudación de sesiones, compresión de ventana de contexto y gestión de cuotas
    - [Parte 5: Cómo usar Audio, Imagen y Video](dev-guide/part5.md) - Especificaciones de audio, arquitecturas de modelo, transcripción de audio, detección de actividad de voz y características de diálogo proactivo/afectivo

-   :material-console-line: **Herramientas de Streaming**

    ---

    Las herramientas de streaming permiten que las herramientas (funciones) transmitan resultados intermedios de vuelta a los agentes y los agentes pueden responder a esos resultados intermedios. Por ejemplo, podemos usar herramientas de streaming para monitorear los cambios en el precio de las acciones y hacer que el agente reaccione a ello. Otro ejemplo es que podemos hacer que el agente monitoree el flujo de video, y cuando hay cambios en el flujo de video, el agente puede reportar los cambios.

    - [Herramientas de Streaming](streaming-tools.md)

-   :material-console-line: **Artículo del blog: Google ADK + Vertex AI Live API**

    ---

    Este artículo muestra cómo usar Bidi-streaming (en vivo) en ADK para streaming de audio/video en tiempo real. Ofrece un ejemplo de servidor Python usando LiveRequestQueue para construir agentes de IA personalizados e interactivos.

    - [Artículo del blog: Google ADK + Vertex AI Live API](https://medium.com/google-cloud/google-adk-vertex-ai-live-api-125238982d5e)

-   :material-console-line: **Artículo del blog: Potencia el Desarrollo de ADK con Claude Code Skills**

    ---

    Este artículo demuestra cómo usar Claude Code Skills para acelerar el desarrollo de ADK, con un ejemplo de construcción de una aplicación de chat con Bidi-streaming. Aprende cómo aprovechar la asistencia de codificación impulsada por IA para construir mejores agentes más rápido.

    - [Artículo del blog: Potencia el Desarrollo de ADK con Claude Code Skills](https://medium.com/@kazunori279/supercharge-adk-development-with-claude-code-skills-d192481cbe72)

</div>