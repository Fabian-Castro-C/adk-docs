---
catalog_title: Spanner Tools
catalog_description: Interact with Spanner to retrieve data, search, and execute SQL
catalog_icon: /adk-docs/assets/tools-spanner.png
---

# Herramienta de base de datos Spanner para ADK

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v1.11.0</span>
</div>

Este es un conjunto de herramientas destinadas a proporcionar integración con Spanner, a saber:

* **`list_table_names`**: Obtiene los nombres de tablas presentes en una base de datos GCP Spanner.
* **`list_table_indexes`**: Obtiene los índices de tablas presentes en una base de datos GCP Spanner.
* **`list_table_index_columns`**: Obtiene las columnas de índice de tablas presentes en una base de datos GCP Spanner.
* **`list_named_schemas`**: Obtiene el esquema nombrado de una base de datos Spanner.
* **`get_table_schema`**: Obtiene el esquema de tabla de la base de datos Spanner e información de metadatos.
* **`execute_sql`**: Ejecuta una consulta SQL en la base de datos Spanner y obtiene el resultado.
* **`similarity_search`**: Búsqueda por similitud en Spanner usando una consulta de texto.

Están empaquetadas en el conjunto de herramientas `SpannerToolset`.

```py
--8<-- "examples/python/snippets/tools/built-in-tools/spanner.py"
```