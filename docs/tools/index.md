---
hide:
  - toc
---

# Herramientas e Integraciones para Agentes

Consulta las siguientes herramientas e integraciones preconfiguradas que puedes usar con agentes ADK:

### Gemini

{{$ render_catalog('tools/gemini-api/*.md') $}}

### Google Cloud

{{$ render_catalog('tools/google-cloud/*.md') $}}

### Terceros

{{$ render_catalog('tools/third-party/*.md') $}}

## Usar herramientas preconfiguradas con agentes ADK

Sigue estos pasos generales para incluir herramientas en tus agentes ADK:

1. **Importar:** Importa la herramienta deseada desde el módulo de herramientas. Esto es
   `agents.tools` en Python, `@google/adk` en TypeScript,
   `google.golang.org/adk/tool` en Go, o `com.google.adk.tools` en Java.
2. **Configurar:** Inicializa la herramienta, proporcionando los parámetros requeridos si los hay.
3. **Registrar:** Agrega la herramienta inicializada a la lista ***tools*** de tu Agente.

Una vez agregada a un agente, el agente puede decidir usar la herramienta basándose en el prompt del usuario
y sus instrucciones. El framework maneja la ejecución de la
herramienta cuando el agente la llama.

!!! note "Nota: Limitaciones al usar múltiples herramientas"
    Algunas herramientas ADK ***no pueden ser usadas con otras herramientas en el mismo agente***.
    Para más información sobre herramientas con estas limitaciones, consulta
    [Limitaciones para herramientas ADK](/tools/limitations/#one-tool-one-agent).

## Construir herramientas para agentes

Si las herramientas anteriores no satisfacen tus necesidades, puedes construir herramientas para tus flujos de trabajo ADK
usando las siguientes guías:

*   **[Function Tools](/tools-custom/function-tools/)**: Construye herramientas personalizadas para
    las necesidades específicas de tu agente ADK.
*   **[MCP Tools](/tools-custom/mcp-tools/)**: Conecta servidores MCP como herramientas
    para tus agentes ADK.
*   **[OpenAPI Integration](/tools-custom/openapi-tools/)**:
    Genera herramientas invocables directamente desde una Especificación OpenAPI.