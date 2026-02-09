# Almacenamiento en caché de contexto con Gemini

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v1.15.0</span>
</div>

Cuando trabajas con agentes para completar tareas, es posible que quieras reutilizar
instrucciones extendidas o grandes conjuntos de datos a través de múltiples solicitudes de agente a un
modelo de IA generativa. Reenviar estos datos para cada solicitud de agente es lento,
ineficiente y puede ser costoso. Usar características de almacenamiento en caché de contexto en modelos de IA
generativa puede acelerar significativamente las respuestas y reducir el número de tokens
enviados al modelo para cada solicitud.

La característica de Almacenamiento en Caché de Contexto del ADK te permite cachear datos de solicitud con modelos de IA
generativa que lo soportan, incluyendo Gemini 2.0 y modelos superiores. Este documento
explica cómo configurar y usar esta característica.

## Configurar el almacenamiento en caché de contexto

Configuras la característica de almacenamiento en caché de contexto a nivel del objeto `App` del ADK,
que envuelve tu agente. Usa la clase `ContextCacheConfig` para configurar
estos ajustes, como se muestra en el siguiente ejemplo de código:

```python
from google.adk import Agent
from google.adk.apps.app import App
from google.adk.agents.context_cache_config import ContextCacheConfig

root_agent = Agent(
  # configure an agent using Gemini 2.0 or higher
)

# Create the app with context caching configuration
app = App(
    name='my-caching-agent-app',
    root_agent=root_agent,
    context_cache_config=ContextCacheConfig(
        min_tokens=2048,    # Minimum tokens to trigger caching
        ttl_seconds=600,    # Store for up to 10 minutes
        cache_intervals=5,  # Refresh after 5 uses
    ),
)
```

## Configuración de ajustes

La clase `ContextCacheConfig` tiene los siguientes ajustes que controlan cómo
funciona el almacenamiento en caché para tu agente. Cuando configuras estos ajustes, se aplican a
todos los agentes dentro de tu aplicación.

-   **`min_tokens`** (int): El número mínimo de tokens requeridos en una solicitud
    para habilitar el almacenamiento en caché. Este ajuste te permite evitar la sobrecarga del almacenamiento en caché
    para solicitudes muy pequeñas donde el beneficio de rendimiento sería insignificante.
    Por defecto es `0`.
-   **`ttl_seconds`** (int): El tiempo de vida (TTL) para la caché en segundos.
    Este ajuste determina cuánto tiempo se almacena el contenido cacheado antes de que se
    actualice. Por defecto es `1800` (30 minutos).
-   **`cache_intervals`** (int): El número máximo de veces que el mismo contenido cacheado
    puede ser usado antes de que expire. Este ajuste te permite
    controlar con qué frecuencia se actualiza la caché, incluso si el TTL no ha
    expirado. Por defecto es `10`.

## Próximos pasos

Para una implementación completa de cómo usar y probar la característica de almacenamiento en caché de contexto,
consulta el siguiente ejemplo:

-   [`cache_analysis`](https://github.com/google/adk-python/tree/main/contributing/samples/cache_analysis):
    Un ejemplo de código que demuestra cómo analizar el rendimiento del almacenamiento en caché de
    contexto.

Si tu caso de uso requiere que proporciones instrucciones que se usan a lo largo de
una sesión, considera usar el parámetro `static_instruction` para un agente, que
te permite modificar las instrucciones del sistema para un modelo generativo. Para más
detalles, consulta este código de ejemplo:

-   [`static_instruction`](https://github.com/google/adk-python/tree/main/contributing/samples/static_instruction):
    Una implementación de un agente de mascota digital usando instrucciones estáticas.