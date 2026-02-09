# Introducción al Contexto Conversacional: Session, State y Memory

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span><span class="lst-typescript">TypeScript</span><span class="lst-go">Go</span><span class="lst-java">Java</span>
</div>

Las conversaciones significativas de múltiples turnos requieren que los agentes comprendan el contexto. Al igual que los humanos, necesitan recordar el historial de la conversación: lo que se ha dicho y hecho para mantener la continuidad y evitar repeticiones. El Agent Development Kit (ADK) proporciona formas estructuradas de gestionar este contexto a través de `Session`, `State` y `Memory`.

## Conceptos Fundamentales

Piensa en las diferentes instancias de tus conversaciones con el agente como **hilos de conversación** distintos, que potencialmente se basan en **conocimiento a largo plazo**.

1.  **`Session`**: El Hilo de Conversación Actual

    *   Representa una *única interacción continua* entre un usuario y tu sistema de agente.
    *   Contiene la secuencia cronológica de mensajes y acciones realizadas por el agente (referidas como `Events`) durante *esa interacción específica*.
    *   Una `Session` también puede contener datos temporales (`State`) relevantes solo *durante esta conversación*.

2.  **`State` (`session.state`)**: Datos Dentro de la Conversación Actual

    *   Datos almacenados dentro de una `Session` específica.
    *   Se utilizan para gestionar información relevante *solo* para el hilo de conversación *actual y activo* (por ejemplo, artículos en un carrito de compras *durante este chat*, preferencias del usuario mencionadas *en esta sesión*).

3.  **`Memory`**: Información Consultable Entre Sesiones

    *   Representa un almacén de información que puede abarcar *múltiples sesiones pasadas* o incluir fuentes de datos externas.
    *   Actúa como una base de conocimiento que el agente puede *consultar* para recordar información o contexto más allá de la conversación inmediata.

## Gestión del Contexto: Servicios

ADK proporciona servicios para gestionar estos conceptos:

1.  **`SessionService`**: Gestiona los diferentes hilos de conversación (objetos `Session`)

    *   Maneja el ciclo de vida: crear, recuperar, actualizar (añadiendo `Events`, modificando `State`) y eliminar `Session`s individuales.

2.  **`MemoryService`**: Gestiona el Almacén de Conocimiento a Largo Plazo (`Memory`)

    *   Maneja la ingesta de información (a menudo desde `Session`s completadas) en el almacén a largo plazo.
    *   Proporciona métodos para buscar este conocimiento almacenado basándose en consultas.

**Implementaciones**: ADK ofrece diferentes implementaciones tanto para `SessionService` como para `MemoryService`, permitiéndote elegir el backend de almacenamiento que mejor se adapte a las necesidades de tu aplicación. En particular, se proporcionan **implementaciones en memoria** para ambos servicios; estas están diseñadas específicamente para **pruebas locales y desarrollo rápido**. Es importante recordar que **todos los datos almacenados utilizando estas opciones en memoria (sesiones, estado o conocimiento a largo plazo) se pierden cuando tu aplicación se reinicia**. Para persistencia y escalabilidad más allá de las pruebas locales, ADK también ofrece opciones de servicios basados en la nube y bases de datos.

**En Resumen:**

*   **`Session` & `State`**: Se centran en la **interacción actual** – el historial y datos de la *única conversación activa*. Gestionados principalmente por un `SessionService`.
*   **Memory**: Se centra en la **información pasada y externa** – un *archivo consultable* que potencialmente abarca múltiples conversaciones. Gestionado por un `MemoryService`.

## ¿Qué Sigue?

En las siguientes secciones, profundizaremos en cada uno de estos componentes:

*   **`Session`**: Comprender su estructura y `Events`.
*   **`State`**: Cómo leer, escribir y gestionar eficazmente datos específicos de la sesión.
*   **`SessionService`**: Elegir el backend de almacenamiento adecuado para tus sesiones.
*   **`MemoryService`**: Explorar opciones para almacenar y recuperar contexto más amplio.

Comprender estos conceptos es fundamental para construir agentes que puedan participar en conversaciones complejas, con estado y conscientes del contexto.