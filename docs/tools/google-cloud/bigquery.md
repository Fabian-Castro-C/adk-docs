---
catalog_title: BigQuery Tools
catalog_description: Connect with BigQuery to retrieve data and perform analysis
catalog_icon: /assets/tools-bigquery.png
---

# Herramientas de BigQuery para ADK

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v1.1.0</span>
</div>

Estas son un conjunto de herramientas destinadas a proporcionar integración con BigQuery, a saber:

* **`list_dataset_ids`**: Obtiene los ids de conjuntos de datos de BigQuery presentes en un proyecto de GCP.
* **`get_dataset_info`**: Obtiene metadatos sobre un conjunto de datos de BigQuery.
* **`list_table_ids`**: Obtiene los ids de tablas presentes en un conjunto de datos de BigQuery.
* **`get_table_info`**: Obtiene metadatos sobre una tabla de BigQuery.
* **`execute_sql`**: Ejecuta una consulta SQL en BigQuery y obtiene el resultado.
* **`forecast`**: Ejecuta un pronóstico de series temporales con BigQuery AI usando la función `AI.FORECAST`.
* **`ask_data_insights`**: Responde preguntas sobre datos en tablas de BigQuery usando lenguaje natural.

Están empaquetadas en el conjunto de herramientas `BigQueryToolset`.

```py
--8<-- "examples/python/snippets/tools/built-in-tools/bigquery.py"
```

Nota: Si deseas acceder a un agente de datos de BigQuery como una herramienta, consulta [Herramientas de agentes de datos para ADK](data-agent.md).