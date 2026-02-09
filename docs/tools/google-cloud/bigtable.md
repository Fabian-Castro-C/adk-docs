---
catalog_title: Bigtable Tools
catalog_description: Interact with Bigtable to retrieve data and execute SQL
catalog_icon: /adk-docs/assets/tools-bigtable.png
---

# Herramienta de base de datos Bigtable para ADK

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v1.12.0</span>
</div>

Estas son un conjunto de herramientas destinadas a proporcionar integración con Bigtable, a saber:

* **`list_instances`**: Obtiene instancias de Bigtable en un proyecto de Google Cloud.
* **`get_instance_info`**: Obtiene información de metadatos de la instancia en un proyecto de Google Cloud.
* **`list_tables`**: Obtiene tablas en una instancia de GCP Bigtable.
* **`get_table_info`**: Obtiene información de metadatos de tabla en un GCP Bigtable.
* **`execute_sql`**: Ejecuta una consulta SQL en una tabla de Bigtable y obtiene el resultado.

Están empaquetadas en el conjunto de herramientas `BigtableToolset`.

```py
--8<-- "examples/python/snippets/tools/built-in-tools/bigtable.py"
```