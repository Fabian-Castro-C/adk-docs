# Configuración de Tiempo de Ejecución

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

`RunConfig` define el comportamiento y las opciones de tiempo de ejecución para agentes en ADK. Controla
la configuración de voz y transmisión, llamadas a funciones, guardado de artefactos y límites en
llamadas LLM.

Al construir una ejecución de agente, puedes pasar un `RunConfig` para personalizar cómo el
agente interactúa con los modelos, maneja el audio y transmite respuestas. Por defecto,
no se habilita transmisión y las entradas no se retienen como artefactos. Usa `RunConfig`
para sobrescribir estos valores predeterminados.

## Definición de Clase

La clase `RunConfig` contiene parámetros de configuración para el comportamiento de tiempo de ejecución de un agente.

-   Python ADK usa Pydantic para esta validación.
-   Go ADK tiene estructuras mutables por defecto.
-   Java ADK típicamente usa clases de datos inmutables.


- TypeScript ADK usa una interfaz estándar, con seguridad de tipos proporcionada por el compilador de TypeScript.

=== "Python"

    ```python
    class RunConfig(BaseModel):
        """Configuraciones para el comportamiento de tiempo de ejecución de los agentes."""

        model_config = ConfigDict(
            extra='forbid',
        )

        speech_config: Optional[types.SpeechConfig] = None
        response_modalities: Optional[list[str]] = None
        save_input_blobs_as_artifacts: bool = False
        support_cfc: bool = False
        streaming_mode: StreamingMode = StreamingMode.NONE
        output_audio_transcription: Optional[types.AudioTranscriptionConfig] = None
        max_llm_calls: int = 500
    ```

=== "TypeScript"

    ```typescript
    export interface RunConfig {
      speechConfig?: SpeechConfig;
      responseModalities?: Modality[];
      saveInputBlobsAsArtifacts: boolean;
      supportCfc: boolean;
      streamingMode: StreamingMode;
      outputAudioTranscription?: AudioTranscriptionConfig;
      maxLlmCalls: number;
      // ... y otras propiedades
    }

    export enum StreamingMode {
      NONE = 'none',
      SSE = 'sse',
      BIDI = 'bidi',
    }
    ```

=== "Go"

    ```go
    type StreamingMode string

    const (
    	StreamingModeNone StreamingMode = "none"
    	StreamingModeSSE  StreamingMode = "sse"
    )

    // RunConfig controla el comportamiento de tiempo de ejecución.
    type RunConfig struct {
    	// Modo de transmisión, None o StreamingMode.SSE.
    	StreamingMode StreamingMode
    	// Si se deben guardar o no los blobs de entrada como artefactos
    	SaveInputBlobsAsArtifacts bool
    }
    ```

=== "Java"

    ```java
    public abstract class RunConfig {

      public enum StreamingMode {
        NONE,
        SSE,
        BIDI
      }

      public abstract @Nullable SpeechConfig speechConfig();

      public abstract ImmutableList<Modality> responseModalities();

      public abstract boolean saveInputBlobsAsArtifacts();

      public abstract @Nullable AudioTranscriptionConfig outputAudioTranscription();

      public abstract int maxLlmCalls();

      // ...
    }
    ```

## Parámetros de Tiempo de Ejecución

