# Parte 1: Introducción a ADK Bidi-streaming

El Kit de Desarrollo de Agentes ([ADK](https://google.github.io/adk-docs/)) de Google proporciona un framework listo para producción para construir aplicaciones Bidi-streaming con modelos Gemini. Esta guía introduce la arquitectura de streaming de ADK, que permite comunicación bidireccional en tiempo real entre usuarios y agentes de IA a través de canales multimodales (texto, audio, video).

**Lo que aprenderás**: Esta parte cubre los fundamentos de Bidi-streaming, la tecnología subyacente de Live API (Gemini Live API y Vertex AI Live API), los componentes arquitectónicos de ADK (`LiveRequestQueue`, `Runner`, `Agent`), y un ejemplo completo de implementación con FastAPI. Comprenderás cómo ADK maneja la gestión de sesiones, orquestación de herramientas y abstracción de plataformas—reduciendo meses de desarrollo de infraestructura a configuración declarativa.

## Demo de ADK Bidi-streaming

Para ayudarte a entender los conceptos en esta guía, proporcionamos una aplicación demo funcional que muestra ADK bidirectional streaming en acción. Esta demo basada en FastAPI implementa el ciclo de vida completo de streaming con una arquitectura práctica del mundo real.

**Repositorio de Demo**: [adk-samples/python/agents/bidi-demo](https://github.com/google/adk-samples/tree/main/python/agents/bidi-demo)

![ADK Bidi-streaming Demo](assets/bidi-demo-screen.png)

La demo incluye:

- **Comunicación WebSocket**: Streaming bidireccional en tiempo real con tareas upstream/downstream concurrentes
- **Solicitudes Multimodales**: Entrada de texto, audio e imagen/video con transcripción automática
- **Respuestas Flexibles**: Salida de texto o audio según las capacidades del modelo
- **UI Interactiva**: Interfaz web con consola de eventos para monitorear eventos de Live API
- **Integración con Google Search**: Agente equipado con capacidades de llamada a herramientas

**Recomendamos encarecidamente instalar y ejecutar esta demo** antes de sumergirse en la guía. La experimentación práctica te ayudará a entender los conceptos más profundamente, y el código de la demo sirve como referencia práctica a lo largo de todas las partes de esta guía.

Para instrucciones de instalación y detalles de uso, consulta el [README de la demo](https://github.com/google/adk-samples/tree/main/python/agents/bidi-demo).

## 1.1 ¿Qué es Bidi-streaming?

Bidi-streaming (Streaming bidireccional) representa un cambio fundamental respecto a las interacciones tradicionales de IA. En lugar del patrón rígido de "preguntar y esperar", permite **comunicación bidireccional en tiempo real** donde tanto humanos como IA pueden hablar, escuchar y responder simultáneamente. Esto crea conversaciones naturales, similares a las humanas, con respuestas inmediatas y la capacidad revolucionaria de interrumpir interacciones en curso.

Piensa en la diferencia entre enviar correos electrónicos y tener una conversación telefónica. Las interacciones tradicionales de IA son como correos electrónicos—envías un mensaje completo, esperas una respuesta completa, luego envías otro mensaje completo. Bidi-streaming es como una conversación telefónica—fluida, natural, con la capacidad de interrumpir, aclarar y responder en tiempo real.

### Características Clave

Estas características distinguen Bidi-streaming de las interacciones tradicionales de IA y lo hacen excepcionalmente poderoso para crear experiencias de usuario atractivas:

- **Comunicación Bidireccional**: Intercambio continuo de datos sin esperar respuestas completas. Los usuarios pueden interrumpir a la IA a mitad de respuesta con nueva entrada, creando un flujo conversacional natural. La IA responde después de detectar que el usuario ha terminado de hablar (mediante detección automática de actividad de voz o señales de actividad explícitas).

- **Interrupción Responsiva**: Quizás la característica más importante para la experiencia natural del usuario—los usuarios pueden interrumpir al agente a mitad de respuesta con nueva entrada, justo como en una conversación humana. Si una IA está explicando física cuántica y de repente preguntas "espera, ¿qué es un electrón?", la IA se detiene inmediatamente y aborda tu pregunta.

- **Mejor para Multimodal**: Bidi-streaming sobresale en interacciones multimodales porque puede procesar diferentes tipos de entrada simultáneamente a través de una única conexión. Los usuarios pueden hablar mientras muestran documentos, escribir preguntas de seguimiento durante llamadas de voz, o cambiar sin problemas entre modos de comunicación sin perder contexto. Este enfoque unificado elimina la complejidad de gestionar canales separados para cada modalidad.

```mermaid
sequenceDiagram
    participant Client as Usuario
    participant Agent as Agente

    Client->>Agent: "¡Hola!"
    Client->>Agent: "Explica la historia de Japón"
    Agent->>Client: "¡Hola!"
    Agent->>Client: "¡Claro! La historia de Japón es..." (contenido parcial)
    Client->>Agent: "Ah, espera."

    Agent->>Client: "OK, ¿cómo puedo ayudar?" [interrupted: true]
```

### Diferencia con Otros Tipos de Streaming

Entender cómo Bidi-streaming difiere de otros enfoques es crucial para apreciar su valor único. El panorama de streaming incluye varios patrones distintos, cada uno sirviendo diferentes casos de uso:

!!! note "Comparación de Tipos de Streaming"

    **Bidi-streaming** difiere fundamentalmente de otros enfoques de streaming:

    - **Streaming del Lado del Servidor**: Flujo de datos unidireccional del servidor al cliente. Como ver una transmisión de video en vivo—recibes datos continuos pero no puedes interactuar con ellos en tiempo real. Útil para dashboards o feeds en vivo, pero no para conversaciones.

    - **Streaming a Nivel de Token**: Entrega secuencial de tokens de texto sin interrupción. La IA genera respuesta palabra por palabra, pero debes esperar a que se complete antes de enviar nueva entrada. Como ver a alguien escribir un mensaje en tiempo real—lo ves formándose, pero no puedes interrumpir.

    - **Bidi-streaming**: Comunicación bidireccional completa con soporte de interrupción. IA conversacional verdadera donde ambas partes pueden hablar, escuchar y responder simultáneamente. Esto es lo que permite el diálogo natural donde puedes interrumpir, aclarar o cambiar de tema a mitad de conversación.

### Aplicaciones del Mundo Real

Bidi-streaming revoluciona las aplicaciones de IA agéntica al permitir que los agentes operen con capacidad de respuesta e inteligencia similares a las humanas. Estas aplicaciones muestran cómo el streaming transforma interacciones estáticas de IA en experiencias dinámicas impulsadas por agentes que se sienten genuinamente inteligentes y proactivas.

En un video de la [demo de Shopper's Concierge](https://www.youtube.com/watch?v=LwHPYyw7u6U), la característica multimodal de Bidi-streaming mejora significativamente la experiencia del usuario de comercio electrónico al habilitar una experiencia de compra más rápida e intuitiva. La combinación de comprensión conversacional y búsqueda rápida y paralelizada culmina en capacidades avanzadas como prueba virtual, aumentando la confianza del comprador y reduciendo la fricción de las compras en línea.

<div class="video-grid">
  <div class="video-item">
    <div class="video-container">
      <iframe src="https://www.youtube-nocookie.com/embed/LwHPYyw7u6U?si=xxIEhnKBapzQA6VV" title="Shopper's Concierge" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
    </div>
  </div>
</div>

Además, hay muchas aplicaciones posibles del mundo real para Bidi-streaming:

#### Servicio al Cliente y Centros de Contacto

Esta es la aplicación más directa. La tecnología puede crear agentes virtuales sofisticados que van mucho más allá de los chatbots tradicionales.

- Caso de uso: Un cliente llama a la línea de soporte de una empresa minorista sobre un producto defectuoso.
- Multimodalidad (video): El cliente puede decir, "Mi cafetera está goteando por abajo, déjame mostrarte." Luego pueden usar la cámara de su teléfono para transmitir video en vivo del problema. El agente de IA puede usar sus capacidades de visión para identificar el modelo y el punto específico de falla.
- Interacción en Vivo e Interrupción: Si el agente dice, "Bien, estoy procesando una devolución para tu cafetera Modelo X," el cliente puede interrumpir con, "No, espera, es la Modelo Y Pro," y el agente puede corregir inmediatamente su curso sin reiniciar la conversación.

#### Comercio Electrónico y Compras Personalizadas

El agente puede actuar como un comprador personal en vivo e interactivo, mejorando la experiencia de retail en línea.

- Caso de Uso: Un usuario está navegando un sitio web de moda y quiere consejos de estilo.
- Multimodalidad (Voz e Imagen): El usuario puede sostener una prenda de vestir frente a su webcam y preguntar, "¿Puedes encontrarme un par de zapatos que combinen bien con estos pantalones?" El agente analiza el color y estilo de los pantalones.
- Interacción en Vivo: La conversación puede ser un ir y venir fluido: "Muéstrame algo más casual." ... "Bien, ¿qué tal estos tenis?" ... "Perfecto, agrega los azules en talla 10 a mi carrito."

#### Servicio de Campo y Asistencia Técnica

Los técnicos que trabajan en campo pueden usar un asistente activado por voz y manos libres para obtener ayuda en tiempo real.

- Caso de Uso: Un técnico de HVAC está en sitio tratando de diagnosticar una unidad de aire acondicionado comercial compleja.
- Multimodalidad (Video y Voz): El técnico, usando gafas inteligentes o usando un teléfono, puede transmitir su punto de vista al agente de IA. Pueden preguntar, "Estoy escuchando un ruido extraño de este compresor. ¿Puedes identificarlo y mostrar el diagrama de flujo de diagnóstico para este modelo?"
- Interacción en Vivo: El agente puede guiar al técnico paso a paso, y el técnico puede hacer preguntas aclaratorias o interrumpir en cualquier punto sin quitar las manos de sus herramientas.

#### Salud y Telemedicina

El agente puede servir como primer punto de contacto para la admisión de pacientes, triaje y consultas básicas.

- Caso de Uso: Un paciente usa la app de un proveedor para una consulta preliminar sobre una condición de la piel.
- Multimodalidad (Video/Imagen): El paciente puede compartir de forma segura un video en vivo o imagen de alta resolución de una erupción. La IA puede realizar un análisis preliminar y hacer preguntas aclaratorias.

#### Servicios Financieros y Gestión de Patrimonio

Un agente puede proporcionar a los clientes una forma segura, interactiva y rica en datos para gestionar sus finanzas.

- Caso de Uso: Un cliente quiere revisar su portafolio de inversión y discutir tendencias del mercado.
- Multimodalidad (Compartir Pantalla): El agente puede compartir su pantalla para mostrar gráficos, tablas y rendimiento del portafolio. El cliente también podría compartir su pantalla para señalar un artículo de noticias específico y preguntar, "¿Cuál es el impacto potencial de este evento en mis acciones de tecnología?"
- Interacción en Vivo: Analizar la asignación actual del portafolio del cliente accediendo a los datos de su cuenta. Simular el impacto de una operación potencial en el perfil de riesgo del portafolio.

## 1.2 Gemini Live API y Vertex AI Live API

Las capacidades de Bidi-streaming de ADK están impulsadas por la tecnología Live API, disponible a través de dos plataformas: **[Gemini Live API](https://ai.google.dev/gemini-api/docs/live)** (vía Google AI Studio) y **[Vertex AI Live API](https://cloud.google.com/vertex-ai/generative-ai/docs/live-api)** (vía Google Cloud). Ambas proporcionan conversaciones de streaming en tiempo real y baja latencia con modelos Gemini, pero sirven diferentes necesidades de desarrollo y despliegue.

A lo largo de esta guía, usamos **"Live API"** para referirnos a ambas plataformas colectivamente, especificando "Gemini Live API" o "Vertex AI Live API" solo al discutir características o diferencias específicas de la plataforma.

### ¿Qué es Live API?

Live API es la tecnología de IA conversacional en tiempo real de Google que habilita **Bidi-streaming de baja latencia** con modelos Gemini. A diferencia de las APIs tradicionales de solicitud-respuesta, Live API establece conexiones WebSocket persistentes que soportan:

**Capacidades Principales:**

- **Streaming multimodal**: Procesa flujos continuos de audio, video y texto en tiempo real
- **Detección de Actividad de Voz (VAD)**: Detecta automáticamente cuándo los usuarios terminan de hablar, habilitando cambios de turno naturales sin señales explícitas. La IA sabe cuándo comenzar a responder y cuándo esperar más entrada
- **Respuestas inmediatas**: Entrega respuestas habladas o de texto similares a las humanas con latencia mínima
- **Interrupción inteligente**: Permite a los usuarios interrumpir a la IA a mitad de respuesta, justo como conversaciones humanas
- **Transcripción de Audio**: Transcripción en tiempo real tanto de entrada del usuario como de salida del modelo, habilitando características de accesibilidad y registro de conversación sin servicios de transcripción separados
- **Gestión de Sesiones**: Conversaciones largas pueden abarcar múltiples conexiones a través de reanudación de sesión, con la API preservando historial completo de conversación y contexto a través de reconexiones
- **Integración de Herramientas**: La llamada a funciones funciona sin problemas en modo streaming, con herramientas ejecutándose en segundo plano mientras la conversación continúa

**Características Nativas del Modelo de Audio:**

- **Audio Proactivo**: El modelo puede iniciar respuestas basándose en conciencia del contexto, creando interacciones más naturales donde la IA ofrece ayuda o aclaración proactivamente (solo modelos de Audio Nativo)
- **Diálogo Afectivo**: Los modelos avanzados entienden el tono de voz y contexto emocional, adaptando respuestas para coincidir con el estado de ánimo conversacional y el sentimiento del usuario (solo modelos de Audio Nativo)

!!! note "Aprende Más"

    Para información detallada sobre modelos de Audio Nativo y estas características, consulta [Parte 5: Audio y Video - Proactividad y Diálogo Afectivo](part5.md#proactivity-and-affective-dialog).

**Especificaciones Técnicas:**

- **Entrada de audio**: PCM de 16 bits a 16kHz (mono)
- **Salida de audio**: PCM de 16 bits a 24kHz (modelos de audio nativo)
- **Entrada de video**: 1 fotograma por segundo, resolución recomendada 768x768
- **Ventanas de contexto**: Varía según el modelo (típicamente 32k-128k tokens para modelos Live API). Consulta [modelos Gemini](https://ai.google.dev/gemini-api/docs/models/gemini) para límites específicos.
- **Idiomas**: Más de 24 idiomas soportados con detección automática

### Gemini Live API vs Vertex AI Live API

Ambas APIs proporcionan la misma tecnología central Live API, pero difieren en plataforma de despliegue, autenticación y características empresariales:

| **Aspecto** | **Gemini Live API** | **Vertex AI Live API** |
|--------|-----------------|-------------------|
| **Acceso** | Google AI Studio | Google Cloud |
| **Autenticación** | Clave API (`GOOGLE_API_KEY`) | Credenciales de Google Cloud (`GOOGLE_CLOUD_PROJECT`, `GOOGLE_CLOUD_LOCATION`) |
| **Mejor para** | Prototipado rápido, desarrollo, experimentación | Despliegues en producción, aplicaciones empresariales |
| **Duración de Sesión** | Solo audio: 15 min<br>Audio+video: 2 min<br>Con [Parte 4: Compresión de Ventana de Contexto](part4.md#live-api-context-window-compression): Ilimitado | Ambos: 10 min<br>Con [Parte 4: Compresión de Ventana de Contexto](part4.md#live-api-context-window-compression): Ilimitado |
| **Sesiones Concurrentes** | Cuotas basadas en niveles (ver [cuotas API](https://ai.google.dev/gemini-api/docs/quota)) | Hasta 1,000 por proyecto (configurable vía solicitudes de cuota) |
| **Características Empresariales** | Básicas | Monitoreo avanzado, registro, SLAs, reanudación de sesión (24h) |
| **Complejidad de Configuración** | Mínima (solo clave API) | Requiere configuración de proyecto Google Cloud |
| **Versión API** | `v1beta` | `v1beta1` |
| **Endpoint API** | `generativelanguage.googleapis.com` | `{location}-aiplatform.googleapis.com` |
| **Facturación** | Uso rastreado vía clave API | Facturación de proyecto Google Cloud |

!!! note "Notas de Referencia de Live API"

    **Límites de sesiones concurrentes**: Basados en cuota y pueden variar según el nivel de cuenta o configuración. Verifica tus cuotas actuales en Google AI Studio o Google Cloud Console.

    **Documentación Oficial**: [Guía Gemini Live API](https://ai.google.dev/gemini-api/docs/live-guide) | [Descripción General Vertex AI Live API](https://cloud.google.com/vertex-ai/generative-ai/docs/live-api)

## 1.3 ADK Bidi-streaming: Para Construir Aplicaciones de Agentes en Tiempo Real

Construir aplicaciones de Agentes en tiempo real desde cero presenta desafíos de ingeniería significativos. Mientras Live API proporciona la tecnología de streaming subyacente, integrarla en aplicaciones de producción requiere resolver problemas complejos: gestionar conexiones WebSocket y lógica de reconexión, orquestar ejecución de herramientas y manejo de respuestas, persistir estado de conversación a través de sesiones, coordinar flujos de datos concurrentes para entradas multimodales, y manejar diferencias de plataforma entre entornos de desarrollo y producción.

ADK transforma estos desafíos en APIs simples y declarativas. En lugar de pasar meses construyendo infraestructura para gestión de sesiones, orquestación de herramientas y persistencia de estado, los desarrolladores pueden enfocarse en definir comportamiento del agente y crear experiencias de usuario. Esta sección explora lo que ADK maneja automáticamente y por qué es el camino recomendado para construir aplicaciones de streaming listas para producción.

**Live API Cruda vs. ADK Bidi-streaming:**

| Característica | Live API Cruda (SDK `google-genai`) | ADK Bidi-streaming (SDK `adk-python` y `adk-java`) |
|---------|-----------------------------------|------------------------------------------------------|
| **Framework de Agentes** | ❌ No disponible | ✅ Agente único, multi-agente con sub-agentes, y agentes de flujo de trabajo secuencial, Ecosistema de herramientas, Listo para despliegue, Evaluación, Seguridad y más (ver [docs de Agente ADK](https://google.github.io/adk-docs/agents/)) |
| **Ejecución de Herramientas** | ❌ Ejecución manual de herramientas y manejo de respuestas | ✅ Ejecución automática de herramientas (ver [Parte 3: Eventos de Llamada a Herramientas](part3.md#tool-call-events)) |
| **Gestión de Conexión** | ❌ Reconexión manual y reanudación de sesión | ✅ Reconexión automática y reanudación de sesión (ver [Parte 4: Reanudación de Sesión Live API](part4.md#live-api-session-resumption)) |
| **Modelo de Eventos** | ❌ Estructuras de eventos personalizadas y serialización | ✅ Modelo de eventos unificado con metadatos (ver [Parte 3: Manejo de Eventos](part3.md)) |
| **Framework de Procesamiento de Eventos Async** | ❌ Coordinación async manual y manejo de streams | ✅ `LiveRequestQueue`, generador async `run_live()`, coordinación automática de flujo bidireccional (ver [Parte 2](part2.md) y [Parte 3](part3.md)) |
| **Persistencia de Sesión a Nivel de Aplicación** | ❌ Implementación manual | ✅ Bases de datos SQL (PostgreSQL, MySQL, SQLite), Vertex AI, en memoria (ver [docs de Sesión ADK](https://google.github.io/adk-docs/sessions/)) |

### Flexibilidad de Plataforma

Una de las características más poderosas de ADK es su soporte transparente para tanto [Gemini Live API](https://ai.google.dev/gemini-api/docs/live) como [Vertex AI Live API](https://cloud.google.com/vertex-ai/generative-ai/docs/live-api). Esta flexibilidad de plataforma habilita un flujo de trabajo desarrollo-a-producción sin costuras: desarrolla localmente con Gemini API usando claves API gratuitas, luego despliega a producción con Vertex AI usando infraestructura empresarial de Google Cloud—todo **sin cambiar código de aplicación**, solo configuración de entorno.

#### Cómo Funciona la Selección de Plataforma

ADK usa la variable de entorno `GOOGLE_GENAI_USE_VERTEXAI` para determinar qué plataforma Live API usar:

- `GOOGLE_GENAI_USE_VERTEXAI=FALSE` (o no configurada): Usa Gemini Live API vía Google AI Studio
- `GOOGLE_GENAI_USE_VERTEXAI=TRUE`: Usa Vertex AI Live API vía Google Cloud

Esta variable de entorno es leída por el SDK subyacente `google-genai` cuando ADK crea la conexión LLM. No se necesitan cambios de código al cambiar plataformas—solo cambios de configuración de entorno.

##### Fase de Desarrollo: Gemini Live API (Google AI Studio)

```bash
# .env.development
GOOGLE_GENAI_USE_VERTEXAI=FALSE
GOOGLE_API_KEY=your_api_key_here
```

**Beneficios:**

- Prototipado rápido con claves API gratuitas de Google AI Studio
- No se requiere configuración de Google Cloud
- Experimentación instantánea con características de streaming
- Cero costos de infraestructura durante desarrollo

##### Fase de Producción: Vertex AI Live API (Google Cloud)

```bash
# .env.production
GOOGLE_GENAI_USE_VERTEXAI=TRUE
GOOGLE_CLOUD_PROJECT=your_project_id
GOOGLE_CLOUD_LOCATION=us-central1
```

**Beneficios:**

- Infraestructura de grado empresarial vía Google Cloud
- Monitoreo avanzado, registro y controles de costos
- Integración con servicios existentes de Google Cloud
- SLAs de producción y soporte
- **No se requieren cambios de código** - solo configuración de entorno

Al manejar la complejidad de gestión de sesiones, orquestación de herramientas, persistencia de estado y diferencias de plataforma, ADK te permite enfocarte en construir experiencias de agentes inteligentes en lugar de luchar con infraestructura de streaming. El mismo código funciona sin problemas a través de entornos de desarrollo y producción, dándote todo el poder de Bidi-streaming sin la carga de implementación.

## 1.4 Descripción General de la Arquitectura ADK Bidi-streaming

Ahora que entiendes la tecnología Live API y por qué ADK agrega valor, exploremos cómo ADK realmente funciona. Esta sección mapea el flujo completo de datos desde tu aplicación a través del pipeline de ADK hacia Live API y de vuelta, mostrando qué componentes manejan qué responsabilidades.

Verás cómo componentes clave como `LiveRequestQueue`, `Runner` y `Agent` orquestan conversaciones de streaming sin requerir que gestiones conexiones WebSocket, coordines flujos async o manejes diferencias API específicas de la plataforma.

### Arquitectura de Alto Nivel

```mermaid
graph TB
    subgraph "Application"
        subgraph "Client"
            C1["Web / Mobile"]
        end

        subgraph "Transport Layer"
            T1["WebSocket / SSE (e.g. FastAPI)"]
        end
    end

    subgraph "ADK"
        subgraph "ADK Bidi-streaming"
            L1[LiveRequestQueue]
            L2[Runner]
            L3[Agent]
            L4[LLM Flow]
        end

        subgraph "LLM Integration"
            G1[GeminiLlmConnection]
            G2[Gemini Live API / Vertex AI Live API]
        end
    end

    C1 <--> T1
    T1 -->|"live_request_queue.send()"| L1
    L1 -->|"runner.run_live(queue)"| L2
    L2 -->|"agent.run_live()"| L3
    L3 -->|"_llm_flow.run_live()"| L4
    L4 -->|"llm.connect()"| G1
    G1 <--> G2
    G1 -->|"yield LlmResponse"| L4
    L4 -->|"yield Event"| L3
    L3 -->|"yield Event"| L2
    L2 -->|"yield Event"| T1

    classDef external fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef adk fill:#f3e5f5,stroke:#4a148c,stroke-width:2px

    class C1,T1 external
    class L1,L2,L3,L4,G1,G2 adk
```

| El desarrollador proporciona: | ADK proporciona: | Live API proporciona: |
|---------------------|---------------|------------------|
| **Web / Mobile**: Aplicaciones frontend con las que los usuarios interactúan, manejando UI/UX, captura de entrada del usuario y visualización de respuestas<br><br>**Servidor [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket) / [SSE](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)**: Servidor de comunicación en tiempo real (como [FastAPI](https://fastapi.tiangolo.com/)) que gestiona conexiones de clientes, maneja protocolos de streaming y enruta mensajes entre clientes y ADK<br><br>**`Agent`**: Definición de agente de IA personalizado con instrucciones, herramientas y comportamiento específicos adaptados a las necesidades de tu aplicación | **[LiveRequestQueue](https://github.com/google/adk-python/blob/29c1115959b0084ac1169748863b35323da3cf50/src/google/adk/agents/live_request_queue.py)**: Cola de mensajes que almacena en búfer y secuencia mensajes entrantes del usuario (contenido de texto, blobs de audio, señales de control) para procesamiento ordenado por el agente<br><br>**[Runner](https://github.com/google/adk-python/blob/29c1115959b0084ac1169748863b35323da3cf50/src/google/adk/runners.py)**: Motor de ejecución que orquesta sesiones de agente, gestiona estado de conversación y proporciona la interfaz de streaming `run_live()`<br><br>**[RunConfig](https://github.com/google/adk-python/blob/29c1115959b0084ac1169748863b35323da3cf50/src/google/adk/agents/run_config.py)**: Configuración para comportamiento de streaming, modalidades y características avanzadas<br><br>**Componentes internos** (gestionados automáticamente, no usados directamente por desarrolladores): [LLM Flow](https://github.com/google/adk-python/blob/29c1115959b0084ac1169748863b35323da3cf50/src/google/adk/flows/llm_flows/base_llm_flow.py) para pipeline de procesamiento y [GeminiLlmConnection](https://github.com/google/adk-python/blob/29c1115959b0084ac1169748863b35323da3cf50/src/google/adk/models/gemini_llm_connection.py) para traducción de protocolo | **[Gemini Live API](https://ai.google.dev/gemini-api/docs/live)** (vía Google AI Studio) y **[Vertex AI Live API](https://cloud.google.com/vertex-ai/generative-ai/docs/live-api)** (vía Google Cloud): Servicios de modelo de lenguaje en tiempo real de Google que procesan entrada de streaming, generan respuestas, manejan interrupciones, soportan contenido multimodal (texto, audio, video) y proporcionan capacidades avanzadas de IA como llamada a funciones y comprensión contextual |

Esta arquitectura demuestra la clara separación de responsabilidades de ADK: tu aplicación maneja interacción del usuario y protocolos de transporte, ADK gestiona la orquestación de streaming y estado, y Live API proporcionan la inteligencia de IA. Al abstraer la complejidad de gestión de conexión de streaming del lado del LLM, bucles de eventos y traducción de protocolo, ADK te permite enfocarte en construir comportamiento del agente y experiencias de usuario en lugar de infraestructura de streaming.

## 1.5 Ciclo de Vida de Aplicación ADK Bidi-streaming

ADK Bidi-streaming integra sesión Live API en el ciclo de vida de aplicación del framework ADK. Esta integración crea un ciclo de vida de cuatro fases que combina gestión de agentes de ADK con capacidades de streaming en tiempo real de Live API:

- **Fase 1: Inicialización de Aplicación** (Una Vez al Inicio)
  - Inicialización de Aplicación ADK
    - Crear un [Agent](https://google.github.io/adk-docs/agents/): para interactuar con usuarios, utilizar herramientas externas y coordinar con otros agentes.
    - Crear un [SessionService](https://google.github.io/adk-docs/sessions/session/#managing-sessions-with-a-sessionservice): para obtener o crear `Session` de ADK
    - Crear un [Runner](https://google.github.io/adk-docs/runtime/): para proporcionar un runtime para el Agent

- **Fase 2: Inicialización de Sesión** (Una Vez por Sesión de Usuario)
  - Inicialización de `Session` de ADK:
    - Obtener o Crear una `Session` de ADK usando el `SessionService`
  - Inicialización de ADK Bidi-streaming:
    - Crear un [RunConfig](part4.md) para configurar ADK Bidi-streaming
    - Crear un [LiveRequestQueue](part2.md) para enviar mensajes de usuario al `Agent`
    - Iniciar un bucle de eventos [run_live()](part3.md)

- **Fase 3: Bidi-streaming con bucle de eventos `run_live()`** (Una o Más Veces por Sesión de Usuario)
  - Upstream: Usuario envía mensaje al agente con `LiveRequestQueue`
  - Downstream: Agente responde al usuario con `Event`

- **Fase 4: Terminar sesión Live API** (Una o Más Veces por Sesión de Usuario)
  - `LiveRequestQueue.close()`

**Descripción General del Flujo del Ciclo de Vida:**

```mermaid
graph TD
    A[Fase 1: Init Aplicación<br/>Una Vez al Inicio] --> B[Fase 2: Init Sesión<br/>Por Conexión de Usuario]
    B --> C[Fase 3: Bidi-streaming<br/>Comunicación Activa]
    C --> D[Fase 4: Terminar<br/>Cerrar Sesión]
    D -.Nueva Conexión.-> B

    style A fill:#e3f2fd
    style B fill:#e8f5e9
    style C fill:#fff3e0
    style D fill:#ffebee
```

Este diagrama de flujo muestra las fases del ciclo de vida de alto nivel y cómo se conectan. El diagrama de secuencia detallado a continuación ilustra los componentes específicos e interacciones dentro de cada fase.

```mermaid
sequenceDiagram
    participant Client as Cliente
    participant App as Servidor de Aplicación
    participant Queue as LiveRequestQueue
    participant Runner
    participant Agent as Agente
    participant API as Live API

    rect rgb(230, 240, 255)
        Note over App: Fase 1: Inicialización de Aplicación (Una Vez al Inicio)
        App->>Agent: 1. Crear Agent(model, tools, instruction)
        App->>App: 2. Crear SessionService()
        App->>Runner: 3. Crear Runner(app_name, agent, session_service)
    end

    rect rgb(240, 255, 240)
        Note over Client,API: Fase 2: Inicialización de Sesión (Cada Vez que un Usuario se Conecta)
        Client->>App: 1. WebSocket connect(user_id, session_id)
        App->>App: 2. get_or_create_session(app_name, user_id, session_id)
        App->>App: 3. Crear RunConfig(streaming_mode, modalities)
        App->>Queue: 4. Crear LiveRequestQueue()
        App->>Runner: 5. Iniciar run_live(user_id, session_id, queue, config)
        Runner->>API: Conectar a sesión Live API
    end

    rect rgb(255, 250, 240)
        Note over Client,API: Fase 3: Bidi-streaming con Bucle de Eventos run_live()

        par Upstream: Usuario envía mensajes vía LiveRequestQueue
            Client->>App: Mensaje de usuario (text/audio/video)
            App->>Queue: send_content() / send_realtime()
            Queue->>Runner: Solicitud en búfer
            Runner->>Agent: Procesar solicitud
            Agent->>API: Stream a Live API
        and Downstream: Agente responde vía Events
            API->>Agent: Respuesta streaming
            Agent->>Runner: Procesar respuesta
            Runner->>App: yield Event (text/audio/tool/turn)
            App->>Client: Reenviar Event vía WebSocket
        end

        Note over Client,API: (Bucle de eventos continúa hasta señal de cierre)
    end

    rect rgb(255, 240, 240)
        Note over Client,API: Fase 4: Terminar sesión Live API
        Client->>App: WebSocket disconnect
        App->>Queue: close()
        Queue->>Runner: Señal de cierre
        Runner->>API: Desconectar de Live API
        Runner->>App: run_live() sale
    end
```

En las siguientes secciones, verás cada fase detallada, mostrando exactamente cuándo crear cada componente y cómo funcionan juntos. Entender este patrón de ciclo de vida es esencial para construir aplicaciones de streaming robustas que puedan manejar múltiples sesiones concurrentes eficientemente.


### Fase 1: Inicialización de Aplicación

Estos componentes se crean una vez cuando tu aplicación inicia y se comparten a través de todas las sesiones de streaming. Definen las capacidades de tu agente, gestionan historial de conversación y orquestan la ejecución de streaming.

#### Define Tu Agente

El `Agent` es el núcleo de tu aplicación de streaming—define qué puede hacer tu IA, cómo debe comportarse y qué modelo de IA lo impulsa. Configuras tu agente con un modelo específico, herramientas que puede usar (como Google Search o APIs personalizadas), e instrucciones que moldean su personalidad y comportamiento.

```python title='Implementación de Demo: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/google_search_agent/agent.py#L10-L15" target="_blank">agent.py:10-15</a>'
"""Definición del Agente de Google Search para demo ADK Bidi-streaming."""

import os
from google.adk.agents import Agent
from google.adk.tools import google_search

# Modelos predeterminados para Live API con soporte de audio nativo:
# - Gemini Live API: gemini-2.5-flash-native-audio-preview-12-2025
# - Vertex AI Live API: gemini-live-2.5-flash-native-audio
agent = Agent(
    name="google_search_agent",
    model=os.getenv("DEMO_AGENT_MODEL", "gemini-2.5-flash-native-audio-preview-12-2025"),
    tools=[google_search],
    instruction="You are a helpful assistant that can search the web."
)
```

La instancia del agente es **sin estado y reutilizable**—la creas una vez y la usas para todas las sesiones de streaming. La configuración del agente se cubre en la [documentación de Agent ADK](https://google.github.io/adk-docs/agents/).

!!! note "Disponibilidad de Modelos"

    Para los últimos modelos soportados y sus capacidades, consulta [Parte 5: Entendiendo Arquitecturas de Modelos de Audio](part5.md#understanding-audio-model-architectures).

!!! note "Agent vs LlmAgent"

    `Agent` es la abreviatura recomendada para `LlmAgent` (ambos se importan de `google.adk.agents`). Son idénticos - usa el que prefieras. Esta guía usa `Agent` por brevedad, pero puedes ver `LlmAgent` en otra documentación y ejemplos de ADK.

#### Define Tu SessionService

La [Session](https://google.github.io/adk-docs/sessions/session/) de ADK gestiona estado e historial de conversación a través de sesiones de streaming. Almacena y recupera datos de sesión, habilitando características como reanudación de conversación y persistencia de contexto.

Para crear una `Session`, u obtener una existente para un `session_id` especificado, cada aplicación ADK necesita tener un [SessionService](https://google.github.io/adk-docs/sessions/session/#managing-sessions-with-a-sessionservice). Para propósito de desarrollo, ADK proporciona un `InMemorySessionService` simple que perderá el estado de la `Session` cuando la aplicación se apague.

```python title='Implementación de Demo: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py#L37" target="_blank">main.py:37</a>'
from google.adk.sessions import InMemorySessionService

# Define tu servicio de sesión
session_service = InMemorySessionService()
```

Para aplicaciones de producción, elige un servicio de sesión persistente basado en tu infraestructura:

**Usa `DatabaseSessionService` si:**

- Necesitas almacenamiento persistente con SQLite, PostgreSQL o MySQL
- Estás construyendo apps de un solo servidor (SQLite) o despliegues multi-servidor (PostgreSQL/MySQL)
- Quieres control total sobre almacenamiento de datos y respaldos
- Ejemplos:
    - SQLite: `DatabaseSessionService(db_url="sqlite:///./sessions.db")`
    - PostgreSQL: `DatabaseSessionService(db_url="postgresql://user:pass@host/db")`

**Usa `VertexAiSessionService` si:**

- Ya estás usando Google Cloud Platform
- Quieres almacenamiento gestionado con escalabilidad integrada
- Necesitas integración estrecha con características de Vertex AI
- Ejemplo: `VertexAiSessionService(project="my-project")`

Ambos proporcionan capacidades de persistencia de sesión—elige según tus requisitos de infraestructura y escala. Con servicios de sesión persistentes, el estado de la `Session` se preservará incluso después del apagado de la aplicación. Consulta la [documentación de Gestión de Sesiones ADK](https://google.github.io/adk-docs/sessions/) para más detalles.

#### Define Tu Runner

El [Runner](https://google.github.io/adk-docs/runtime/) proporciona el runtime para el `Agent`. Gestiona el flujo de conversación, coordina ejecución de herramientas, maneja eventos e integra con almacenamiento de sesiones. Creas una instancia de runner al inicio de la aplicación y la reutilizas para todas las sesiones de streaming.

```python title='Implementación de Demo: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py#L50" target="_blank">main.py:50,53</a>'
from google.adk.runners import Runner

APP_NAME = "bidi-demo"

# Define tu runner
runner = Runner(
    app_name=APP_NAME,
    agent=agent,
    session_service=session_service
)
```

El parámetro `app_name` es requerido e identifica tu aplicación en almacenamiento de sesiones. Todas las sesiones para tu aplicación se organizan bajo este nombre.

### Fase 2: Inicialización de Sesión

#### Obtener o Crear Sesión

La `Session` de ADK proporciona un "hilo de conversación" de la aplicación Bidi-streaming. Así como no comenzarías cada mensaje de texto desde cero, los agentes necesitan contexto respecto a la interacción en curso. `Session` es el objeto ADK diseñado específicamente para rastrear y gestionar estos hilos de conversación individuales.

##### `Session` de ADK vs sesión Live API

La `Session` de ADK (gestionada por SessionService) proporciona **almacenamiento de conversación persistente** a través de múltiples sesiones Bidi-streaming (puede abarcar horas, días o incluso meses), mientras que la sesión Live API (gestionada por backend Live API) es **un contexto de streaming transitorio** que existe solo durante un único bucle de eventos Bidi-streaming (abarca minutos u horas típicamente) que discutiremos más adelante. Cuando el bucle inicia, ADK inicializa la sesión Live API con historial de la `Session` de ADK, luego actualiza la `Session` de ADK conforme ocurren nuevos eventos.

!!! note "Aprende Más"

    Para una comparación detallada con diagramas de secuencia, consulta [Parte 4: `Session` de ADK vs sesión Live API](part4.md#adk-session-vs-live-api-session).

##### Los Identificadores de Sesión Son Definidos por la Aplicación

Las sesiones se identifican por tres parámetros: `app_name`, `user_id` y `session_id`. Esta jerarquía de tres niveles habilita aplicaciones multi-tenant donde cada usuario puede tener múltiples sesiones concurrentes.

Tanto `user_id` como `session_id` son **identificadores de cadena arbitrarios** que defines según las necesidades de tu aplicación. ADK no realiza validación de formato más allá de `.strip()` en `session_id`—puedes usar cualquier valor de cadena que tenga sentido para tu aplicación:

- **Ejemplos de `user_id`**: UUIDs de usuario (`"550e8400-e29b-41d4-a716-446655440000"`), direcciones de email (`"alice@example.com"`), IDs de base de datos (`"user_12345"`), o identificadores simples (`"demo-user"`)
- **Ejemplos de `session_id`**: Tokens de sesión personalizados, UUIDs, IDs basados en timestamp (`"session_2025-01-27_143022"`), o identificadores simples (`"demo-session"`)

**Auto-generación**: Si pasas `session_id=None` o una cadena vacía a `create_session()`, ADK genera automáticamente un UUID para ti (ej., `"550e8400-e29b-41d4-a716-446655440000"`).

**Jerarquía organizacional**: Estos identificadores organizan sesiones en una estructura de tres niveles:

```text
app_name → user_id → session_id → Session
```

Este diseño habilita escenarios como:

- Aplicaciones multi-tenant donde diferentes usuarios tienen espacios de conversación aislados
- Usuarios individuales con múltiples hilos de chat concurrentes (ej., diferentes temas)
- Aislamiento de sesión por dispositivo o por navegador

##### Patrón Recomendado: Obtener-o-Crear

El patrón de producción recomendado es verificar si una sesión existe primero, luego crearla solo si es necesario. Este enfoque maneja de manera segura tanto sesiones nuevas como reanudación de conversación:

```python title='Implementación de Demo: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py#L155-L161" target="_blank">main.py:155-161</a>'
# Obtener o crear sesión (maneja tanto sesiones nuevas como reconexiones)
session = await session_service.get_session(
    app_name=APP_NAME,
    user_id=user_id,
    session_id=session_id
)
if not session:
    await session_service.create_session(
        app_name=APP_NAME,
        user_id=user_id,
        session_id=session_id
    )
```

Este patrón funciona correctamente en todos los escenarios:

- **Conversaciones nuevas**: Si la sesión no existe, se crea automáticamente
- **Reanudando conversaciones**: Si la sesión ya existe (ej., reconexión después de interrupción de red), la sesión existente se reutiliza con historial completo de conversación
- **Idempotente**: Seguro de llamar múltiples veces sin errores

**Importante**: La sesión debe existir antes de llamar `runner.run_live()` con los mismos identificadores. Si la sesión no existe, `run_live()` lanzará `ValueError: Session not found`.

#### Crear RunConfig

[RunConfig](part4.md) define el comportamiento de streaming para esta sesión específica—qué modalidades usar (texto o audio), si habilitar transcripción, detección de actividad de voz, proactividad y otras características avanzadas.

```python title='Implementación de Demo: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py#L110-L124" target="_blank">main.py:110-124</a>'
from google.adk.agents.run_config import RunConfig, StreamingMode
from google.genai import types

# Los modelos de audio nativo requieren modalidad de respuesta AUDIO con transcripción de audio
response_modalities = ["AUDIO"]
run_config = RunConfig(
    streaming_mode=StreamingMode.BIDI,
    response_modalities=response_modalities,
    input_audio_transcription=types.AudioTranscriptionConfig(),
    output_audio_transcription=types.AudioTranscriptionConfig(),
    session_resumption=types.SessionResumptionConfig()
)
```

`RunConfig` es **específico de sesión**—cada sesión de streaming puede tener diferente configuración. Por ejemplo, un usuario podría preferir respuestas solo de texto mientras otro usa modo de voz. Consulta [Parte 4: Entendiendo RunConfig](part4.md) para opciones completas de configuración.

#### Crear LiveRequestQueue

`LiveRequestQueue` es el canal de comunicación para enviar mensajes al agente durante streaming. Es una cola async segura para hilos que almacena en búfer mensajes de usuario (contenido de texto, blobs de audio, señales de actividad) para procesamiento ordenado.

```python title='Implementación de Demo: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py#L163" target="_blank">main.py:163</a>'
from google.adk.agents.live_request_queue import LiveRequestQueue

live_request_queue = LiveRequestQueue()
```

`LiveRequestQueue` es **específico de sesión y con estado**—creas una nueva cola para cada sesión de streaming y la cierras cuando la sesión termina. A diferencia de `Agent` y `Runner`, las colas no pueden reutilizarse a través de sesiones.

!!! warning "Una Cola Por Sesión"

    Nunca reutilices un `LiveRequestQueue` a través de múltiples sesiones de streaming. Cada llamada a `run_live()` requiere una cola nueva. Reutilizar colas puede causar problemas de orden de mensajes y corrupción de estado.

    La señal de cierre persiste en la cola (ver [`live_request_queue.py:59-60`](https://github.com/google/adk-python/blob/fd2c0f556b786417a9f6add744827b07e7a06b7d/src/google/adk/agents/live_request_queue.py#L66-L67)) y termina el bucle de envío (ver [`base_llm_flow.py:264-266`](https://github.com/google/adk-python/blob/fd2c0f556b786417a9f6add744827b07e7a06b7d/src/google/adk/flows/llm_flows/base_llm_flow.py#L260-L262)). Reutilizar una cola trasladaría esta señal y cualquier mensaje restante de la sesión anterior.

### Fase 3: Bidi-streaming con bucle de eventos `run_live()`

Una vez que el bucle de streaming está corriendo, puedes enviar mensajes al agente y recibir respuestas **concurrentemente**—esto es Bidi-streaming en acción. El agente puede estar generando una respuesta mientras estás enviando nueva entrada, habilitando conversación natural basada en interrupción.

#### Enviar Mensajes al Agente

Usa métodos de `LiveRequestQueue` para enviar diferentes tipos de mensajes al agente durante la sesión de streaming:

```python title='Implementación de Demo: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py#L169-L217" target="_blank">main.py:169-217</a>'
from google.genai import types

# Enviar contenido de texto
content = types.Content(parts=[types.Part(text=json_message["text"])])
live_request_queue.send_content(content)

# Enviar blob de audio
audio_blob = types.Blob(
    mime_type="audio/pcm;rate=16000",
    data=audio_data
)
live_request_queue.send_realtime(audio_blob)
```

Estos métodos son **no bloqueantes**—inmediatamente agregan mensajes a la cola sin esperar procesamiento. Esto habilita experiencias de usuario suaves y responsivas incluso durante procesamiento pesado de IA.

Consulta [Parte 2: Enviando mensajes con LiveRequestQueue](part2.md) para documentación detallada de API.

#### Recibir y Procesar Eventos

El generador async `run_live()` continuamente produce objetos `Event` conforme el agente procesa entrada y genera respuestas. Cada evento representa una ocurrencia discreta—generación de texto parcial, chunks de audio, ejecución de herramientas, transcripción, interrupción o completación de turno.

```python title='Implementación de Demo: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py#L219-L234" target="_blank">main.py:219-234</a>'
async for event in runner.run_live(
    user_id=user_id,
    session_id=session_id,
    live_request_queue=live_request_queue,
    run_config=run_config
):
    event_json = event.model_dump_json(exclude_none=True, by_alias=True)
    await websocket.send_text(event_json)
```

Los eventos están diseñados para **entrega de streaming**—recibes respuestas parciales conforme se generan, no solo mensajes completos. Esto habilita actualizaciones de UI en tiempo real y experiencias de usuario responsivas.

Consulta [Parte 3: Manejo de eventos con run_live()](part3.md) para patrones comprensivos de manejo de eventos.

### Fase 4: Terminar sesión Live API

Cuando la sesión de streaming debe terminar (usuario se desconecta, conversación se completa, ocurre timeout), cierra la cola graciosamente para señalar terminación y terminar la sesión Live API.

#### Cerrar la Cola

Envía una señal de cierre a través de la cola para terminar el bucle de streaming:

```python title='Implementación de Demo: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py#L253" target="_blank">main.py:253</a>'
live_request_queue.close()
```

Esto señala a `run_live()` que detenga producción de eventos y salga del bucle de generador async. El agente completa cualquier procesamiento en progreso y la sesión de streaming termina limpiamente.

### Ejemplo de Aplicación FastAPI

Aquí hay una aplicación FastAPI WebSocket completa mostrando todas las cuatro fases integradas con Bidi-streaming apropiado. El patrón clave es **tareas upstream/downstream**: la tarea upstream recibe mensajes de WebSocket y los envía a `LiveRequestQueue`, mientras que la tarea downstream recibe objetos `Event` de `run_live()` y los envía a WebSocket.

!!! note "Implementación de Demo Completa"

    Para la implementación lista para producción con soporte multimodal (texto, audio, imagen), consulta el archivo [`main.py`](https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py) completo.

**Implementación Completa:**

```python
import asyncio
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from google.adk.runners import Runner
from google.adk.agents.run_config import RunConfig, StreamingMode
from google.adk.agents.live_request_queue import LiveRequestQueue
from google.adk.sessions import InMemorySessionService
from google.genai import types
from google_search_agent.agent import agent

# ========================================
# Fase 1: Inicialización de Aplicación (una vez al inicio)
# ========================================

APP_NAME = "bidi-demo"

app = FastAPI()

# Define tu servicio de sesión
session_service = InMemorySessionService()

# Define tu runner
runner = Runner(
    app_name=APP_NAME,
    agent=agent,
    session_service=session_service
)

# ========================================
# Endpoint WebSocket
# ========================================

@app.websocket("/ws/{user_id}/{session_id}")
async def websocket_endpoint(websocket: WebSocket, user_id: str, session_id: str) -> None:
    await websocket.accept()

    # ========================================
    # Fase 2: Inicialización de Sesión (una vez por sesión de streaming)
    # ========================================

    # Crear RunConfig
    response_modalities = ["AUDIO"]
    run_config = RunConfig(
        streaming_mode=StreamingMode.BIDI,
        response_modalities=response_modalities,
        input_audio_transcription=types.AudioTranscriptionConfig(),
        output_audio_transcription=types.AudioTranscriptionConfig(),
        session_resumption=types.SessionResumptionConfig()
    )

    # Obtener o crear sesión
    session = await session_service.get_session(
        app_name=APP_NAME,
        user_id=user_id,
        session_id=session_id
    )
    if not session:
        await session_service.create_session(
            app_name=APP_NAME,
            user_id=user_id,
            session_id=session_id
        )

    # Crear LiveRequestQueue
    live_request_queue = LiveRequestQueue()

    # ========================================
    # Fase 3: Sesión Activa (comunicación bidireccional concurrente)
    # ========================================

    async def upstream_task() -> None:
        """Recibe mensajes de WebSocket y envía a LiveRequestQueue."""
        try:
            while True:
                # Recibir mensaje de texto de WebSocket
                data: str = await websocket.receive_text()

                # Enviar a LiveRequestQueue
                content = types.Content(parts=[types.Part(text=data)])
                live_request_queue.send_content(content)
        except WebSocketDisconnect:
            # Cliente desconectado - señalar cola para cerrar
            pass

    async def downstream_task() -> None:
        """Recibe Events de run_live() y envía a WebSocket."""
        async for event in runner.run_live(
            user_id=user_id,
            session_id=session_id,
            live_request_queue=live_request_queue,
            run_config=run_config
        ):
            # Enviar evento como JSON a WebSocket
            await websocket.send_text(
                event.model_dump_json(exclude_none=True, by_alias=True)
            )

    # Ejecutar ambas tareas concurrentemente
    try:
        await asyncio.gather(
            upstream_task(),