# Migración del esquema de base de datos de sesiones

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v1.22.1</span>
</div>

Si estás usando `DatabaseSessionService` y actualizando a la versión de ADK Python
v1.22.0 o superior, debes migrar tu base de datos al nuevo esquema de base de datos
de sesiones. A partir de la versión de ADK Python v1.22.0, el esquema de base de datos para
`DatabaseSessionService` ha sido actualizado de `v0`, que utiliza serialización
basada en pickle, a `v1`, que utiliza serialización basada en JSON. Las bases de datos
con esquema de sesiones `v0` previas continuarán funcionando con ADK Python v1.22.0 y versiones superiores,
pero el esquema `v1` puede ser requerido en versiones futuras.


## Migrar base de datos de sesiones

Se proporciona un script de migración para facilitar el proceso de migración. El script
lee datos de tu base de datos existente, los convierte al nuevo formato y
los escribe en una nueva base de datos. Puedes ejecutar la migración usando el comando
`migrate session` de la Interfaz de Línea de Comandos (CLI) de ADK, como se muestra en los siguientes ejemplos:

!!! warning "Requerido: ADK Python v1.22.1 o superior"

    ADK Python v1.22.1 es requerido para este procedimiento porque incluye la
    función de interfaz de línea de comandos de migración y correcciones de errores para soportar el
    cambio de esquema de base de datos de sesiones.

=== "SQLite"

    ```bash
    adk migrate session \
      --source_db_url=sqlite:///source.db \
      --dest_db_url=sqlite:///dest.db
    ```

=== "PostgreSQL"

    ```bash
    adk migrate session \
      --source_db_url=postgresql://localhost:5432/v0 \
      --dest_db_url=postgresql://localhost:5432/v1
    ```

Después de ejecutar la migración, actualiza tu configuración de `DatabaseSessionService`
para usar la nueva URL de base de datos que especificaste para `dest_db_url`.