| Parámetro                       | Tipo Python                                  | Tipo TypeScript                       | Tipo Go         | Tipo Java                                             | Predeterminado (Py / TS / Go / Java)                                                           | Descripción                                                                                                                                                                                   |
| :------------------------------ | :------------------------------------------- | :------------------------------------ |:----------------|:------------------------------------------------------| :--------------------------------------------------------------------------------------------- |:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `speech_config`                 | `Optional[types.SpeechConfig]`               | `SpeechConfig` (opcional)             | N/A             | `SpeechConfig` (nullable via `@Nullable`)             | `None` / `undefined`/ N/A / `null`                                                             | Configura la síntesis de voz (voz, idioma) usando el tipo `SpeechConfig`.                                                                                                                    |
| `response_modalities`           | `Optional[list[str]]`                        | `Modality[]` (opcional)               | N/A             | `ImmutableList<Modality>`                             | `None` / `undefined` / N/A / Empty `ImmutableList`                                             | Lista de modalidades de salida deseadas (ej., Python: `["TEXT", "AUDIO"]`; Java/TS: usa objetos `Modality` estructurados).                                                                   |
| `save_input_blobs_as_artifacts` | `bool`                                       | `boolean`                             | `bool`          | `boolean`                                             | `False` / `false` / `false` / `false`                                                          | Si es `true`, guarda blobs de entrada (ej., archivos cargados) como artefactos de ejecución para depuración/auditoría.                                                                       |
| `streaming_mode`                | `StreamingMode`                              | `StreamingMode`                       | `StreamingMode` | `StreamingMode`                                       | `StreamingMode.NONE` / `StreamingMode.NONE` / `agent.StreamingModeNone` / `StreamingMode.NONE` | Establece el comportamiento de transmisión: `NONE` (predeterminado), `SSE` (eventos enviados por el servidor), o `BIDI` (bidireccional).                                                     |
| `output_audio_transcription`    | `Optional[types.AudioTranscriptionConfig]`   | `AudioTranscriptionConfig` (opcional) | N/A             | `AudioTranscriptionConfig` (nullable via `@Nullable`) | `None` / `undefined` / N/A / `null`                                                            | Configura la transcripción de la salida de audio generada usando el tipo `AudioTranscriptionConfig`.                                                                                         |
| `max_llm_calls`                 | `int`                                        | `number`                              | N/A             | `int`                                                 | `500` / `500` / N/A / `500`                                                                    | Limita el total de llamadas LLM por ejecución. `0` o negativo significa ilimitado. Exceder los límites del lenguaje (ej. `sys.maxsize`, `Number.MAX_SAFE_INTEGER`) genera un error.         |
| `support_cfc`                   | `bool`                                       | `boolean`                             | N/A             | `bool`                                                | `False` / `false` / N/A / `false`                                                              | **Python/TypeScript:** Habilita Llamadas a Funciones Composicionales. Requiere `streaming_mode=SSE` y usa la API LIVE. **Experimental.**                                                     |


### `speech_config`

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

!!! Note
    La interfaz o definición de `SpeechConfig` es la misma, independientemente del lenguaje.

Configuración de voz para agentes en vivo con capacidades de audio. La
clase `SpeechConfig` tiene la siguiente estructura:

```python
class SpeechConfig(_common.BaseModel):
    """La configuración de generación de voz."""

    voice_config: Optional[VoiceConfig] = Field(
        default=None,
        description="""La configuración para el hablante a usar.""",
    )
    language_code: Optional[str] = Field(
        default=None,
        description="""Código de idioma (ISO 639. ej. en-US) para la síntesis de voz.
        Solo disponible para la API Live.""",
    )
```

El parámetro `voice_config` usa la clase `VoiceConfig`:

```python
class VoiceConfig(_common.BaseModel):
    """La configuración para la voz a usar."""

    prebuilt_voice_config: Optional[PrebuiltVoiceConfig] = Field(
        default=None,
        description="""La configuración para el hablante a usar.""",
    )
```

Y `PrebuiltVoiceConfig` tiene la siguiente estructura:

```python
class PrebuiltVoiceConfig(_common.BaseModel):
    """La configuración para el hablante predefinido a usar."""

    voice_name: Optional[str] = Field(
        default=None,
        description="""El nombre de la voz predefinida a usar.""",
    )
```

Estas clases de configuración anidadas te permiten especificar:

* `voice_config`: El nombre de la voz predefinida a usar (en `PrebuiltVoiceConfig`)
* `language_code`: Código de idioma ISO 639 (ej., "en-US") para la síntesis de voz

Al implementar agentes habilitados por voz, configura estos parámetros para controlar
cómo suena tu agente cuando habla.

### `response_modalities`

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Define las modalidades de salida para el agente. Si no se establece, el valor predeterminado es AUDIO.
Las modalidades de respuesta determinan cómo el agente se comunica con los usuarios a través de
varios canales (ej., texto, audio).

### `save_input_blobs_as_artifacts`

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Cuando está habilitado, los blobs de entrada se guardarán como artefactos durante la ejecución del agente.
Esto es útil para propósitos de depuración y auditoría, permitiendo a los desarrolladores revisar
los datos exactos recibidos por los agentes.

### `support_cfc`

<div class="language-support-tag" title="This feature is an experimental preview release.">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-preview">Experimental</span>
</div>

