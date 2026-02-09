---
catalog_title: Daytona
catalog_description: Ejecuta código, ejecuta comandos de shell y administra archivos en sandboxes seguros
catalog_icon: /adk-docs/assets/plugins-daytona.png
---

# Daytona

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span>
</div>

El [plugin ADK de Daytona](https://github.com/daytonaio/daytona-adk-plugin) conecta tu agente
ADK a sandboxes de [Daytona](https://www.daytona.io/). Esta integración le da
a tu agente la capacidad de ejecutar código, ejecutar comandos de shell y administrar archivos en
entornos aislados, habilitando la ejecución segura de código generado por IA.

## Casos de uso

- **Ejecución Segura de Código**: Ejecuta código Python, JavaScript y TypeScript en
  sandboxes aislados sin arriesgar tu entorno local.

- **Automatización de Comandos Shell**: Ejecuta comandos de shell con tiempos de espera
  configurables y directorios de trabajo para tareas de construcción, instalaciones u
  operaciones del sistema.

- **Administración de Archivos**: Sube scripts y conjuntos de datos a sandboxes, luego recupera
  salidas generadas y resultados.

## Prerrequisitos

- Una cuenta de [Daytona](https://www.daytona.io/)
- Clave API de Daytona

## Instalación

```bash
pip install daytona-adk
```

## Uso con agente

```python
from daytona_adk import DaytonaPlugin
from google.adk.agents import Agent

plugin = DaytonaPlugin(
  api_key="your-daytona-api-key" # O establece la variable de entorno DAYTONA_API_KEY
)

root_agent = Agent(
    model="gemini-2.5-pro",
    name="sandbox_agent",
    instruction="Help users execute code and commands in a secure sandbox",
    tools=plugin.get_tools(),
)
```

## Herramientas disponibles

Herramienta | Descripción
---- | -----------
`execute_code_in_daytona` | Ejecuta código Python, JavaScript o TypeScript
`execute_command_in_daytona` | Ejecuta comandos de shell
`upload_file_to_daytona` | Sube scripts o archivos de datos al sandbox
`read_file_from_daytona` | Lee salidas de scripts o archivos generados
`start_long_running_command_daytona` | Inicia procesos en segundo plano (servidores, observadores)

## Aprende más

Para una guía detallada sobre cómo construir un agente generador de código que escribe, prueba y
verifica código en sandboxes seguros, consulta
[esta guía](https://www.daytona.io/docs/en/google-adk-code-generator).

## Recursos adicionales

- [Guía del Agente Generador de Código](https://www.daytona.io/docs/en/google-adk-code-generator)
- [Daytona ADK en PyPI](https://pypi.org/project/daytona-adk/)
- [Daytona ADK en GitHub](https://github.com/daytonaio/daytona-adk-plugin)
- [Documentación de Daytona](https://www.daytona.io/docs)