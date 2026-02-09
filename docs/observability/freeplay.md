# Observabilidad y Evaluación de Agentes con Freeplay

[Freeplay](https://freeplay.ai/) proporciona un flujo de trabajo de extremo a extremo para construir
y optimizar agentes de IA, y puede integrarse con ADK. Con Freeplay todo tu
equipo puede colaborar fácilmente para iterar sobre las instrucciones del agente (prompts),
experimentar con y comparar diferentes modelos y cambios de agentes, ejecutar evaluaciones tanto
offline como online para medir la calidad, monitorear producción y revisar datos manualmente.

Beneficios clave de Freeplay:

* **Observabilidad simple** - enfocada en agentes, llamadas a LLM y llamadas a herramientas para una fácil revisión humana
* **Evaluaciones online/calificadores automatizados** - para detección de errores en producción
* **Evaluaciones offline y comparación de experimentos** - para probar cambios antes de desplegar
* **Gestión de prompts** - soporta enviar cambios directamente desde el playground de Freeplay al código
* **Flujo de trabajo de revisión humana** - para colaboración en análisis de errores y anotación de datos
* **UI poderosa** - hace posible que expertos del dominio colaboren estrechamente con ingenieros

Freeplay y ADK se complementan entre sí. ADK te brinda un framework de orquestación de agentes
poderoso y expresivo mientras que Freeplay se conecta para observabilidad, gestión de prompts,
evaluación y pruebas. Una vez que te integras con Freeplay, puedes
actualizar prompts y evaluaciones desde la UI de Freeplay o desde el código, para que cualquiera en
tu equipo pueda contribuir.

<iframe width="672" height="378" src="https://www.youtube.com/embed/AV2zCkp4aYM?si=HVuOJFLMEkkpocF7" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Comenzando

A continuación se presenta una guía para comenzar con Freeplay y ADK. También puedes encontrar un
repositorio de ejemplo completo de un agente ADK
[aquí](https://github.com/228Labs/freeplay-google-demo).

### Crea una Cuenta de Freeplay

Regístrate para obtener una [cuenta gratuita de Freeplay](https://freeplay.ai/signup).

Después de crear una cuenta, puedes definir las siguientes variables de entorno:

```
FREEPLAY_PROJECT_ID=
FREEPLAY_API_KEY=
FREEPLAY_API_URL=
```

### Usa la Biblioteca ADK de Freeplay

Instala la biblioteca ADK de Freeplay:

```
pip install freeplay-python-adk
```

Freeplay capturará automáticamente los logs OTel de tu aplicación ADK cuando
inicialices la observabilidad:

```python
from freeplay_python_adk.client import FreeplayADK
FreeplayADK.initialize_observability()
```

También querrás pasar el plugin de Freeplay a tu App:

```python
from app.agent import root_agent
from freeplay_python_adk.freeplay_observability_plugin import FreeplayObservabilityPlugin
from google.adk.runners import App

app = App(
    name="app",
    root_agent=root_agent,
    plugins=[FreeplayObservabilityPlugin()],
)

__all__ = ["app"]
```

Ahora puedes usar ADK como lo harías normalmente, y verás logs fluyendo a
Freeplay en la sección de Observabilidad.

## Observabilidad

La función de Observabilidad de Freeplay te brinda una vista clara de cómo se está
comportando tu agente en producción. Puedes profundizar en trazas individuales de agentes para
entender cada paso y diagnosticar problemas:

![Trace detail](https://raw.githubusercontent.com/freeplayai/freeplay-google-demo/refs/heads/main/docs/images/trace_detail.png)

También puedes usar la funcionalidad de filtrado de Freeplay para buscar y filtrar los
datos a través de cualquier segmento de interés:

![Filter](https://raw.githubusercontent.com/freeplayai/freeplay-google-demo/refs/heads/main/docs/images/filter.png)

## Gestión de Prompts (opcional)

Freeplay ofrece
[gestión de prompts nativa](https://docs.freeplay.ai/docs/managing-prompts),
que simplifica el proceso de versionar y probar diferentes versiones de prompts.
Te permite experimentar con cambios a las instrucciones de agentes ADK en la
UI de Freeplay, probar diferentes modelos y enviar actualizaciones directamente a tu código,
similar a una feature flag.

Para aprovechar las capacidades de gestión de prompts de Freeplay junto con ADK, querrás
usar el wrapper de agente ADK de Freeplay. `FreeplayLLMAgent` extiende la clase base
`LlmAgent` de ADK, así que en lugar de tener que codificar directamente tus prompts como
instrucciones del agente, puedes versionar prompts en la aplicación Freeplay.

Primero define un prompt en Freeplay yendo a Prompts -> Create prompt template:

![Prompt](https://raw.githubusercontent.com/freeplayai/freeplay-google-demo/refs/heads/main/docs/images/prompt.png)

Al crear tu plantilla de prompt necesitarás agregar 3 elementos, como se describe
en las siguientes secciones:

### Mensaje del Sistema

Esto corresponde a la sección "instructions" en tu código.

### Variable de Contexto del Agente

Agregar lo siguiente en la parte inferior de tu mensaje del sistema creará una variable
para que el contexto del agente en curso sea pasado:

```python
{{agent_context}}
```

### Bloque de Historial

Haz clic en nuevo mensaje y cambia el rol a 'history'. Esto asegurará que los
mensajes pasados se pasen cuando estén presentes.

![Prompt Editor](https://raw.githubusercontent.com/freeplayai/freeplay-google-demo/refs/heads/main/docs/images/prompt_editor.png)

Ahora en tu código puedes usar el ```FreeplayLLMAgent```:

```python
from freeplay_python_adk.client import FreeplayADK
from freeplay_python_adk.freeplay_llm_agent import (
    FreeplayLLMAgent,
)

FreeplayADK.initialize_observability()

root_agent = FreeplayLLMAgent(
    name="social_product_researcher",
    tools=[tavily_search],
)
```

Cuando se invoca ```social_product_researcher```, el prompt será
recuperado de Freeplay y formateado con las variables de entrada apropiadas.

## Evaluación

Freeplay te permite definir, versionar y ejecutar
[evaluaciones](https://docs.freeplay.ai/docs/evaluations) desde la aplicación web
de Freeplay. Puedes definir evaluaciones para cualquiera de tus prompts o agentes
yendo a Evaluations -> "New evaluation".

![Creating a new evaluation in Freeplay](https://raw.githubusercontent.com/freeplayai/freeplay-google-demo/refs/heads/main/docs/images/eval_create.png)

Estas evaluaciones pueden configurarse para ejecutarse tanto para monitoreo online como para
evaluación offline. Los conjuntos de datos para evaluación offline pueden cargarse a Freeplay
o guardarse desde ejemplos de logs.

## Gestión de Conjuntos de Datos

A medida que obtienes datos fluyendo hacia Freeplay, puedes usar estos logs para comenzar a construir
[conjuntos de datos](https://docs.freeplay.ai/docs/datasets) para probar de manera
repetida. Usa logs de producción para crear conjuntos de datos dorados o colecciones de
casos de fallo que puedes usar para probar a medida que realizas cambios.

![Save test case](https://raw.githubusercontent.com/freeplayai/freeplay-google-demo/refs/heads/main/docs/images/save_test_case.png)

## Pruebas por Lotes

A medida que iteras sobre tu agente, puedes ejecutar pruebas por lotes (es decir, experimentos
offline) tanto a nivel de
[prompt](https://docs.freeplay.ai/docs/component-level-test-runs) como
[extremo a extremo](https://docs.freeplay.ai/docs/end-to-end-test-runs) del agente.
Esto te permite comparar múltiples modelos diferentes o cambios de prompts y
cuantificar cambios cara a cara a través de la ejecución completa de tu agente.

[Aquí](https://github.com/freeplayai/freeplay-google-demo/blob/main/examples/example_test_run.py)
hay un ejemplo de código para ejecutar una prueba por lotes en Freeplay con ADK.

## Regístrate ahora

Ve a [Freeplay](https://freeplay.ai/) para registrarte y obtener una cuenta, y revisa una integración completa de Freeplay <> ADK [aquí](https://github.com/freeplayai/freeplay-google-demo/tree/main)