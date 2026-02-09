# Patrones de Diseño y Mejores Prácticas para Callbacks

Los callbacks ofrecen ganchos poderosos en el ciclo de vida del agente. Aquí hay patrones de diseño comunes que ilustran cómo aprovecharlos efectivamente en ADK, seguidos de mejores prácticas para la implementación.

## Patrones de Diseño

Estos patrones demuestran formas típicas de mejorar o controlar el comportamiento del agente usando callbacks:

### 1. Barreras de Protección y Aplicación de Políticas { #guardrails-policy-enforcement }

**Descripción del Patrón:**
Interceptar solicitudes antes de que lleguen al LLM o las herramientas para aplicar reglas.

**Implementación:**
- Usa `before_model_callback` para inspeccionar el prompt del `LlmRequest`
- Usa `before_tool_callback` para inspeccionar los argumentos de la herramienta
- Si se detecta una violación de política (ej., temas prohibidos, profanidad):
  - Retorna una respuesta predefinida (`LlmResponse` o `dict`/`Map`) para bloquear la operación
  - Opcionalmente actualiza `context.state` para registrar la violación

**Caso de Uso de Ejemplo:**
Un `before_model_callback` verifica `llm_request.contents` en busca de palabras clave sensibles y retorna un `LlmResponse` estándar "Cannot process this request" si se encuentran, evitando la llamada al LLM.

### 2. Gestión Dinámica de Estado { #dynamic-state-management }

**Descripción del Patrón:**
Leer y escribir en el estado de sesión dentro de los callbacks para hacer que el comportamiento del agente sea consciente del contexto y pasar datos entre pasos.

**Implementación:**
- Accede a `callback_context.state` o `tool_context.state`
- Las modificaciones (`state['key'] = value`) se rastrean automáticamente en el `Event.actions.state_delta` subsiguiente
- Los cambios son persistidos por el `SessionService`

**Caso de Uso de Ejemplo:**
Un `after_tool_callback` guarda un `transaction_id` del resultado de la herramienta en `tool_context.state['last_transaction_id']`. Un `before_agent_callback` posterior podría leer `state['user_tier']` para personalizar el saludo del agente.

### 3. Registro y Monitoreo { #logging-and-monitoring }

**Descripción del Patrón:**
Agregar registro detallado en puntos específicos del ciclo de vida para observabilidad y depuración.

**Implementación:**
- Implementa callbacks (ej., `before_agent_callback`, `after_tool_callback`, `after_model_callback`)
- Imprime o envía logs estructurados que contengan:
  - Nombre del agente
  - Nombre de la herramienta
  - ID de invocación
  - Datos relevantes del contexto o argumentos

**Caso de Uso de Ejemplo:**
Mensajes de log como `INFO: [Invocation: e-123] Before Tool: search_api - Args: {'query': 'ADK'}`.

### 4. Almacenamiento en Caché { #caching }

**Descripción del Patrón:**
Evitar llamadas redundantes al LLM o ejecuciones de herramientas almacenando resultados en caché.

**Pasos de Implementación:**
1. **Antes de la Operación:** En `before_model_callback` o `before_tool_callback`:
   - Genera una clave de caché basada en la solicitud/argumentos
   - Verifica `context.state` (o una caché externa) para esta clave
   - Si se encuentra, retorna el `LlmResponse` o resultado almacenado en caché directamente

2. **Después de la Operación:** Si ocurrió un fallo de caché:
   - Usa el callback `after_` correspondiente para almacenar el nuevo resultado en la caché usando la clave

**Caso de Uso de Ejemplo:**
`before_tool_callback` para `get_stock_price(symbol)` verifica `state[f"cache:stock:{symbol}"]`. Si está presente, retorna el precio en caché; de lo contrario, permite la llamada a la API y `after_tool_callback` guarda el resultado en la clave de estado.

### 5. Modificación de Solicitud/Respuesta { #request-response-modification }

**Descripción del Patrón:**
Alterar datos justo antes de que se envíen al LLM/herramienta o justo después de que se reciban.

**Opciones de Implementación:**
- **`before_model_callback`:** Modifica `llm_request` (ej., agrega instrucciones del sistema basadas en `state`)
- **`after_model_callback`:** Modifica el `LlmResponse` retornado (ej., formatea texto, filtra contenido)
- **`before_tool_callback`:** Modifica el diccionario `args` de la herramienta (o Map en Java)
- **`after_tool_callback`:** Modifica el diccionario `tool_response` (o Map en Java)

**Caso de Uso de Ejemplo:**
`before_model_callback` agrega "User language preference: Spanish" a `llm_request.config.system_instruction` si `context.state['lang'] == 'es'`.

### 6. Omisión Condicional de Pasos { #conditional-skipping-of-steps }

**Descripción del Patrón:**
Prevenir operaciones estándar (ejecución del agente, llamada al LLM, ejecución de herramienta) basándose en ciertas condiciones.

**Implementación:**
- Retorna un valor desde un callback `before_` para omitir la ejecución normal:
  - `Content` desde `before_agent_callback`
  - `LlmResponse` desde `before_model_callback`
  - `dict` desde `before_tool_callback`
