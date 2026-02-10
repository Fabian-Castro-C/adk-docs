---
catalog_title: Asana
catalog_description: Manage projects, tasks, and goals for team collaboration
catalog_icon: /assets/tools-asana.png
---

# Asana

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de Asana](https://developers.asana.com/docs/using-asanas-mcp-server)
conecta tu agente ADK a la plataforma de gestión de trabajo [Asana](https://asana.com/).
Esta integración le da a tu agente la capacidad de gestionar proyectos,
tareas, objetivos y colaboración en equipo usando lenguaje natural.

## Casos de uso

- **Rastrear Estado de Proyectos**: Obtén actualizaciones en tiempo real sobre el progreso del proyecto, visualiza
  informes de estado y recupera información sobre hitos y fechas límite.

- **Gestionar Tareas**: Crea, actualiza y organiza tareas usando lenguaje natural.
  Deja que tu agente maneje asignaciones de tareas, cambios de estado y actualizaciones de prioridad.

- **Monitorear Objetivos**: Accede y actualiza los Objetivos de Asana para rastrear objetivos del equipo y
  resultados clave en toda tu organización.

## Requisitos previos

- Una cuenta de [Asana](https://asana.com/) con acceso a un espacio de trabajo

## Uso con agente

=== "Python"

    === "Local MCP Server"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="asana_agent",
            instruction="Help users manage projects, tasks, and goals in Asana",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="npx",
                            args=[
                                "-y",
                                "mcp-remote",
                                "https://mcp.asana.com/sse",
                            ]
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

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "asana_agent",
            instruction: "Help users manage projects, tasks, and goals in Asana",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "npx",
                        args: [
                            "-y",
                            "mcp-remote",
                            "https://mcp.asana.com/sse",
                        ],
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

!!! note

    Cuando ejecutas este agente por primera vez, una ventana del navegador se abre
    automáticamente para solicitar acceso mediante OAuth. Alternativamente, puedes usar la
    URL de autorización impresa en la consola. Debes aprobar esta solicitud para
    permitir que el agente acceda a tus datos de Asana.

## Herramientas disponibles

El servidor MCP de Asana incluye más de 30 herramientas organizadas por categoría. Las herramientas se
descubren automáticamente cuando tu agente se conecta. Usa la
[Interfaz Web de ADK](/runtime/web-interface/) para ver las herramientas disponibles en el
gráfico de trazas después de ejecutar tu agente.

Categoría | Descripción
-------- | -----------
Rastreo de proyectos | Obtén actualizaciones de estado de proyectos e informes
Gestión de tareas | Crea, actualiza y organiza tareas
Información de usuarios | Accede a detalles de usuarios y asignaciones
Objetivos | Rastrea y actualiza Objetivos de Asana
Organización de equipos | Gestiona estructuras de equipos y membresía
Búsqueda de objetos | Búsqueda rápida con autocompletado en objetos de Asana

## Recursos adicionales

- [Documentación del Servidor MCP de Asana](https://developers.asana.com/docs/using-asanas-mcp-server)
- [Guía de Integración MCP de Asana](https://developers.asana.com/docs/integrating-with-asanas-mcp-server)