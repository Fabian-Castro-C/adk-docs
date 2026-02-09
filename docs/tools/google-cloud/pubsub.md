---
catalog_title: Pub/Sub Tools
catalog_description: Publish, pull, and acknowledge messages from Google Cloud Pub/Sub
catalog_icon: /adk-docs/assets/tools-pubsub.png
---

# Herramientas Pub/Sub para ADK

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v1.22.0</span>
</div>

El `PubSubToolset` permite a los agentes interactuar con el
servicio [Google Cloud Pub/Sub](https://cloud.google.com/pubsub)
para publicar, extraer y confirmar mensajes.

## Requisitos previos

Antes de usar el `PubSubToolset`, necesitas:

1.  **Habilitar la API de Pub/Sub** en tu proyecto de Google Cloud.
2.  **Autenticar y autorizar**: Asegúrate de que el principal (por ejemplo, usuario, cuenta de servicio) que ejecuta el agente tenga los permisos IAM necesarios para realizar operaciones de Pub/Sub. Para más información sobre los roles de Pub/Sub, consulta la [documentación de control de acceso de Pub/Sub](https://cloud.google.com/pubsub/docs/access-control).
3.  **Crear un tema o suscripción**: [Crea un tema](https://cloud.google.com/pubsub/docs/create-topic) para publicar mensajes y [crea una suscripción](https://cloud.google.com/pubsub/docs/create-subscription) para recibirlos.


## Uso

```py
--8<-- "examples/python/snippets/tools/built-in-tools/pubsub.py"
```
## Herramientas

El `PubSubToolset` incluye las siguientes herramientas:

### `publish_message`

Publica un mensaje en un tema de Pub/Sub.

| Parámetro      | Tipo                | Descripción                                                                                             |
| -------------- | ------------------- | ------------------------------------------------------------------------------------------------------- |
| `topic_name`   | `str`               | El nombre del tema de Pub/Sub (por ejemplo, `projects/my-project/topics/my-topic`).                            |
| `message`      | `str`               | El contenido del mensaje a publicar.                                                                         |
| `attributes`   | `dict[str, str]`    | (Opcional) Atributos para adjuntar al mensaje.                                                         |
| `ordering_key` | `str`               | (Opcional) La clave de ordenamiento para el mensaje. Si estableces este parámetro, los mensajes se publican en orden. |

### `pull_messages`

Extrae mensajes de una suscripción de Pub/Sub.

| Parámetro           | Tipo    | Descripción                                                                                                 |
| ------------------- | ------- | ----------------------------------------------------------------------------------------------------------- |
| `subscription_name` | `str`   | El nombre de la suscripción de Pub/Sub (por ejemplo, `projects/my-project/subscriptions/my-sub`).                      |
| `max_messages`      | `int`   | (Opcional) El número máximo de mensajes a extraer. Por defecto es `1`.                                         |
| `auto_ack`          | `bool`  | (Opcional) Si se deben confirmar automáticamente los mensajes. Por defecto es `False`.                            |

### `acknowledge_messages`

Confirma uno o más mensajes en una suscripción de Pub/Sub.

| Parámetro           | Tipo          | Descripción                                                                                       |
| ------------------- | ------------- | ------------------------------------------------------------------------------------------------- |
| `subscription_name` | `str`         | El nombre de la suscripción de Pub/Sub (por ejemplo, `projects/my-project/subscriptions/my-sub`).            |
| `ack_ids`           | `list[str]`   | Una lista de IDs de confirmación para confirmar.                                                      |