- El framework interpreta este valor retornado como el resultado para ese paso

**Caso de Uso de Ejemplo:**
`before_tool_callback` verifica `tool_context.state['api_quota_exceeded']`. Si es `True`, retorna `{'error': 'API quota exceeded'}`, evitando que la función de herramienta real se ejecute.

### 7. Acciones Específicas de Herramientas (Autenticación y Control de Resumen) { #tool-specific-actions-authentication-summarization-control }

**Descripción del Patrón:**
Manejar acciones específicas del ciclo de vida de la herramienta, principalmente autenticación y control del resumen LLM de resultados de herramientas.

**Implementación:**
Usa `ToolContext` dentro de callbacks de herramientas (`before_tool_callback`, `after_tool_callback`):

- **Autenticación:** Llama a `tool_context.request_credential(auth_config)` en `before_tool_callback` si se requieren credenciales pero no se encuentran (ej., vía `tool_context.get_auth_response` o verificación de estado). Esto inicia el flujo de autenticación.
- **Resumen:** Establece `tool_context.actions.skip_summarization = True` si la salida de diccionario cruda de la herramienta debe pasarse de vuelta al LLM o potencialmente mostrarse directamente, omitiendo el paso de resumen LLM predeterminado.

**Caso de Uso de Ejemplo:**
Un `before_tool_callback` para una API segura verifica un token de autenticación en el estado; si falta, llama a `request_credential`. Un `after_tool_callback` para una herramienta que retorna JSON estructurado podría establecer `skip_summarization = True`.

### 8. Manejo de Artefactos { #artifact-handling }

**Descripción del Patrón:**
Guardar o cargar archivos relacionados con la sesión o blobs de datos grandes durante el ciclo de vida del agente.

**Implementación:**
- **Guardar:** Usa `callback_context.save_artifact` / `await tool_context.save_artifact` para almacenar datos:
  - Reportes generados
  - Logs
  - Datos intermedios
- **Cargar:** Usa `load_artifact` para recuperar artefactos previamente almacenados
- **Rastreo:** Los cambios se rastrean vía `Event.actions.artifact_delta`

**Caso de Uso de Ejemplo:**
Un `after_tool_callback` para una herramienta "generate_report" guarda el archivo de salida usando `await tool_context.save_artifact("report.pdf", report_part)`. Un `before_agent_callback` podría cargar un artefacto de configuración usando `callback_context.load_artifact("agent_config.json")`.

## Mejores Prácticas para Callbacks

### Principios de Diseño

**Mantén el Enfoque:**
Diseña cada callback para un propósito único y bien definido (ej., solo registro, solo validación). Evita callbacks monolíticos.

**Cuida el Rendimiento:**
Los callbacks se ejecutan sincrónicamente dentro del bucle de procesamiento del agente. Evita operaciones largas o bloqueantes (llamadas de red, cómputo pesado). Descarga si es necesario, pero ten en cuenta que esto añade complejidad.

### Manejo de Errores

**Maneja Errores con Elegancia:**
- Usa bloques `try...except/catch` dentro de tus funciones de callback
- Registra errores apropiadamente
- Decide si la invocación del agente debe detenerse o intentar recuperación
- No permitas que los errores de callback bloqueen todo el proceso

### Gestión de Estado

**Gestiona el Estado Cuidadosamente:**
- Sé deliberado sobre leer y escribir en `context.state`
- Los cambios son inmediatamente visibles dentro de la invocación _actual_ y persistidos al final del procesamiento del evento
- Usa claves de estado específicas en lugar de modificar estructuras amplias para evitar efectos secundarios no deseados
- Considera usar prefijos de estado (`State.APP_PREFIX`, `State.USER_PREFIX`, `State.TEMP_PREFIX`) para claridad, especialmente con implementaciones persistentes de `SessionService`

### Confiabilidad

**Considera la Idempotencia:**
Si un callback realiza acciones con efectos secundarios externos (ej., incrementar un contador externo), diseñalo para que sea idempotente (seguro de ejecutar múltiples veces con la misma entrada) si es posible, para manejar posibles reintentos en el framework o tu aplicación.

### Pruebas y Documentación

**Prueba Exhaustivamente:**
- Prueba unitariamente tus funciones de callback usando objetos de contexto simulados
- Realiza pruebas de integración para asegurar que los callbacks funcionen correctamente dentro del flujo completo del agente

**Asegura la Claridad:**
- Usa nombres descriptivos para tus funciones de callback
- Agrega docstrings claros explicando su propósito, cuándo se ejecutan, y cualquier efecto secundario (especialmente modificaciones de estado)

**Usa el Tipo de Contexto Correcto:**
Siempre usa el tipo de contexto específico proporcionado (`CallbackContext` para agente/modelo, `ToolContext` para herramientas) para asegurar acceso a los métodos y propiedades apropiados.

Al aplicar estos patrones y mejores prácticas, puedes usar callbacks efectivamente para crear comportamientos de agente más robustos, observables y personalizados en ADK.