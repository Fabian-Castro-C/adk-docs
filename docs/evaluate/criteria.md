# Criterios de Evaluación

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span>
</div>

Esta página describe los criterios de evaluación proporcionados por ADK para evaluar el
rendimiento del agente, incluyendo la trayectoria de uso de herramientas, la calidad de la respuesta y la seguridad.

Criterio                                | Descripción                                               | Basado en Referencia | Requiere Rúbricas | LLM como Juez | Soporta [Simulación de Usuario](./user-sim.md)
:--------------------------------------- | :-------------------------------------------------------- | :-------------- | :--------------- | :------------- | :----------------------------------------
`tool_trajectory_avg_score`              | Coincidencia exacta de la trayectoria de llamadas a herramientas                       | Sí             | No               | No             | No
`response_match_score`                   | Similitud ROUGE-1 con respuesta de referencia                  | Sí             | No               | No             | No
`final_response_match_v2`                | Coincidencia semántica juzgada por LLM con respuesta de referencia           | Sí             | No               | Sí            | No
`rubric_based_final_response_quality_v1` | Calidad de respuesta final juzgada por LLM basada en rúbricas personalizadas | No              | Sí              | Sí            | Sí
`rubric_based_tool_use_quality_v1`       | Calidad de uso de herramientas juzgada por LLM basada en rúbricas personalizadas     | No              | Sí              | Sí            | Sí
`hallucinations_v1`                      | Fundamentación de la respuesta del agente juzgada por LLM contra el contexto | No              | No               | Sí            | Sí
`safety_v1`                              | Seguridad/inofensividad de la respuesta del agente                     | No              | No               | Sí            | Sí
`per_turn_user_simulator_quality_v1`     | Calidad del simulador de usuario juzgada por LLM                         | No              | No               | Sí            | Sí

## tool_trajectory_avg_score

Este criterio compara la secuencia de herramientas llamadas por el agente contra una lista
de llamadas esperadas y calcula una puntuación promedio basada en uno de los tipos de coincidencia:
`EXACT`, `IN_ORDER` o `ANY_ORDER`.

#### ¿Cuándo Usar Este Criterio?

Este criterio es ideal para escenarios donde la corrección del agente depende de las llamadas
a herramientas. Dependiendo de qué tan estrictamente deben seguirse las llamadas a herramientas, puedes elegir
entre uno de tres tipos de coincidencia: `EXACT`, `IN_ORDER` y `ANY_ORDER`.

Esta métrica es particularmente valiosa para:

*   **Pruebas de regresión:** Asegurar que las actualizaciones del agente no alteren involuntariamente
    el comportamiento de llamadas a herramientas para casos de prueba establecidos.
*   **Validación de flujo de trabajo:** Verificar que los agentes sigan correctamente flujos de trabajo predefinidos
    que requieren llamadas API específicas en un orden específico.
*   **Tareas de alta precisión:** Evaluar tareas donde pequeñas desviaciones en los parámetros de herramientas
    o el orden de llamadas pueden llevar a resultados significativamente diferentes o incorrectos.

Usa coincidencia `EXACT` cuando necesites imponer una ruta de ejecución de herramienta específica y
considerar cualquier desviación—ya sea en el nombre de la herramienta, argumentos u orden—como un fallo.

Usa coincidencia `IN_ORDER` cuando quieras asegurar que ciertas llamadas clave a herramientas ocurran en un
orden específico, pero permitir que otras llamadas a herramientas ocurran entre medio. Esta opción es
útil para asegurar si ciertas acciones clave o llamadas a herramientas ocurren y en cierto orden,
dejando algo de margen para que otras llamadas a herramientas ocurran también.

Usa coincidencia `ANY_ORDER` cuando quieras asegurar que ciertas llamadas clave a herramientas ocurran, pero
no te importe su orden, y permitir que otras llamadas a herramientas ocurran entre medio. Este criterio es útil para casos donde ocurren múltiples llamadas a herramientas sobre el
mismo concepto, como cuando tu agente emite 5 consultas de búsqueda. Realmente no te importa
el orden en que se emiten las consultas de búsqueda, siempre que ocurran.

#### Detalles

Para cada invocación que está siendo evaluada, este criterio compara la lista de
llamadas a herramientas producidas por el agente contra la lista de llamadas esperadas a herramientas usando
uno de tres tipos de coincidencia. Si las llamadas a herramientas coinciden según el tipo de coincidencia seleccionado, se otorga una puntuación de 1.0 para esa invocación, de lo contrario la puntuación es 0.0.
El valor final es el promedio de estas puntuaciones en todas las invocaciones del
caso de evaluación.

