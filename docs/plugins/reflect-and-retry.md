# Plugin de Herramienta Reflect and Retry

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v1.16.0</span>
</div>

El plugin Reflect and Retry Tool puede ayudar a tu agente a recuperarse de respuestas de error
de las [Herramientas](/tools-custom/) de ADK y reintentar automáticamente la
solicitud de herramienta. Este plugin intercepta fallos de herramientas, proporciona orientación estructurada
al modelo de IA para reflexión y corrección, y reintenta la operación hasta un
límite configurable. Este plugin puede ayudarte a construir más resiliencia en tus
flujos de trabajo de agentes, incluyendo las siguientes capacidades:

*   **Seguro para concurrencia**: Utiliza bloqueos para manejar de forma segura ejecuciones de herramientas paralelas.
*   **Ámbito configurable**: Rastrea fallos por invocación (predeterminado) o globalmente.
*   **Rastreo granular**: Los conteos de fallos se rastrean por herramienta.
*   **Extracción de errores personalizada**: Soporta la detección de errores en respuestas normales de herramientas.

## Agregar Plugin Reflect and Retry

Agrega este plugin a tu flujo de trabajo de ADK añadiéndolo a la configuración de plugins del
objeto App de tu proyecto ADK, como se muestra a continuación:

```python
from google.adk.apps.app import App
from google.adk.plugins import ReflectAndRetryToolPlugin

app = App(
    name="my_app",
    root_agent=root_agent,
    plugins=[
        ReflectAndRetryToolPlugin(max_retries=3),
    ],
)
```

Con esta configuración, si alguna herramienta llamada por un agente devuelve un error, la
solicitud se actualiza e intenta de nuevo, hasta un máximo de 3 intentos, por herramienta.

## Configuraciones de ajustes

El Plugin Reflect and Retry tiene las siguientes opciones de configuración:

*   **`max_retries`**: (opcional) Número total de intentos adicionales que el sistema
    realiza para recibir una respuesta sin error. El valor predeterminado es 3.
*   **`throw_exception_if_retry_exceeded`**: (opcional) Si se establece en `False`, el
    sistema no genera un error si el intento de reintento final falla. El valor
    predeterminado es `True`.
*   **`tracking_scope`**: (opcional)
    *   **`TrackingScope.INVOCATION`**: Rastrea fallos de herramientas a través de una sola
        invocación y usuario. Este valor es el predeterminado.
    *   **`TrackingScope.GLOBAL`**: Rastrea fallos de herramientas a través de todas las invocaciones
        y todos los usuarios.

### Configuración avanzada

Puedes modificar aún más el comportamiento de este plugin extendiendo la
clase `ReflectAndRetryToolPlugin`. El siguiente ejemplo de código
demuestra una extensión simple del comportamiento seleccionando
respuestas con un estado de error:

```python
class CustomRetryPlugin(ReflectAndRetryToolPlugin):
  async def extract_error_from_result(self, *, tool, tool_args,tool_context,
  result):
    # Detectar error basado en el contenido de la respuesta
    if result.get('status') == 'error':
        return result
    return None  # No se detectó error

# agrega este plugin modificado a tu objeto App:
error_handling_plugin = CustomRetryPlugin(max_retries=5)
```

## Próximos pasos

Para ejemplos de código completos usando el plugin Reflect and Retry, consulta lo siguiente:

*   Ejemplo de código
    [Básico](https://github.com/google/adk-python/tree/main/contributing/samples/plugin_reflect_tool_retry/basic)
*   Ejemplo de código
    [Nombre de función alucinado](https://github.com/google/adk-python/tree/main/contributing/samples/plugin_reflect_tool_retry/hallucinating_func_name)