Habilita el soporte para Llamadas a Funciones Composicionales (CFC). Solo aplicable cuando se usa
StreamingMode.SSE. Cuando está habilitado, se invocará la API LIVE ya que solo ella
soporta la funcionalidad CFC.

!!! example "Lanzamiento experimental"

    La característica `support_cfc` es experimental y su API o comportamiento podría
    cambiar en versiones futuras.

### `streaming_mode`

<div class="language-support-tag" title="This feature is an experimental preview release.">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-go">Go v0.1.0</span>
</div>

Configura el comportamiento de transmisión del agente. Valores posibles:

* `StreamingMode.NONE`: Sin transmisión; respuestas entregadas como unidades completas
* `StreamingMode.SSE`: Transmisión de Eventos Enviados por el Servidor; transmisión unidireccional del servidor al cliente
* `StreamingMode.BIDI`: Transmisión bidireccional; comunicación simultánea en ambas direcciones

Los modos de transmisión afectan tanto el rendimiento como la experiencia del usuario. La transmisión SSE permite a los usuarios ver respuestas parciales a medida que se generan, mientras que la transmisión BIDI habilita experiencias interactivas en tiempo real.

### `output_audio_transcription`

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Configuración para transcribir salidas de audio de agentes en vivo con capacidad de
respuesta de audio. Esto habilita la transcripción automática de respuestas de audio para
accesibilidad, mantenimiento de registros y aplicaciones multimodales.

### `max_llm_calls`

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

Establece un límite en el número total de llamadas LLM para una ejecución de agente dada.

* Valores mayores que 0 y menores que `sys.maxsize`: Impone un límite en las llamadas LLM
* Valores menores o iguales a 0: Permite llamadas LLM ilimitadas *(no recomendado para producción)*

Este parámetro previene el uso excesivo de API y procesos potencialmente descontrolados.
Dado que las llamadas LLM a menudo incurren en costos y consumen recursos, establecer
límites apropiados es crucial.

## Reglas de Validación

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

La clase `RunConfig` valida sus parámetros para asegurar el funcionamiento apropiado del agente. Mientras que Python ADK usa `Pydantic` para validación automática de tipos, Java y TypeScript ADK dependen de sus sistemas de tipos estáticos y pueden incluir verificaciones explícitas en el constructor de `RunConfig`.
Para el parámetro `max_llm_calls` específicamente:

1. Valores extremadamente grandes (como `sys.maxsize` en Python, `Integer.MAX_VALUE` en Java, o `Number.MAX_SAFE_INTEGER` en TypeScript) típicamente no están permitidos para prevenir problemas.

2. Valores de cero o menos usualmente desencadenarán una advertencia sobre interacciones LLM ilimitadas.

### Configuración básica de tiempo de ejecución

=== "Python"

    ```python
    from google.genai.adk import RunConfig, StreamingMode

    config = RunConfig(
        streaming_mode=StreamingMode.NONE,
        max_llm_calls=100
    )
    ```

=== "TypeScript"

    ```typescript
    import { RunConfig, StreamingMode } from '@google/adk';

    const config: RunConfig = {
      streamingMode: StreamingMode.NONE,
      maxLlmCalls: 100,
    };
    ```

=== "Go"

    ```go
    import "google.golang.org/adk/agent"

    config := agent.RunConfig{
        StreamingMode: agent.StreamingModeNone,
    }
    ```

=== "Java"

    ```java
    import com.google.adk.agents.RunConfig;
    import com.google.adk.agents.RunConfig.StreamingMode;

    RunConfig config = RunConfig.builder()
            .setStreamingMode(StreamingMode.NONE)
            .setMaxLlmCalls(100)
            .build();
    ```

Esta configuración crea un agente sin transmisión con un límite de 100 llamadas LLM,
adecuado para agentes simples orientados a tareas donde las respuestas completas son
preferibles.

### Habilitando transmisión

=== "Python"

    ```python
    from google.genai.adk import RunConfig, StreamingMode

    config = RunConfig(
        streaming_mode=StreamingMode.SSE,
        max_llm_calls=200
    )
    ```

=== "TypeScript"

    ```typescript
    import { RunConfig, StreamingMode } from '@google/adk';

    const config: RunConfig = {
      streamingMode: StreamingMode.SSE,
      maxLlmCalls: 200,
    };
    ```