La comparación puede hacerse usando uno de los siguientes tipos de coincidencia:

*   **`EXACT`**: Requiere una coincidencia perfecta entre las llamadas a herramientas reales y esperadas,
    sin llamadas a herramientas extras o faltantes.
*   **`IN_ORDER`**: Requiere que todas las llamadas a herramientas de la lista esperada estén presentes
    en la lista real, en el mismo orden, pero permite que otras llamadas a herramientas
    aparezcan entre medio.
*   **`ANY_ORDER`**: Requiere que todas las llamadas a herramientas de la lista esperada estén
    presentes en la lista real, en cualquier orden, y permite que otras llamadas a herramientas
    aparezcan entre medio.

#### ¿Cómo Usar Este Criterio?

Por defecto, `tool_trajectory_avg_score` usa el tipo de coincidencia `EXACT`. Puedes especificar
solo un umbral para este criterio en `EvalConfig` bajo el diccionario `criteria`
para el tipo de coincidencia `EXACT`. El valor debe ser un float entre 0.0 y
1.0, que representa la puntuación mínima aceptable para que el caso de evaluación pase. Si
esperas que las trayectorias de herramientas coincidan exactamente en todas las invocaciones, debes establecer
el umbral en 1.0.

Ejemplo de entrada `EvalConfig` para coincidencia `EXACT`:

```json
{
  "criteria": {
    "tool_trajectory_avg_score": 1.0
  }
}
```

O podrías especificar el `match_type` explícitamente:

```json
{
  "criteria": {
    "tool_trajectory_avg_score": {
      "threshold": 1.0,
      "match_type": "EXACT"
    }
  }
}
```


Si quieres usar el tipo de coincidencia `IN_ORDER` o `ANY_ORDER`, puedes especificarlo vía
el campo `match_type` junto con el umbral.

Ejemplo de entrada `EvalConfig` para coincidencia `IN_ORDER`:

```json
{
  "criteria": {
    "tool_trajectory_avg_score": {
      "threshold": 1.0,
      "match_type": "IN_ORDER"
    }
  }
}
```

Ejemplo de entrada `EvalConfig` para coincidencia `ANY_ORDER`:

```json
{
  "criteria": {
    "tool_trajectory_avg_score": {
      "threshold": 1.0,
      "match_type": "ANY_ORDER"
    }
  }
}
```

#### Salida Y Cómo Interpretar

La salida es una puntuación entre 0.0 y 1.0, donde 1.0 indica una coincidencia perfecta
entre las trayectorias de herramientas reales y esperadas para todas las invocaciones, y 0.0
indica una falta total de coincidencia para todas las invocaciones. Puntuaciones más altas son mejores. Una
puntuación por debajo de 1.0 significa que para al menos una invocación, la trayectoria de llamadas a herramientas del agente
se desvió de la esperada.

## response_match_score

Este criterio evalúa si la respuesta final del agente coincide con una respuesta final
dorada/esperada usando Rouge-1.

### ¿Cuándo Usar Este Criterio?

Usa este criterio cuando necesites una medida cuantitativa de qué tan cerca está
la salida del agente de la salida esperada en términos de superposición de contenido.

### Detalles

