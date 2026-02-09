---
catalog_title: ElevenLabs
catalog_description: Generate speech, clone voices, transcribe audio, and create sound effects
catalog_icon: /adk-docs/assets/tools-elevenlabs.png
---

# ElevenLabs

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de ElevenLabs](https://github.com/elevenlabs/elevenlabs-mcp)
conecta tu agente ADK a la plataforma de audio con IA [ElevenLabs](https://elevenlabs.io/). Esta integración le da a tu agente la capacidad de generar voz,
clonar voces, transcribir audio, crear efectos de sonido y construir experiencias
conversacionales de IA utilizando lenguaje natural.


## Casos de uso

- **Generación de Texto a Voz**: Convierte texto en voz de sonido natural
  utilizando una variedad de voces, con control detallado sobre estabilidad, estilo
  y configuraciones de similitud.

- **Clonación y Diseño de Voces**: Clona voces a partir de muestras de audio o genera nuevas
  voces a partir de descripciones de texto de características deseadas como edad, género,
  acento y tono.

- **Procesamiento de Audio**: Aísla el habla del ruido de fondo, convierte audio para
  sonar como voces diferentes, o transcribe el habla a texto con
  identificación de hablante.

- **Efectos de Sonido y Paisajes Sonoros**: Genera efectos de sonido y paisajes
  sonoros ambientales a partir de descripciones de texto, como "una tormenta eléctrica en una jungla densa
  con animales reaccionando al clima."

## Prerrequisitos

- Regístrate para obtener una [cuenta de ElevenLabs](https://elevenlabs.io/app/sign-up)
- Genera una [clave API](https://elevenlabs.io/app/settings/api-keys) desde la
  configuración de tu cuenta

## Uso con agente

=== "Python"

    === "Local MCP Server"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        ELEVENLABS_API_KEY = "YOUR_ELEVENLABS_API_KEY"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="elevenlabs_agent",
            instruction="Help users generate speech, clone voices, and process audio",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="uvx",
                            args=["elevenlabs-mcp"],
                            env={
                                "ELEVENLABS_API_KEY": ELEVENLABS_API_KEY,
                            }
                        ),
                        timeout=30,
                    ),
                )
            ],
        )
        ```

=== "TypeScript"

    === "Local MCP Server"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const ELEVENLABS_API_KEY = "YOUR_ELEVENLABS_API_KEY";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "elevenlabs_agent",
            instruction: "Help users generate speech, clone voices, and process audio",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "uvx",
                        args: ["elevenlabs-mcp"],
                        env: {
                            ELEVENLABS_API_KEY: ELEVENLABS_API_KEY,
                        },
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

## Herramientas disponibles

### Texto a voz y voz

Herramienta | Descripción
---- | -----------
`text_to_speech` | Genera voz a partir de texto utilizando una voz especificada
`speech_to_speech` | Transforma audio para sonar como una voz diferente
`text_to_voice` | Genera una vista previa de voz a partir de una descripción de texto
`create_voice_from_preview` | Guarda una vista previa de voz generada en tu biblioteca
`voice_clone` | Clona una voz a partir de muestras de audio
`get_voice` | Obtiene detalles sobre una voz específica
`search_voices` | Busca voces en tu biblioteca
`search_voice_library` | Busca en la biblioteca de voces pública
`list_models` | Lista los modelos de texto a voz disponibles

### Procesamiento de audio

Herramienta | Descripción
---- | -----------
`speech_to_text` | Transcribe audio a texto con identificación de hablante
`text_to_sound_effects` | Genera efectos de sonido a partir de descripciones de texto
`isolate_audio` | Separa el habla del ruido de fondo y la música
`play_audio` | Reproduce un archivo de audio localmente
`compose_music` | Genera música a partir de una descripción
`create_composition_plan` | Crea un plan para composición musical

### IA conversacional

Herramienta | Descripción
---- | -----------
`create_agent` | Crea un agente de IA conversacional
`get_agent` | Obtiene detalles sobre un agente específico
`list_agents` | Lista todos tus agentes de IA conversacional
`add_knowledge_base_to_agent` | Agrega una base de conocimiento a un agente
`make_outbound_call` | Inicia una llamada telefónica saliente utilizando un agente
`list_phone_numbers` | Lista los números de teléfono disponibles
`get_conversation` | Obtiene detalles sobre una conversación específica
`list_conversations` | Lista todas las conversaciones

### Cuenta

Herramienta | Descripción
---- | -----------
`check_subscription` | Verifica tu suscripción y uso de créditos

## Configuración

El servidor MCP de ElevenLabs se puede configurar usando variables de entorno:

Variable | Descripción | Por defecto
-------- | ----------- | -------
`ELEVENLABS_API_KEY` | Tu clave API de ElevenLabs | Requerido
`ELEVENLABS_MCP_BASE_PATH` | Ruta base para operaciones de archivos | `~/Desktop`
`ELEVENLABS_MCP_OUTPUT_MODE` | Cómo se devuelven los archivos generados | `files`
`ELEVENLABS_API_RESIDENCY` | Región de residencia de datos (solo empresarial) | `us`

### Modos de salida

La variable de entorno `ELEVENLABS_MCP_OUTPUT_MODE` admite tres modos:

- **`files`** (por defecto): Guarda archivos en disco y devuelve rutas de archivos
- **`resources`**: Devuelve archivos como recursos MCP (datos binarios codificados en base64)
- **`both`**: Guarda archivos en disco Y devuelve como recursos MCP

## Recursos adicionales

- [Repositorio del Servidor MCP de ElevenLabs](https://github.com/elevenlabs/elevenlabs-mcp)
- [Presentando ElevenLabs MCP](https://elevenlabs.io/blog/introducing-elevenlabs-mcp)
- [Documentación de ElevenLabs](https://elevenlabs.io/docs)