=== "Go"

    ```go
    import "google.golang.org/adk/agent"

    config := agent.RunConfig{
        StreamingMode: agent.StreamingModeSSE,
    }
    ```

=== "Java"

    ```java
    import com.google.adk.agents.RunConfig;
    import com.google.adk.agents.RunConfig.StreamingMode;

    RunConfig config = RunConfig.builder()
        .setStreamingMode(StreamingMode.SSE)
        .setMaxLlmCalls(200)
        .build();
    ```

Usar transmisión SSE permite a los usuarios ver las respuestas a medida que se generan,
proporcionando una sensación más receptiva para chatbots y asistentes.

### Habilitando soporte de voz

=== "Python"

    ```python
    from google.genai.adk import RunConfig, StreamingMode
    from google.genai import types

    config = RunConfig(
        speech_config=types.SpeechConfig(
            language_code="en-US",
            voice_config=types.VoiceConfig(
                prebuilt_voice_config=types.PrebuiltVoiceConfig(
                    voice_name="Kore"
                )
            ),
        ),
        response_modalities=["AUDIO", "TEXT"],
        save_input_blobs_as_artifacts=True,
        support_cfc=True,
        streaming_mode=StreamingMode.SSE,
        max_llm_calls=1000,
    )
    ```

=== "TypeScript"

    ```typescript
    import { RunConfig, StreamingMode } from '@google/adk';

    const config: RunConfig = {
        speechConfig: {
            languageCode: "en-US",
            voiceConfig: {
                prebuiltVoiceConfig: {
                    voiceName: "Kore"
                }
            },
        },
        responseModalities: [
          { modality: "AUDIO" },
          { modality: "TEXT" }
        ],
        saveInputBlobsAsArtifacts: true,
        supportCfc: true,
        streamingMode: StreamingMode.SSE,
        maxLlmCalls: 1000,
    };
    ```

=== "Java"

    ```java
    import com.google.adk.agents.RunConfig;
    import com.google.adk.agents.RunConfig.StreamingMode;
    import com.google.common.collect.ImmutableList;
    import com.google.genai.types.Content;
    import com.google.genai.types.Modality;
    import com.google.genai.types.Part;
    import com.google.genai.types.PrebuiltVoiceConfig;
    import com.google.genai.types.SpeechConfig;
    import com.google.genai.types.VoiceConfig;

    RunConfig runConfig =
        RunConfig.builder()
            .setStreamingMode(StreamingMode.SSE)
            .setMaxLlmCalls(1000)
            .setSaveInputBlobsAsArtifacts(true)
            .setResponseModalities(ImmutableList.of(new Modality("AUDIO"), new Modality("TEXT")))
            .setSpeechConfig(
                SpeechConfig.builder()
                    .voiceConfig(
                        VoiceConfig.builder()
                            .prebuiltVoiceConfig(
                                PrebuiltVoiceConfig.builder().voiceName("Kore").build())
                            .build())
                    .languageCode("en-US")
                    .build())
            .build();
    ```

Este ejemplo completo configura un agente con:

* Capacidades de voz usando la voz "Kore" (inglés estadounidense)
* Modalidades de salida tanto de audio como de texto
* Guardado de artefactos para blobs de entrada (útil para depuración)
* Soporte CFC experimental habilitado **(Python y TypeScript)**
* Transmisión SSE para interacción receptiva
* Un límite de 1000 llamadas LLM

### Habilitando Soporte CFC

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">Typescript v0.2.0</span><span class="lst-preview">Experimental</span>
</div>

=== "Python"

    ```python
    from google.genai.adk import RunConfig, StreamingMode

    config = RunConfig(
        streaming_mode=StreamingMode.SSE,
        support_cfc=True,
        max_llm_calls=150
    )
    ```

=== "TypeScript"

    ```typescript
    import { RunConfig, StreamingMode } from '@google/adk';

    const config: RunConfig = {
        streamingMode: StreamingMode.SSE,
        supportCfc: true,
        maxLlmCalls: 150,
    };
    ```

Habilitar Llamadas a Funciones Composicionales (CFC) crea un agente que puede
ejecutar dinámicamente funciones basadas en salidas del modelo, poderoso para aplicaciones
que requieren flujos de trabajo complejos.

!!! example "Lanzamiento experimental"

    La característica de transmisión de Llamadas a Funciones Composicionales (CFC) es un
    lanzamiento experimental.