ROUGE-1 específicamente mide la superposición de unigramas (palabras individuales) entre el
texto generado por el sistema (resumen candidato) y el texto de referencia. Básicamente verifica cuántas palabras individuales del texto de referencia están presentes
en el texto candidato. Para aprender más, consulta los detalles sobre
[ROUGE-1](https://github.com/google-research/google-research/tree/master/rouge).

### ¿Cómo Usar Este Criterio?

Puedes especificar un umbral para este criterio en `EvalConfig` bajo el
diccionario `criteria`. El valor debe ser un float entre 0.0 y 1.0, que
representa la puntuación mínima aceptable para que el caso de evaluación pase.

Ejemplo de entrada `EvalConfig`:

```json
{
  "criteria": {
    "response_match_score": 0.8
  }
}
```

### Salida Y Cómo Interpretar

El rango de valores para este criterio es [0,1], siendo más deseables los valores más cercanos a 1.

## final_response_match_v2

Este criterio evalúa si la respuesta final del agente coincide con una respuesta final
dorada/esperada usando LLM como juez.

### ¿Cuándo Usar Este Criterio?

Usa este criterio cuando necesites evaluar la corrección de la respuesta final de un agente
contra una referencia, pero requieras flexibilidad en cómo se presenta la respuesta. Es adecuado para casos donde diferentes formulaciones o formatos son
aceptables, siempre que el significado central y la información coincidan con la referencia.
Este criterio es una buena elección para evaluar respuesta a preguntas,
resumen u otras tareas generativas donde la equivalencia semántica es más
importante que la superposición léxica exacta, convirtiéndolo en una alternativa más sofisticada
a `response_match_score`.

### Detalles

Este criterio usa un Modelo de Lenguaje Grande (LLM) como juez para determinar si la
respuesta final del agente es semánticamente equivalente a la respuesta de referencia proporcionada. Está diseñado para ser más flexible que las métricas de coincidencia léxica (como
`response_match_score`), ya que se enfoca en si la respuesta del agente contiene
la información correcta, tolerando diferencias en formato, formulación
o la inclusión de detalles correctos adicionales.

Para cada invocación, el criterio solicita a un LLM juez que califique la respuesta del agente
como "válida" o "inválida" en comparación con la referencia. Esto se repite
múltiples veces para robustez (configurable vía `num_samples`), y un voto de mayoría
determina si la invocación recibe una puntuación de 1.0 (válida) o 0.0
(inválida). La puntuación final del criterio es la fracción de invocaciones consideradas válidas
en todo el caso de evaluación.

### ¿Cómo Usar Este Criterio?

Este criterio usa `LlmAsAJudgeCriterion`, permitiéndote configurar el
umbral de evaluación, el modelo juez y el número de muestras por invocación.

Ejemplo de entrada `EvalConfig`:

```json
{
  "criteria": {
    "final_response_match_v2": {
      "threshold": 0.8,
      "judge_model_options": {
            "judge_model": "gemini-2.5-flash",
            "num_samples": 5
          }
        }
    }
  }
}
```

### Salida Y Cómo Interpretar

El criterio retorna una puntuación entre 0.0 y 1.0. Una puntuación de 1.0 significa que el LLM
juez consideró la respuesta final del agente como válida para todas las invocaciones,
mientras que una puntuación más cercana a 0.0 indica que muchas respuestas fueron juzgadas como inválidas
cuando se compararon con las respuestas de referencia. Valores más altos son mejores.

## rubric_based_final_response_quality_v1

Este criterio evalúa la calidad de la respuesta final de un agente contra un
conjunto de rúbricas definidas por el usuario usando LLM como juez.

### ¿Cuándo Usar Este Criterio?

Usa este criterio cuando necesites evaluar aspectos de calidad de respuesta que van
más allá de la simple corrección o equivalencia semántica con una referencia. Es ideal
para evaluar atributos matizados como tono, estilo, utilidad o adherencia a
pautas conversacionales específicas definidas en tus rúbricas. Este criterio es
particularmente útil cuando no existe una única respuesta de referencia, o cuando la calidad
depende de múltiples factores subjetivos.

### Detalles

Este criterio proporciona una forma flexible de evaluar la calidad de la respuesta basada en
criterios específicos que defines como rúbricas. Por ejemplo, podrías definir
rúbricas para verificar si una respuesta es concisa, si infiere correctamente la intención del usuario,
o si evita jerga.

El criterio usa un LLM como juez para evaluar la respuesta final del agente
contra cada rúbrica, produciendo un veredicto `yes` (1.0) o `no` (0.0) para cada.
Como otras métricas basadas en LLM, muestrea el modelo juez múltiples veces por
invocación y usa un voto de mayoría para determinar la puntuación para cada rúbrica en
esa invocación. La puntuación general para una invocación es el promedio de sus
puntuaciones de rúbricas. La puntuación final del criterio para el caso de evaluación es el promedio de
estas puntuaciones generales en todas las invocaciones.

### ¿Cómo Usar Este Criterio?

Este criterio usa `RubricsBasedCriterion`, que requiere una lista de rúbricas a
ser proporcionada en el `EvalConfig`. Cada rúbrica debe definirse con un ID único
y su contenido.

Ejemplo de entrada `EvalConfig`:

```json
{
  "criteria": {
    "rubric_based_final_response_quality_v1": {
      "threshold": 0.8,
      "judge_model_options": {
        "judge_model": "gemini-2.5-flash",
        "num_samples": 5
      },
      "rubrics": [
        {
          "rubric_id": "conciseness",
          "rubric_content": {
            "text_property": "The agent's response is direct and to the point."
          }
        },
        {
          "rubric_id": "intent_inference",
          "rubric_content": {
            "text_property": "The agent's response accurately infers the user's underlying goal from ambiguous queries."
          }
        }
      ]
    }
  }
}
```

### Salida Y Cómo Interpretar

El criterio genera una puntuación general entre 0.0 y 1.0, donde 1.0 indica
que las respuestas del agente satisfacieron todas las rúbricas en todas las invocaciones, y 0.0
indica que no se satisficieron rúbricas. Los resultados también incluyen
puntuaciones detalladas por rúbrica para cada invocación. Valores más altos son mejores.

## rubric_based_tool_use_quality_v1

Este criterio evalúa la calidad del uso de herramientas de un agente contra un
conjunto de rúbricas definidas por el usuario usando LLM como juez.

### ¿Cuándo Usar Este Criterio?

Usa este criterio cuando necesites evaluar *cómo* un agente usa herramientas, en lugar
de solo *si* la respuesta final es correcta. Es ideal para evaluar si
el agente seleccionó la herramienta correcta, usó los parámetros correctos o siguió una
secuencia específica de llamadas a herramientas. Esto es útil para validar procesos de razonamiento
del agente, depurar errores en el uso de herramientas y asegurar adherencia a
flujos de trabajo prescritos, especialmente en casos donde múltiples rutas de uso de herramientas podrían llevar a una
respuesta final similar pero solo una ruta se considera correcta.

### Detalles

Este criterio proporciona una forma flexible de evaluar el uso de herramientas basada en
reglas específicas que defines como rúbricas. Por ejemplo, podrías definir rúbricas para verificar
si se llamó una herramienta específica, si sus parámetros eran correctos o si las herramientas se
llamaron en un orden particular.

El criterio usa un LLM como juez para evaluar las llamadas a herramientas del agente y
respuestas contra cada rúbrica, produciendo un veredicto `yes` (1.0) o `no` (0.0) para
cada. Como otras métricas basadas en LLM, muestrea el modelo juez múltiples veces
por invocación y usa un voto de mayoría para determinar la puntuación para cada rúbrica
en esa invocación. La puntuación general para una invocación es el promedio de sus
puntuaciones de rúbricas. La puntuación final del criterio para el caso de evaluación es el promedio de
estas puntuaciones generales en todas las invocaciones.

### ¿Cómo Usar Este Criterio?

Este criterio usa `RubricsBasedCriterion`, que requiere una lista de rúbricas a
ser proporcionada en el `EvalConfig`. Cada rúbrica debe definirse con un ID único
y su contenido, describiendo un aspecto específico del uso de herramientas a evaluar.

Ejemplo de entrada `EvalConfig`:

```json
{
  "criteria": {
    "rubric_based_tool_use_quality_v1": {
      "threshold": 1.0,
      "judge_model_options": {
        "judge_model": "gemini-2.5-flash",
        "num_samples": 5
      },
      "rubrics": [
        {
          "rubric_id": "geocoding_called",
          "rubric_content": {
            "text_property": "The agent calls the GeoCoding tool before calling the GetWeather tool."
          }
        },
        {
          "rubric_id": "getweather_called",
          "rubric_content": {
            "text_property": "The agent calls the GetWeather tool with coordinates derived from the user's location."
          }
        }
      ]
    }
  }
}
```

### Salida Y Cómo Interpretar

El criterio genera una puntuación general entre 0.0 y 1.0, donde 1.0 indica
que el uso de herramientas del agente satisfizo todas las rúbricas en todas las invocaciones, y
0.0 indica que no se satisficieron rúbricas. Los resultados también incluyen
puntuaciones detalladas por rúbrica para cada invocación. Valores más altos son mejores.

## hallucinations_v1

Este criterio evalúa si una respuesta del modelo contiene afirmaciones falsas,
contradictorias o no respaldadas.

### ¿Cuándo Usar Este Criterio?

Usa este criterio para asegurar que la respuesta del agente esté fundamentada en el
contexto proporcionado (ej., salidas de herramientas, consulta del usuario, instrucciones) y no contenga
alucinaciones.

### Detalles

Este criterio evalúa si una respuesta del modelo contiene afirmaciones falsas,
contradictorias o no respaldadas basándose en el contexto que incluye
instrucciones del desarrollador, prompt del usuario, definiciones de herramientas e invocaciones de herramientas y sus
resultados. Usa LLM como juez y sigue un proceso de dos pasos:

1.  **Segmentador**: Segmenta la respuesta del agente en oraciones individuales.
2.  **Validador de Oraciones**: Evalúa cada oración segmentada contra el
    contexto proporcionado para fundamentación. Cada oración se etiqueta como `supported`,
    `unsupported`, `contradictory`, `disputed` o `not_applicable`.

La métrica calcula una Puntuación de Precisión: el porcentaje de oraciones que son
`supported` o `not_applicable`. Por defecto, solo se evalúa la respuesta final. Si `evaluate_intermediate_nl_responses` se establece en true en el
criterio, también se evalúan las respuestas intermedias en lenguaje natural de los agentes.

### ¿Cómo Usar Este Criterio?

Este criterio usa `HallucinationsCriterion`, permitiéndote configurar el
umbral de evaluación, el modelo juez, el número de muestras por invocación y
si evaluar respuestas intermedias en lenguaje natural.

Ejemplo de entrada `EvalConfig`:

```json
{
  "criteria": {
    "hallucinations_v1": {
      "threshold": 0.8,
      "judge_model_options": {
            "judge_model": "gemini-2.5-flash",
          },
      "evaluate_intermediate_nl_responses": true
    }
  }
}

```

### Salida Y Cómo Interpretar

El criterio retorna una puntuación entre 0.0 y 1.0. Una puntuación de 1.0 significa que todas
las oraciones en la respuesta del agente están fundamentadas en el contexto, mientras que una puntuación más cercana
a 0.0 indica que muchas oraciones son falsas, contradictorias o no respaldadas.
Valores más altos son mejores.

## safety_v1

Este criterio evalúa la seguridad (inofensividad) de la Respuesta de un Agente.

### ¿Cuándo Usar Este Criterio?

Este criterio debe usarse cuando necesites asegurar que las respuestas del agente
cumplan con las pautas de seguridad y no produzcan contenido dañino o inapropiado. Es esencial para aplicaciones de cara al usuario o cualquier sistema donde
la seguridad de la respuesta sea una prioridad.

### Detalles

Este criterio evalúa si la respuesta del agente contiene algún contenido dañino, como discurso de odio, acoso o información peligrosa. A diferencia de otras
métricas implementadas nativamente dentro de ADK, `safety_v1` delega la evaluación al
SDK de Vertex AI General AI Eval.

### ¿Cómo Usar Este Criterio?

Usar este criterio requiere un Proyecto de Google Cloud. Debes tener
las variables de entorno `GOOGLE_CLOUD_PROJECT` y `GOOGLE_CLOUD_LOCATION` establecidas,
típicamente en un archivo `.env` en el directorio de tu agente, para que el SDK de Vertex AI
funcione correctamente.

Puedes especificar un umbral para este criterio en `EvalConfig` bajo el
diccionario `criteria`. El valor debe ser un float entre 0.0 y 1.0,
representando la puntuación de seguridad mínima para que una respuesta se considere aprobada.

Ejemplo de entrada `EvalConfig`:

```json
{
  "criteria": {
    "safety_v1": 0.8
  }
}
```

### Salida Y Cómo Interpretar

El criterio retorna una puntuación entre 0.0 y 1.0. Puntuaciones más cercanas a 1.0 indican
que la respuesta es segura, mientras que puntuaciones más cercanas a 0.0 indican potenciales problemas de seguridad.

## per_turn_user_simulator_quality_v1

Este criterio evalúa si un simulador de usuario es fiel a un plan de
conversación.

#### ¿Cuándo Usar Este Criterio?

Usa este criterio cuando necesites evaluar un simulador de usuario en una conversación
de múltiples turnos. Está diseñado para evaluar si el simulador sigue el
plan de conversación definido en el `ConversationScenario`.

#### Detalles

Este criterio determina si un simulador de usuario sigue un `ConversationScenario` definido en una conversación de múltiples turnos.

Para el primer turno, este criterio verifica si la respuesta del simulador de usuario coincide con el
`starting_prompt` en el `ConversationScenario`. Para turnos subsecuentes, usa
LLM como juez para evaluar si la respuesta del usuario sigue el `conversation_plan`
en el `ConversationScenario`.

#### ¿Cómo Usar Este Criterio?

Este criterio te permite configurar el umbral de evaluación, el modelo juez
y el número de muestras por invocación. El criterio también te permite especificar una
`stop_signal`, que señala al juez LLM que la conversación se completó.
Para mejores resultados, usa la señal de parada en `LlmBackedUserSimulator`.

Ejemplo de entrada `EvalConfig`:

```json
{
  "criteria": {
    "per_turn_user_simulator_quality_v1": {
      "threshold": 1.0,
      "judge_model_options": {
        "judge_model": "gemini-2.5-flash",
        "num_samples": 5
      },
      "stop_signal": "</finished>"
    }
  }
}
```

#### Salida Y Cómo Interpretar

El criterio retorna una puntuación entre 0.0 y 1.0, representando la fracción de
turnos en los que la respuesta del simulador de usuario fue juzgada como válida según
el escenario de conversación. Una puntuación de 1.0 indica que el simulador se comportó
como se esperaba en todos los turnos, mientras que una puntuación más cercana a 0.0 indica que el
simulador se desvió en muchos turnos. Valores más altos son mejores.