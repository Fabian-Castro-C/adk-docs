---
catalog_title: Data Agents
catalog_description: Analyze data with AI-powered agents
catalog_icon: /assets/tools-vertex-ai.png
---

# Herramientas de Data Agents para ADK

<div class="language-support-tag">
  <span class="lst-supported">Compatible con ADK</span><span class="lst-python">Python v1.23.0</span>
</div>

Estas son un conjunto de herramientas destinadas a proporcionar integración con Data Agents impulsados por [Conversational Analytics API](https://docs.cloud.google.com/gemini/docs/conversational-analytics-api/overview).

Los Data Agents son agentes impulsados por IA que te ayudan a analizar tus datos usando lenguaje natural. Al configurar un Data Agent, puedes elegir entre fuentes de datos compatibles, incluyendo **BigQuery**, **Looker** y **Looker Studio**.

**Requisitos previos**

Antes de usar estas herramientas, debes construir y configurar tus Data Agents en Google Cloud:

* [Construir un data agent usando HTTP y Python](https://docs.cloud.google.com/gemini/docs/conversational-analytics-api/build-agent-http)
* [Construir un data agent usando el SDK de Python](https://docs.cloud.google.com/gemini/docs/conversational-analytics-api/build-agent-sdk)
* [Crear un data agent en BigQuery Studio](https://docs.cloud.google.com/bigquery/docs/create-data-agents#create_a_data_agent)

El `DataAgentToolset` incluye las siguientes herramientas:

* **`list_accessible_data_agents`**: Lista los Data Agents a los que tienes permiso de acceso en el proyecto GCP configurado.
* **`get_data_agent_info`**: Recupera detalles sobre un Data Agent específico dado su nombre de recurso completo.
* **`ask_data_agent`**: Chatea con un Data Agent específico usando lenguaje natural.

Están empaquetadas en el conjunto de herramientas `DataAgentToolset`.

```py
--8<-- "examples/python/snippets/tools/built-in-tools/data_agent.py"
```