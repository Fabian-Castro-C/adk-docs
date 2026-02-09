---
catalog_title: Cartesia
catalog_description: Generate speech, localize voices, and create audio content
catalog_icon: /adk-docs/assets/tools-cartesia.png
---

# Cartesia

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de Cartesia](https://github.com/cartesia-ai/cartesia-mcp) conecta
tu agente ADK a la plataforma de audio de IA [Cartesia](https://cartesia.ai/). Esta
integración le da a tu agente la capacidad de generar habla, localizar voces
entre idiomas y crear contenido de audio usando lenguaje natural.

## Casos de uso

- **Generación de Texto a Voz**: Convierte texto en habla de sonido natural
  usando la diversa biblioteca de voces de Cartesia, con control sobre la selección de voz y
  formato de salida.

- **Localización de Voz**: Transforma voces existentes a diferentes idiomas
  mientras preserva las características del hablante original—ideal para
  creación de contenido multilingüe.

- **Relleno de Audio**: Llena espacios entre segmentos de audio para crear transiciones
  suaves, útil para edición de podcasts o producción de audiolibros.

- **Transformación de Voz**: Convierte clips de audio para que suenen como diferentes voces
  de la biblioteca de Cartesia.

## Requisitos previos

- Regístrate para una [cuenta de Cartesia](https://play.cartesia.ai/sign-in)
- Genera una [clave API](https://play.cartesia.ai/keys) desde el playground de
  Cartesia

## Uso con agente

=== "Python"

    === "Servidor MCP Local"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        CARTESIA_API_KEY = "YOUR_CARTESIA_API_KEY"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="cartesia_agent",
            instruction="Help users generate speech and work with audio content",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="uvx",
                            args=["cartesia-mcp"],
                            env={
                                "CARTESIA_API_KEY": CARTESIA_API_KEY,
                                # "OUTPUT_DIRECTORY": "/path/to/output",  # Opcional
                            }
                        ),
                        timeout=30,
                    ),
                )
            ],
        )
        ```

=== "TypeScript"

    === "Servidor MCP Local"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const CARTESIA_API_KEY = "YOUR_CARTESIA_API_KEY";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "cartesia_agent",
            instruction: "Help users generate speech and work with audio content",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "uvx",
                        args: ["cartesia-mcp"],
                        env: {
                            CARTESIA_API_KEY: CARTESIA_API_KEY,
                            // OUTPUT_DIRECTORY: "/path/to/output",  // Opcional
                        },
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

## Herramientas disponibles

Herramienta | Descripción
---- | -----------
`text_to_speech` | Convierte texto a audio usando una voz especificada
`list_voices` | Lista todas las voces disponibles de Cartesia
`get_voice` | Obtiene detalles sobre una voz específica
`clone_voice` | Clona una voz a partir de muestras de audio
`update_voice` | Actualiza una voz existente
`delete_voice` | Elimina una voz de tu biblioteca
`localize_voice` | Transforma una voz a un idioma diferente
`voice_change` | Convierte un archivo de audio para usar una voz diferente
`infill` | Llena espacios entre segmentos de audio

## Configuración

El servidor MCP de Cartesia puede configurarse usando variables de entorno:

Variable | Descripción | Requerida
-------- | ----------- | --------
`CARTESIA_API_KEY` | Tu clave API de Cartesia | Sí
`OUTPUT_DIRECTORY` | Directorio para almacenar archivos de audio generados | No

## Recursos adicionales

- [Repositorio del Servidor MCP de Cartesia](https://github.com/cartesia-ai/cartesia-mcp)
- [Documentación MCP de Cartesia](https://docs.cartesia.ai/integrations/mcp)
- [Playground de Cartesia](https://play.cartesia.ai/)