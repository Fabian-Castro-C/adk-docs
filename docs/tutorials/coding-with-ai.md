# Programando con IA

La documentación del Agent Development Kit (ADK) soporta el
[estándar `/llms.txt`](https://llmstxt.org/), proporcionando un índice legible por máquina
de la documentación optimizado para Modelos de Lenguaje Grande (LLMs). Esto te permite
usar fácilmente la documentación del ADK como contexto en tu entorno de desarrollo
potenciado por IA.

## ¿Qué es llms.txt?

`llms.txt` es un archivo de texto estandarizado que actúa como un mapa para los LLMs, listando las
páginas de documentación más importantes y sus descripciones. Esto ayuda a las herramientas de IA
a entender la estructura de la documentación del ADK y recuperar información relevante
para responder tus preguntas.

La documentación del ADK proporciona los siguientes archivos que se generan automáticamente
con cada actualización:

Archivo | Mejor para... | URL
---- | ----------- | ---
**`llms.txt`** | Herramientas que pueden obtener enlaces dinámicamente | [`https://google.github.io/llms.txt`](https://google.github.io/llms.txt)
**`llms-full.txt`** | Herramientas que necesitan un volcado de texto estático y único de todo el sitio | [`https://google.github.io/llms-full.txt`](https://google.github.io/llms-full.txt)

## Uso en Herramientas de Desarrollo

Puedes usar estos archivos para potenciar tus asistentes de codificación con IA con conocimiento del ADK.
Esta funcionalidad permite que tus agentes busquen y lean autónomamente la documentación del ADK
mientras planifican tareas y generan código.

### Gemini CLI

El [Gemini CLI](https://geminicli.com/) puede configurarse para consultar la documentación del ADK
usando la
[Extensión ADK Docs](https://github.com/derailed-dash/adk-docs-ext).

**Instalación:**

Para instalar la extensión, ejecuta el siguiente comando:

```bash
gemini extensions install https://github.com/derailed-dash/adk-docs-ext
```

**Uso:**

Una vez instalada, la extensión se habilita automáticamente. Puedes hacer preguntas
sobre ADK directamente en Gemini CLI, y usará el archivo `llms.txt` y la
documentación del ADK para proporcionar respuestas precisas y generar código.

Por ejemplo, puedes hacer la siguiente pregunta desde Gemini CLI:

> ¿Cómo creo una herramienta de función usando Agent Development Kit?

---

### Antigravity

El IDE [Antigravity](https://antigravity.google/) puede configurarse para acceder
a la documentación del ADK ejecutando un servidor MCP personalizado que apunta al
archivo `llms.txt` para ADK.

**Requisitos previos:**

Asegúrate de tener la herramienta [`uv`](https://docs.astral.sh/uv/) instalada, ya que esta
configuración usa `uvx` para ejecutar el servidor de documentación sin instalación
manual.

**Configuración:**

1. Abre la tienda MCP a través del menú **...** (más) en la parte superior del panel de
   agente del editor.
2. Haz clic en **Manage MCP Servers**.
3. Haz clic en **View raw config**.
4. Agrega la siguiente entrada a `mcp_config.json` con tu configuración de servidor MCP
   personalizado. Si este es tu primer servidor MCP, puedes pegar el bloque de código
   completo:

    ```json
    {
      "mcpServers": {
        "adk-docs-mcp": {
          "command": "uvx",
          "args": [
            "--from",
            "mcpdoc",
            "mcpdoc",
            "--urls",
            "AgentDevelopmentKit:https://google.github.io/llms.txt",
            "--transport",
            "stdio"
          ]
        }
      }
    }
    ```

Consulta la
[documentación MCP de Antigravity](https://antigravity.google/docs/mcp) para más
información sobre la gestión de servidores MCP.

**Uso:**

Una vez configurado, puedes dar instrucciones al agente de codificación como:

> Usa la documentación del ADK para construir un agente multi-herramienta que use Gemini 2.5 Pro e
> incluya una herramienta simulada de consulta del clima y una herramienta personalizada de calculadora. Verifica el
> agente usando `adk run`.

---

### Claude Code

[Claude Code](https://code.claude.com/docs/en/overview) puede configurarse para
consultar la documentación del ADK agregando un
[servidor MCP](https://code.claude.com/docs/en/mcp).

**Instalación:**

Para agregar un servidor MCP para la documentación del ADK a Claude Code, ejecuta el siguiente comando:

```bash
claude mcp add adk-docs --transport stdio -- uvx --from mcpdoc mcpdoc --urls AgentDevelopmentKit:https://google.github.io/llms.txt --transport stdio
```

**Uso:**

Una vez instalado, el servidor MCP se habilita automáticamente. Puedes hacer preguntas
sobre ADK directamente en Claude Code, y usará el archivo `llms.txt` y la
documentación del ADK para proporcionar respuestas precisas y generar código.

Por ejemplo, puedes hacer la siguiente pregunta desde Claude Code:

> ¿Cómo creo una herramienta de función usando Agent Development Kit?

---

### Cursor

El IDE [Cursor](https://cursor.com/) puede configurarse para acceder a la documentación del ADK
ejecutando un servidor MCP personalizado que apunta al archivo `llms.txt`
para ADK.

**Requisitos previos:**

Asegúrate de tener la herramienta [`uv`](https://docs.astral.sh/uv/) instalada, ya que esta
configuración usa `uvx` para ejecutar el servidor de documentación sin instalación
manual.

**Configuración:**

1. Abre **Cursor Settings** y navega a la pestaña **Tools & MCP**.
2. Haz clic en **New MCP Server**, lo cual abrirá `mcp.json` para edición.
3. Agrega la siguiente entrada a `mcp.json` con tu configuración de servidor MCP
   personalizado. Si este es tu primer servidor MCP, puedes pegar el bloque de código
   completo:

    ```json
    {
      "mcpServers": {
        "adk-docs-mcp": {
          "command": "uvx",
          "args": [
            "--from",
            "mcpdoc",
            "mcpdoc",
            "--urls",
            "AgentDevelopmentKit:https://google.github.io/llms.txt",
            "--transport",
            "stdio"
          ]
        }
      }
    }
    ```

Consulta la [documentación MCP de Cursor](https://cursor.com/docs/context/mcp) para
más información sobre la gestión de servidores MCP.

**Uso:**

Una vez configurado, puedes dar instrucciones al agente de codificación como:

> Usa la documentación del ADK para construir un agente multi-herramienta que use Gemini 2.5 Pro e
> incluya una herramienta simulada de consulta del clima y una herramienta personalizada de calculadora. Verifica el
> agente usando `adk run`.

---

### Otras Herramientas

Cualquier herramienta que soporte el estándar `llms.txt` o pueda ingerir documentación desde
una URL puede beneficiarse de estos archivos. Puedes proporcionar la URL
`https://google.github.io/llms.txt` (o `llms-full.txt`) a la
configuración de base de conocimiento de tu herramienta o configuración de servidor MCP.