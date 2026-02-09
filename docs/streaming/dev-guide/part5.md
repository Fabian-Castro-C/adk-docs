# Parte 5: Cómo Usar Audio, Imagen y Video

Esta sección cubre las capacidades de audio, imagen y video en la integración de Live API de ADK, incluyendo modelos compatibles, arquitecturas de modelos de audio, especificaciones y mejores prácticas para implementar características de voz y video.

## Cómo Usar Audio

Las capacidades de audio de Live API permiten conversaciones de voz naturales con latencia inferior a un segundo mediante transmisión de audio bidireccional. Esta sección cubre cómo enviar entrada de audio al modelo y recibir respuestas de audio, incluyendo requisitos de formato, mejores prácticas de transmisión y patrones de implementación del lado del cliente.

### Enviando Entrada de Audio

**Requisitos de Formato de Audio:**

Antes de llamar a `send_realtime()`, asegúrate de que tus datos de audio ya estén en el formato correcto:

- **Formato**: PCM de 16 bits (entero con signo)
- **Tasa de Muestreo**: 16,000 Hz (16kHz)
- **Canales**: Mono (canal único)

ADK no realiza conversión de formato de audio. Enviar audio en formatos incorrectos resultará en mala calidad o errores.

```python title='Demo implementation: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py#L181-L184" target="_blank">main.py:181-184</a>'
audio_blob = types.Blob(
    mime_type="audio/pcm;rate=16000",
    data=audio_data
)
live_request_queue.send_realtime(audio_blob)
```

#### Mejores Prácticas para Enviar Entrada de Audio

1. **Transmisión en Fragmentos**: Envía audio en fragmentos pequeños para baja latencia. Elige el tamaño del fragmento según tus requisitos de latencia:

    - **Latencia ultra baja** (conversación en tiempo real): fragmentos de 10-20ms (~320-640 bytes @ 16kHz)
    - **Balanceado** (recomendado): fragmentos de 50-100ms (~1600-3200 bytes @ 16kHz)
    - **Menor sobrecarga**: fragmentos de 100-200ms (~3200-6400 bytes @ 16kHz)

    Usa tamaños de fragmento consistentes durante toda la sesión para un rendimiento óptimo. Ejemplo: 100ms @ 16kHz = 16000 muestras/seg × 0.1 seg × 2 bytes/muestra = 3200 bytes.

2. **Reenvío Inmediato**: El `LiveRequestQueue` de ADK reenvía cada fragmento inmediatamente sin fusionar o agrupar. Elige tamaños de fragmento que cumplan tus requisitos de latencia y ancho de banda. No esperes respuestas del modelo antes de enviar los siguientes fragmentos.

3. **Procesamiento Continuo**: El modelo procesa audio continuamente, no por turnos. Con VAD automático habilitado (el predeterminado), simplemente transmite continuamente y deja que la API detecte el habla.

4. **Señales de Actividad**: Usa `send_activity_start()` / `send_activity_end()` solo cuando deshabilites explícitamente VAD para control manual de turnos. VAD está habilitado por defecto, por lo que las señales de actividad no son necesarias para la mayoría de las aplicaciones.

#### Manejando Entrada de Audio en el Cliente

En aplicaciones basadas en navegador, capturar audio del micrófono y enviarlo al servidor requiere usar la API de Web Audio con procesadores AudioWorklet. El bidi-demo demuestra cómo capturar entrada del micrófono, convertirla al formato PCM de 16 bits requerido a 16kHz y transmitirla continuamente al servidor WebSocket.

**Arquitectura:**

1. **Captura de audio**: Usa Web Audio API para acceder al micrófono con tasa de muestreo de 16kHz
2. **Procesamiento de audio**: El procesador AudioWorklet captura fotogramas de audio en tiempo real
3. **Conversión de formato**: Convierte muestras Float32Array a PCM de 16 bits
4. **Transmisión WebSocket**: Envía fragmentos PCM al servidor vía WebSocket

```javascript title='Demo implementation: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/static/js/audio-recorder.js#L7-L58" target="_blank">audio-recorder.js:7-58</a>'
// Iniciar worklet de grabadora de audio
export async function startAudioRecorderWorklet(audioRecorderHandler) {
    // Crear un AudioContext con tasa de muestreo de 16kHz
    // Esto coincide con el formato de entrada requerido por Live API (PCM de 16 bits @ 16kHz)
    const audioRecorderContext = new AudioContext({ sampleRate: 16000 });

    // Cargar el módulo AudioWorklet que procesará audio en tiempo real
    // AudioWorklet se ejecuta en un hilo separado para procesamiento de audio de baja latencia sin interrupciones
    const workletURL = new URL("./pcm-recorder-processor.js", import.meta.url);
    await audioRecorderContext.audioWorklet.addModule(workletURL);

    // Solicitar acceso al micrófono del usuario
    // channelCount: 1 solicita audio mono (canal único) como requiere Live API
    micStream = await navigator.mediaDevices.getUserMedia({
        audio: { channelCount: 1 }
    });
    const source = audioRecorderContext.createMediaStreamSource(micStream);

    // Crear un AudioWorkletNode que usa nuestro procesador de grabadora PCM personalizado
    // Este nodo capturará fotogramas de audio y los enviará a nuestro manejador
    const audioRecorderNode = new AudioWorkletNode(
        audioRecorderContext,
        "pcm-recorder-processor"
    );

    // Conectar la fuente del micrófono al procesador worklet
    // El procesador recibirá fotogramas de audio y los publicará vía port.postMessage
    source.connect(audioRecorderNode);
    audioRecorderNode.port.onmessage = (event) => {
        // Convertir Float32Array al formato PCM de 16 bits requerido por Live API
        const pcmData = convertFloat32ToPCM(event.data);

        // Enviar los datos PCM al manejador (que reenviará a WebSocket)
        audioRecorderHandler(pcmData);
    };
    return [audioRecorderNode, audioRecorderContext, micStream];
}

// Convertir muestras Float32 a PCM de 16 bits
function convertFloat32ToPCM(inputData) {
    // Crear un Int16Array de la misma longitud
    const pcm16 = new Int16Array(inputData.length);
    for (let i = 0; i < inputData.length; i++) {
        // Web Audio API proporciona muestras Float32 en el rango [-1.0, 1.0]
        // Multiplicar por 0x7fff (32767) para convertir al rango de entero con signo de 16 bits [-32768, 32767]
        pcm16[i] = inputData[i] * 0x7fff;
    }
    // Devolver el ArrayBuffer subyacente (datos binarios) para transmisión eficiente
    return pcm16.buffer;
}
```

```javascript title='Demo implementation: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/static/js/pcm-recorder-processor.js#L1-L18" target="_blank">pcm-recorder-processor.js:1-18</a>'
// pcm-recorder-processor.js - Procesador AudioWorklet para capturar audio
class PCMProcessor extends AudioWorkletProcessor {
    constructor() {
        super();
    }

    process(inputs, outputs, parameters) {
        if (inputs.length > 0 && inputs[0].length > 0) {
            // Usar el primer canal (mono)
            const inputChannel = inputs[0][0];
            // Copiar el búfer para evitar problemas con memoria reciclada
            const inputCopy = new Float32Array(inputChannel);
            this.port.postMessage(inputCopy);
        }
        return true;
    }
}

registerProcessor("pcm-recorder-processor", PCMProcessor);
```

```javascript title='Demo implementation: <a href="https://github.com/google/adk-samples/blob/2f7b82f182659e0990bfb86f6ef400dd82633c07/python/agents/bidi-demo/app/static/js/app.js#L979-L988" target="_blank">app.js:977-986</a>'
// Manejador de grabadora de audio - llamado para cada fragmento de audio
function audioRecorderHandler(pcmData) {
    if (websocket && websocket.readyState === WebSocket.OPEN && is_audio) {
        // Enviar audio como fotograma WebSocket binario (más eficiente que JSON base64)
        websocket.send(pcmData);
        console.log("[CLIENT TO AGENT] Sent audio chunk: %s bytes", pcmData.byteLength);
    }
}
```

**Detalles Clave de Implementación:**

1. **Tasa de Muestreo de 16kHz**: El AudioContext debe crearse con `sampleRate: 16000` para coincidir con los requisitos de Live API. Los navegadores modernos soportan esta tasa.

2. **Audio Mono**: Solicita audio de un solo canal (`channelCount: 1`) ya que Live API espera entrada mono. Esto reduce el ancho de banda y la sobrecarga de procesamiento.

3. **Procesamiento AudioWorklet**: AudioWorklet se ejecuta en un hilo separado del hilo principal de JavaScript, asegurando procesamiento de audio de baja latencia sin interrupciones sin bloquear la interfaz de usuario.

4. **Conversión Float32 a PCM16**: Web Audio API proporciona audio como valores Float32Array en el rango [-1.0, 1.0]. Multiplica por 32767 (0x7fff) para convertir a PCM de entero con signo de 16 bits.

5. **Fotogramas WebSocket Binarios**: Envía datos PCM directamente como ArrayBuffer vía fotogramas WebSocket binarios en lugar de codificar en base64 en JSON. Esto reduce el ancho de banda en ~33% y elimina la sobrecarga de codificación/decodificación.

6. **Transmisión Continua**: El método `process()` de AudioWorklet se llama automáticamente a intervalos regulares (típicamente 128 muestras a la vez para 16kHz). Esto proporciona tamaños de fragmento consistentes para transmisión.

Esta arquitectura asegura captura de audio de baja latencia y transmisión eficiente al servidor, que luego lo reenvía a la Live API de ADK vía `LiveRequestQueue.send_realtime()`.

### Recibiendo Salida de Audio

Cuando `response_modalities=["AUDIO"]` está configurado, el modelo devuelve datos de audio en el flujo de eventos como partes `inline_data`.

**Requisitos de Formato de Audio:**

El modelo genera audio en el siguiente formato:

- **Formato**: PCM de 16 bits (entero con signo)
- **Tasa de Muestreo**: 24,000 Hz (24kHz) para modelos de audio nativos
- **Canales**: Mono (canal único)
- **Tipo MIME**: `audio/pcm;rate=24000`

Los datos de audio llegan como bytes PCM sin procesar, listos para reproducción o procesamiento adicional. No se requiere conversión adicional a menos que necesites una tasa de muestreo o formato diferente.

**Recibiendo Salida de Audio:**

```python
from google.adk.agents.run_config import RunConfig, StreamingMode

# Configurar para salida de audio
run_config = RunConfig(
    response_modalities=["AUDIO"],  # Requerido para respuestas de audio
    streaming_mode=StreamingMode.BIDI
)

# Procesar salida de audio del modelo
async for event in runner.run_live(
    user_id="user_123",
    session_id="session_456",
    live_request_queue=live_request_queue,
    run_config=run_config
):
    # Los eventos pueden contener múltiples partes (texto, audio, etc.)
    if event.content and event.content.parts:
        for part in event.content.parts:
            # Los datos de audio llegan como inline_data con tipo MIME audio/pcm
            if part.inline_data and part.inline_data.mime_type.startswith("audio/pcm"):
                # Los datos ya están decodificados a bytes sin procesar (24kHz, PCM de 16 bits, mono)
                audio_bytes = part.inline_data.data

                # Tu lógica para transmitir audio al cliente
                await stream_audio_to_client(audio_bytes)

                # O guardar en archivo
                # with open("output.pcm", "ab") as f:
                #     f.write(audio_bytes)
```

!!! note "Decodificación Base64 Automática"

    El protocolo de cable de Live API transmite datos de audio como cadenas codificadas en base64. El sistema de tipos google.genai usa la característica de serialización base64 de Pydantic (`val_json_bytes='base64'`) para decodificar automáticamente cadenas base64 en bytes al deserializar respuestas de la API. Cuando accedes a `part.inline_data.data`, recibes bytes listos para usar—no se necesita decodificación base64 manual.

#### Manejando Eventos de Audio en el Cliente

El bidi-demo usa un enfoque arquitectónico diferente: en lugar de procesar audio en el servidor, reenvía todos los eventos (incluyendo datos de audio) al cliente WebSocket y maneja la reproducción de audio en el navegador. Este patrón separa las preocupaciones—el servidor se enfoca en la transmisión de eventos de ADK mientras el cliente maneja la reproducción de medios usando Web Audio API.

```python title='Demo implementation: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py#L225-L233" target="_blank">main.py:225-233</a>'
# El bidi-demo reenvía todos los eventos (incluyendo audio) al cliente WebSocket
async for event in runner.run_live(
    user_id=user_id,
    session_id=session_id,
    live_request_queue=live_request_queue,
    run_config=run_config
):
    event_json = event.model_dump_json(exclude_none=True, by_alias=True)
    await websocket.send_text(event_json)
```

**Implementación Demo (Cliente - JavaScript):**

La implementación del lado del cliente involucra tres componentes: manejo de mensajes WebSocket, configuración del reproductor de audio con AudioWorklet, y el procesador AudioWorklet en sí.

```javascript title='Demo implementation: <a href="https://github.com/google/adk-samples/blob/2f7b82f182659e0990bfb86f6ef400dd82633c07/python/agents/bidi-demo/app/static/js/app.js#L640-L690" target="_blank">app.js:638-688</a>'
// 1. Manejador de Mensajes WebSocket
// Manejar eventos de contenido (texto o audio)
if (adkEvent.content && adkEvent.content.parts) {
    const parts = adkEvent.content.parts;

    for (const part of parts) {
        // Manejar datos inline (audio)
        if (part.inlineData) {
            const mimeType = part.inlineData.mimeType;
            const data = part.inlineData.data;

            // Verificar si estos son datos de audio PCM y el reproductor de audio está listo
            if (mimeType && mimeType.startsWith("audio/pcm") && audioPlayerNode) {
                // Decodificar base64 a ArrayBuffer y enviar a AudioWorklet para reproducción
                audioPlayerNode.port.postMessage(base64ToArray(data));
            }
        }
    }
}

// Decodificar datos de audio base64 a ArrayBuffer
function base64ToArray(base64) {
    // Convertir base64url a base64 estándar (cumplimiento RFC 4648)
    // base64url usa '-' y '_' en lugar de '+' y '/', que son seguros para URL
    let standardBase64 = base64.replace(/-/g, '+').replace(/_/g, '/');

    // Agregar caracteres de relleno '=' si es necesario
    // Las cadenas Base64 deben ser múltiplos de 4 caracteres
    while (standardBase64.length % 4) {
        standardBase64 += '=';
    }

    // Decodificar cadena base64 a cadena binaria usando API del navegador
    const binaryString = window.atob(standardBase64);
    const len = binaryString.length;
    const bytes = new Uint8Array(len);
    // Convertir cada código de carácter (0-255) a un byte
    for (let i = 0; i < len; i++) {
        bytes[i] = binaryString.charCodeAt(i);
    }
    // Devolver el ArrayBuffer subyacente (datos binarios)
    return bytes.buffer;
}
```

```javascript title='Demo implementation: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/static/js/audio-player.js#L5-L24" target="_blank">audio-player.js:5-24</a>'
// 2. Configuración del Reproductor de Audio
// Iniciar worklet de reproductor de audio
export async function startAudioPlayerWorklet() {
    // Crear un AudioContext con tasa de muestreo de 24kHz
    // Esto coincide con el formato de audio de salida de Live API (PCM de 16 bits @ 24kHz)
    // Nota: Diferente de la tasa de entrada (16kHz) - Live API genera audio de mayor calidad
    const audioContext = new AudioContext({
        sampleRate: 24000
    });

    // Cargar el módulo AudioWorklet que manejará la reproducción de audio
    // AudioWorklet se ejecuta en el hilo de renderizado de audio para reproducción suave de baja latencia
    const workletURL = new URL('./pcm-player-processor.js', import.meta.url);
    await audioContext.audioWorklet.addModule(workletURL);

    // Crear un AudioWorkletNode usando nuestro procesador de reproductor PCM personalizado
    // Este nodo recibirá datos de audio vía postMessage y los reproducirá a través de los altavoces
    const audioPlayerNode = new AudioWorkletNode(audioContext, 'pcm-player-processor');

    // Conectar el nodo del reproductor al destino de audio (altavoces/auriculares)
    // Esto establece el gráfico de audio: AudioWorklet → AudioContext.destination
    audioPlayerNode.connect(audioContext.destination);

    return [audioPlayerNode, audioContext];
}
```

```javascript title='Demo implementation: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/static/js/pcm-player-processor.js#L5-L76" target="_blank">pcm-player-processor.js:5-76</a>'
// 3. Procesador AudioWorklet (Búfer Circular)
// Procesador AudioWorklet que almacena en búfer y reproduce audio PCM
class PCMPlayerProcessor extends AudioWorkletProcessor {
    constructor() {
        super();

        // Inicializar búfer circular (24kHz x 180 segundos = ~4.3 millones de muestras)
        // El búfer circular absorbe la fluctuación de red y asegura reproducción suave
        this.bufferSize = 24000 * 180;
        this.buffer = new Float32Array(this.bufferSize);
        this.writeIndex = 0;  // Dónde escribimos nuevos datos de audio
        this.readIndex = 0;   // Dónde leemos para reproducción

        // Manejar mensajes entrantes del hilo principal
        this.port.onmessage = (event) => {
            // Restablecer búfer en interrupción (ej., usuario interrumpe respuesta del modelo)
            if (event.data.command === 'endOfAudio') {
                this.readIndex = this.writeIndex; // Limpiar el búfer saltando lectura a posición de escritura
                return;
            }

            // Decodificar array Int16 del ArrayBuffer entrante
            // Live API envía datos de audio PCM de 16 bits
            const int16Samples = new Int16Array(event.data);

            // Agregar datos de audio al búfer circular para reproducción
            this._enqueue(int16Samples);
        };
    }

    // Empujar datos Int16 entrantes al búfer circular
    _enqueue(int16Samples) {
        for (let i = 0; i < int16Samples.length; i++) {
            // Convertir entero de 16 bits a flotante en [-1.0, 1.0] requerido por Web Audio API
            // Dividir por 32768 (valor positivo máximo para entero con signo de 16 bits)
            const floatVal = int16Samples[i] / 32768;

            // Almacenar en búfer circular en la posición de escritura actual
            this.buffer[this.writeIndex] = floatVal;
            // Mover índice de escritura hacia adelante, envolviendo al final del búfer (búfer circular)
            this.writeIndex = (this.writeIndex + 1) % this.bufferSize;

            // Manejo de desbordamiento: si la escritura alcanza la lectura, mover lectura hacia adelante
            // Esto sobrescribe las muestras más antiguas no reproducidas (raro, solo bajo retraso extremo de red)
            if (this.writeIndex === this.readIndex) {
                this.readIndex = (this.readIndex + 1) % this.bufferSize;
            }
        }
    }

    // Llamado automáticamente por el sistema Web Audio ~128 muestras a la vez
    // Esto se ejecuta en el hilo de renderizado de audio para temporización precisa
    process(inputs, outputs, parameters) {
        const output = outputs[0];
        const framesPerBlock = output[0].length;

        for (let frame = 0; frame < framesPerBlock; frame++) {
            // Escribir muestras al búfer de salida (mono a estéreo)
            output[0][frame] = this.buffer[this.readIndex]; // canal izquierdo
            if (output.length > 1) {
                output[1][frame] = this.buffer[this.readIndex]; // canal derecho (duplicar para estéreo)
            }

            // Mover índice de lectura hacia adelante a menos que el búfer esté vacío (protección contra subflujo)
            if (this.readIndex != this.writeIndex) {
                this.readIndex = (this.readIndex + 1) % this.bufferSize;
            }
            // Si readIndex == writeIndex, nos quedamos sin datos - generar silencio (0.0)
        }

        return true; // Mantener procesador vivo (devolver false para terminar)
    }
}

registerProcessor('pcm-player-processor', PCMPlayerProcessor);
```

**Patrones Clave de Implementación:**

1. **Decodificación Base64**: El servidor envía datos de audio como cadenas codificadas en base64 en JSON. El cliente debe decodificar a ArrayBuffer antes de pasar a AudioWorklet. Manejar tanto codificación base64 estándar como base64url.

2. **Tasa de Muestreo de 24kHz**: El AudioContext debe crearse con `sampleRate: 24000` para coincidir con el formato de salida de Live API (diferente de la entrada de 16kHz).

3. **Arquitectura de Búfer Circular**: Usa un búfer circular para manejar latencia de red variable y asegurar reproducción suave. El búfer almacena muestras Float32 y maneja desbordamiento sobrescribiendo datos más antiguos.

4. **Conversión PCM16 a Float32**: Live API envía enteros con signo de 16 bits. Divide por 32768 para convertir a Float32 en el rango [-1.0, 1.0] requerido por Web Audio API.

5. **Mono a Estéreo**: El procesador duplica audio mono a los canales izquierdo y derecho para salida estéreo, asegurando compatibilidad con todos los dispositivos de audio.

6. **Manejo de Interrupciones**: En eventos de interrupción, envía comando `endOfAudio` para limpiar el búfer estableciendo `readIndex = writeIndex`, previniendo reproducción de audio obsoleto.

Esta arquitectura asegura reproducción de audio suave de baja latencia mientras maneja fluctuaciones de red e interrupciones con gracia.

## Cómo Usar Imagen y Video

Tanto las imágenes como el video en Bidi-streaming de ADK se procesan como fotogramas JPEG. En lugar de la típica transmisión de video usando HLS, mp4 o H.264, ADK usa un enfoque directo de procesamiento de imagen fotograma por fotograma donde tanto imágenes estáticas como fotogramas de video se envían como imágenes JPEG individuales.

**Especificaciones de Imagen/Video:**

- **Formato**: JPEG (`image/jpeg`)
- **Tasa de fotogramas**: 1 fotograma por segundo (1 FPS) máximo recomendado
- **Resolución**: 768x768 píxeles (recomendado)

```python title='Demo implementation: <a href="https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py#L202-L217" target="_blank">main.py:202-217</a>'
# Decodificar datos de imagen base64
image_data = base64.b64decode(json_message["data"])
mime_type = json_message.get("mimeType", "image/jpeg")

# Enviar imagen como blob
image_blob = types.Blob(
    mime_type=mime_type,
    data=image_data
)
live_request_queue.send_realtime(image_blob)
```

**No Adecuado Para**:

- **Reconocimiento de acción de video en tiempo real** - 1 FPS es demasiado lento para capturar movimientos o acciones rápidas
- **Análisis de deportes en vivo o seguimiento de movimiento** - Resolución temporal insuficiente para sujetos en movimiento rápido

**Caso de Uso de Ejemplo para Procesamiento de Imágenes**:

En el [demo de Shopper's Concierge](https://youtu.be/LwHPYyw7u6U?si=lG9gl9aSIuu-F4ME&t=40), la aplicación usa `send_realtime()` para enviar la imagen cargada por el usuario. El agente reconoce el contexto de la imagen y busca artículos relevantes en el sitio de comercio electrónico.

<div class="video-grid">
  <div class="video-item">
    <div class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/LwHPYyw7u6U?si=lG9gl9aSIuu-F4ME&amp;start=40" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
    </div>
  </div>
</div>

### Manejando Entrada de Imagen en el Cliente

En aplicaciones basadas en navegador, capturar imágenes de la cámara web del usuario y enviarlas al servidor requiere usar la API MediaDevices para acceder a la cámara, capturar fotogramas a un lienzo y convertir a formato JPEG. El bidi-demo demuestra cómo abrir un modal de vista previa de cámara, capturar un solo fotograma y enviarlo como JPEG codificado en base64 al servidor WebSocket.

**Arquitectura:**

1. **Acceso a cámara**: Usa `navigator.mediaDevices.getUserMedia()` para acceder a la cámara web
2. **Vista previa de video**: Muestra alimentación de cámara en vivo en un elemento `<video>`
3. **Captura de fotograma**: Dibuja fotograma de video en `<canvas>` y convierte a JPEG
4. **Codificación Base64**: Convierte lienzo a URL de datos base64 para transmisión
5. **Transmisión WebSocket**: Envía como mensaje JSON al servidor

```javascript title='Demo implementation: <a href="https://github.com/google/adk-samples/blob/2f7b82f182659e0990bfb86f6ef400dd82633c07/python/agents/bidi-demo/app/static/js/app.js#L803-L845" target="_blank">app.js:801-843</a>'
// 1. Abriendo Vista Previa de Cámara
// Abrir modal de cámara e iniciar vista previa
async function openCameraPreview() {
    try {
        // Solicitar acceso a la cámara web del usuario con resolución 768x768
        cameraStream = await navigator.mediaDevices.getUserMedia({
            video: {
                width: { ideal: 768 },
                height: { ideal: 768 },
                facingMode: 'user'
            }
        });

        // Establecer el flujo al elemento de video
        cameraPreview.srcObject = cameraStream;

        // Mostrar el modal
        cameraModal.classList.add('show');

    } catch (error) {
        console.error('Error accessing camera:', error);
        addSystemMessage(`Failed to access camera: ${error.message}`);
    }
}

// Cerrar modal de cámara y detener vista previa
function closeCameraPreview() {
    // Detener el flujo de cámara
    if (cameraStream) {
        cameraStream.getTracks().forEach(track => track.stop());
        cameraStream = null;
    }

    // Limpiar la fuente de video
    cameraPreview.srcObject = null;

    // Ocultar el modal
    cameraModal.classList.remove('show');
}
```

```javascript title='Demo implementation: <a href="https://github.com/google/adk-samples/blob/2f7b82f182659e0990bfb86f6ef400dd82633c07/python/agents/bidi-demo/app/static/js/app.js#L848-L916" target="_blank">app.js:846-914</a>'
// 2. Capturando y Enviando Imagen
// Capturar imagen de la vista previa en vivo
function captureImageFromPreview() {
    if (!cameraStream) {
        addSystemMessage('No camera stream available');
        return;
    }

    try {
        // Crear lienzo para capturar el fotograma
        const canvas = document.createElement('canvas');
        canvas.width = cameraPreview.videoWidth;
        canvas.height = cameraPreview.videoHeight;
        const context = canvas.getContext('2d');

        // Dibujar fotograma de video actual al lienzo
        context.drawImage(cameraPreview, 0, 0, canvas.width, canvas.height);

        // Convertir lienzo a URL de datos para mostrar
        const imageDataUrl = canvas.toDataURL('image/jpeg', 0.85);

        // Mostrar la imagen capturada en el chat
        const imageBubble = createImageBubble(imageDataUrl, true);
        messagesDiv.appendChild(imageBubble);

        // Convertir lienzo a blob para enviar al servidor
        canvas.toBlob((blob) => {
            // Convertir blob a base64 para enviar al servidor
            const reader = new FileReader();
            reader.onloadend = () => {
                // Eliminar prefijo data:image/jpeg;base64,
                const base64data = reader.result.split(',')[1];
                sendImage(base64data);
            };
            reader.readAsDataURL(blob);
        }, 'image/jpeg', 0.85);

        // Cerrar el modal de cámara
        closeCameraPreview();

    } catch (error) {
        console.error('Error capturing image:', error);
        addSystemMessage(`Failed to capture image: ${error.message}`);
    }
}

// Enviar imagen al servidor
function sendImage(base64Image) {
    if (websocket && websocket.readyState === WebSocket.OPEN) {
        const jsonMessage = JSON.stringify({
            type: "image",
            data: base64Image,
            mimeType: "image/jpeg"
        });
        websocket.send(jsonMessage);
        console.log("[CLIENT TO AGENT] Sent image");
    }
}
```

**Detalles Clave de Implementación:**

1. **Resolución 768x768**: Solicita resolución ideal de 768x768 para coincidir con la especificación recomendada. El navegador proporcionará la resolución disponible más cercana.

2. **Cámara Frontal**: La restricción `facingMode: 'user'` selecciona la cámara frontal en dispositivos móviles, apropiada para capturas de autorretrato.

3. **Captura de Fotograma en Lienzo**: Usa `canvas.getContext('2d').drawImage()` para capturar un solo fotograma del flujo de video en vivo. Esto crea una instantánea estática del fotograma de video actual.

4. **Compresión JPEG**: El segundo parámetro de `toDataURL()` y `toBlob()` es la calidad (0.0 a 1.0). Usar 0.85 proporciona buena calidad mientras mantiene el tamaño de archivo manejable.

5. **Salida Dual**: El código crea tanto una URL de datos para visualización inmediata en la interfaz de usuario como un blob para codificación base64 eficiente, demostrando un patrón para retroalimentación de usuario receptiva.

6. **Limpieza de Recursos**: Siempre llama `getTracks().forEach(track => track.stop())` al cerrar la cámara para liberar el recurso de hardware y apagar la luz indicadora de cámara.

7. **Codificación Base64**: El FileReader convierte el blob a una URL de datos (`data:image/jpeg;base64,<data>`). Divide en coma y toma la segunda parte para obtener solo los datos base64 sin el prefijo.

Esta implementación proporciona una interfaz de cámara amigable con vista previa, captura de fotograma único y transmisión eficiente al servidor para procesamiento por Live API.

### Soporte de Herramientas de Transmisión de Video Personalizadas

ADK proporciona soporte especial de herramientas para procesar fotogramas de video durante sesiones de transmisión. A diferencia de las herramientas regulares que se ejecutan sincrónicamente, las herramientas de transmisión pueden generar fotogramas de video asincrónicamente mientras el modelo continúa generando respuestas.

**Ciclo de Vida de Herramienta de Transmisión:**

1. **Inicio**: ADK invoca tu función generadora asíncrona cuando el modelo la llama
2. **Transmisión**: Tu función genera resultados continuamente vía `AsyncGenerator`
3. **Detención**: ADK cancela la tarea del generador cuando:
   - El modelo llama a una función `stop_streaming()` que proporcionas
   - La sesión termina
   - Ocurre un error

**Importante**: Debes proporcionar una función `stop_streaming(function_name: str)` como herramienta para permitir que el modelo detenga explícitamente operaciones de transmisión.

Para implementar herramientas de transmisión de video personalizadas que procesan y generan fotogramas de video al modelo, consulta la [documentación de Herramientas de Transmisión](https://google.github.io/adk-docs/streaming/streaming-tools/).

## Comprendiendo Arquitecturas de Modelos de Audio

Al construir aplicaciones de voz con Live API, una de las decisiones más importantes es seleccionar la arquitectura de modelo de audio correcta. Live API soporta dos tipos fundamentalmente diferentes de modelos para procesamiento de audio: **Audio Nativo** y **Semi-Cascada**. Estas arquitecturas de modelo difieren en cómo procesan entrada de audio y generan salida de audio, lo que impacta directamente la naturalidad de respuesta, confiabilidad de ejecución de herramientas, características de latencia e idoneidad de caso de uso general.

Comprender estas arquitecturas te ayuda a tomar decisiones informadas de selección de modelo basadas en los requisitos de tu aplicación—ya sea que priorices IA conversacional natural, confiabilidad de producción o disponibilidad de características específicas.

### Modelos de Audio Nativo

Una arquitectura de modelo de audio de extremo a extremo completamente integrada donde el modelo procesa entrada de audio y genera salida de audio directamente, sin conversión de texto intermedia. Este enfoque habilita habla más humana con prosodia natural.

| Arquitectura de Modelo de Audio | Plataforma | Modelo | Notas |
|-------------------|----------|-------|-------|
| Audio Nativo | Gemini Live API | [gemini-2.5-flash-native-audio-preview-12-2025](https://ai.google.dev/gemini-api/docs/models#gemini-2.5-flash-live) |Disponible públicamente|
| Audio Nativo | Vertex AI Live API | [gemini-live-2.5-flash-native-audio](https://cloud.google.com/vertex-ai/generative-ai/docs/models/gemini/2-5-flash-live-api) | Vista previa pública |

**Características Clave:**

- **Procesamiento de audio de extremo a extremo**: Procesa entrada de audio y genera salida de audio directamente sin convertir a texto intermediamente
- **Prosodia natural**: Produce patrones de habla más humanos, entonación y expresividad emocional
- **Biblioteca de voz extendida**: Soporta todas las voces semi-cascada más voces adicionales del servicio de Text-to-Speech (TTS)
- **Detección automática de idioma**: Determina idioma del contexto de conversación sin configuración explícita
- **Características conversacionales avanzadas**:
  - **[Diálogo afectivo](#proactividad-y-dialogo-afectivo)**: Adapta estilo de respuesta a expresión y tono de entrada, detectando señales emocionales
  - **[Audio proactivo](#proactividad-y-dialogo-afectivo)**: Puede decidir proactivamente cuándo responder, ofrecer sugerencias o ignorar entrada irrelevante
  - **Pensamiento dinámico**: Soporta resúmenes de pensamiento y presupuestos de pensamiento dinámicos
- **Modalidad de respuesta solo AUDIO**: No soporta modalidad de respuesta TEXT con `RunConfig`, resultando en tiempos de respuesta inicial más lentos

### Modelos Semi-Cascada

Una arquitectura híbrida que combina procesamiento de entrada de audio nativo con generación de salida de text-to-speech (TTS). También referida como modelos "En cascada" en alguna documentación.

La entrada de audio se procesa nativamente, pero las respuestas se generan primero como texto y luego se convierten a habla. Esta separación proporciona mejor confiabilidad y ejecución de herramientas más robusta en ambientes de producción.

| Arquitectura de Modelo de Audio | Plataforma | Modelo | Notas |
|-------------------|----------|-------|-------|
| Semi-Cascada | Gemini Live API | [gemini-2.0-flash-live-001](https://ai.google.dev/gemini-api/docs/models#gemini-2.0-flash-live) | Obsoleto el 09 de diciembre de 2025 |
| Semi-Cascada | Vertex AI Live API | [gemini-live-2.5-flash](https://cloud.google.com/vertex-ai/generative-ai/docs/models/gemini/2-5-flash#2.5-flash) | GA privado, no disponible públicamente |

**Características Clave:**

- **Arquitectura híbrida**: Combina procesamiento de entrada de audio nativo con generación de salida de audio basada en TTS
- **Soporte de modalidad de respuesta TEXT**: Soporta modalidad de respuesta TEXT con `RunConfig` además de AUDIO, habilitando respuestas mucho más rápidas para casos de uso solo texto
- **Control de idioma explícito**: Soporta configuración manual de código de idioma vía `speech_config.language_code`
- **Calidad TTS establecida**: Aprovecha tecnología text-to-speech probada para salida de audio consistente
- **Voces soportadas**: Puck, Charon, Kore, Fenrir, Aoede, Leda, Orus, Zephyr (8 voces preconfiguradas)

### Cómo Manejar Nombres de Modelos

Al construir aplicaciones ADK, necesitarás especificar qué modelo usar. El enfoque recomendado es usar variables de entorno para configuración de modelo, lo que proporciona flexibilidad a medida que la disponibilidad y nomenclatura de modelos cambia con el tiempo.

**Patrón Recomendado:**

```python
import os
from google.adk.agents import Agent

# Usar variable de entorno con respaldo a un valor predeterminado sensato
agent = Agent(
    name="my_agent",
    model=os.getenv("DEMO_AGENT_MODEL", "gemini-2.5-flash-native-audio-preview-12-2025"),
    tools=[...],
    instruction="..."
)
```

**Por qué usar variables de entorno:**

- **Disponibilidad de modelo cambia**: Los modelos se lanzan, actualizan y quedan obsoletos regularmente (ej., `gemini-2.0-flash-live-001` quedó obsoleto el 09 de diciembre de 2025)
- **Nombres específicos de plataforma**: Gemini Live API y Vertex AI Live API usan diferentes convenciones de nomenclatura de modelo para la misma funcionalidad
- **Cambio fácil**: Cambia modelos sin modificar código actualizando el archivo `.env`
- **Configuración específica de entorno**: Usa diferentes modelos para desarrollo, staging y producción

**Configuración en archivo `.env`:**

```bash
# Para Gemini Live API (disponible públicamente)
DEMO_AGENT_MODEL=gemini-2.5-flash-native-audio-preview-12-2025

# Para Vertex AI Live API (si usas Vertex AI)
# DEMO_AGENT_MODEL=gemini-live-2.5-flash-native-audio
```

!!! note "Orden de Carga de Variable de Entorno"

    Al usar archivos `.env` con `python-dotenv`, debes llamar `load_dotenv()` **antes** de importar cualquier módulo que lea variables de entorno. De lo contrario, `os.getenv()` devolverá `None` y recurrirá al valor predeterminado, ignorando tu configuración `.env`.

    **Orden correcto en `main.py`:**

    ```python
    from dotenv import load_dotenv
    from pathlib import Path

    # Cargar archivo .env ANTES de importar agente
    load_dotenv(Path(__file__).parent / ".env")

    # Ahora es seguro importar módulos que usan variables de entorno
    from google_search_agent.agent import agent
    ```

    **Orden incorrecto (no funcionará):**

    ```python
    from dotenv import load_dotenv
    from google_search_agent.agent import agent  # El agente lee var de entorno aquí

    # ¡Demasiado tarde! El agente ya se inicializó con modelo predeterminado
    load_dotenv(Path(__file__).parent / ".env")
    ```

    Este es un comportamiento de importación de Python: cuando importas un módulo, su código de nivel superior se ejecuta inmediatamente. Si tu módulo de agente llama `os.getenv("DEMO_AGENT_MODEL")` en el momento de importación, el archivo `.env` ya debe estar cargado.

**Seleccionando el modelo correcto:**

1. **Elige plataforma**: Decide entre Gemini Live API (pública) o Vertex AI Live API (empresarial)
2. **Elige arquitectura**:
   - Audio Nativo para IA conversacional natural con características avanzadas
   - Semi-Cascada para confiabilidad de producción con ejecución de herramientas
3. **Verifica disponibilidad actual**: Consulta las tablas de modelos arriba y documentación oficial
4. **Configura variable de entorno**: Establece `DEMO_AGENT_MODEL` en tu archivo `.env` (ver [`agent.py:11-16`](https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/google_search_agent/agent.py#L11-L16) y [`main.py:99-152`](https://github.com/google/adk-samples/blob/31847c0723fbf16ddf6eed411eb070d1c76afd1a/python/agents/bidi-demo/app/main.py#L99-L152))

### Compatibilidad y Disponibilidad de Modelos de Live API

Para la información más reciente sobre compatibilidad y disponibilidad de modelos de Live API:

- **Modelos de Gemini Live API**: Ver la [documentación de modelos Gemini](https://ai.google.dev/gemini-api/docs/models/gemini)
- **Modelos de Vertex AI Live API**: Ver la [documentación de modelos Vertex AI](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/models)

Siempre verifica la disponibilidad de modelo y soporte de características en la documentación oficial antes de desplegar a producción.

## Transcripción de Audio

Live API proporciona capacidades de transcripción de audio integradas que automáticamente convierten habla a texto tanto para entrada de usuario como salida de modelo. Esto elimina la necesidad de servicios de transcripción externos y habilita subtítulos en tiempo real, registro de conversaciones y características de accesibilidad. ADK expone estas capacidades a través de `RunConfig`, permitiéndote habilitar transcripción para cualquiera o ambas direcciones de audio.

!!! note "Fuente"

    [Gemini Live API - Transcripciones de audio](https://ai.google.dev/gemini-api/docs/live-guide#audio-transcriptions)

**Configuración:**

```python
from google.genai import types
from google.adk.agents.run_config import RunConfig

# Comportamiento predeterminado: Transcripción de audio está HABILITADA por defecto
# Tanto transcripción de entrada como salida están automáticamente configuradas
run_config = RunConfig(
    response_modalities=["AUDIO"]
    # input_audio_transcription tiene como valor predeterminado AudioTranscriptionConfig()
    # output_audio_transcription tiene como valor predeterminado AudioTranscriptionConfig()
)

# Para deshabilitar transcripción explícitamente:
run_config = RunConfig(
    response_modalities=["AUDIO"],
    input_audio_transcription=None,   # Deshabilitar explícitamente transcripción de entrada de usuario
    output_audio_transcription=None   # Deshabilitar explícitamente transcripción de salida de modelo
)

# Habilitar solo transcripción de entrada (deshabilitar salida):
run_config = RunConfig(
    response_modalities=["AUDIO"],
    input_audio_transcription=types.AudioTranscriptionConfig(),  # Habilitar explícitamente (redundante con predeterminado)
    output_audio_transcription=None  # Deshabilitar explícitamente
)

# Habilitar solo transcripción de salida (deshabilitar entrada):
run_config = RunConfig(
    response_modalities=["AUDIO"],
    input_audio_transcription=None,  # Deshabilitar explícitamente
    output_audio_transcription=types.AudioTranscriptionConfig()  # Habilitar explícitamente (redundante con predeterminado)
)
```

**Estructura de Evento**:

Las transcripciones se entregan como objetos `types.Transcription` en el objeto `Event`:

```python
from dataclasses import dataclass
from typing import Optional
from google.genai import types

@dataclass
class Event:
    content: Optional[Content]  # Contenido de audio/texto
    input_transcription: Optional[types.Transcription]  # Habla de usuario → texto
    output_transcription: Optional[types.Transcription]  # Habla de modelo → texto
    # ... otros campos
```

!!! note "Aprende Más"

    Para la estructura Event completa, ver [Parte 3: La Clase Event](part3.md#the-event-class).

Cada objeto `Transcription` tiene dos atributos:
- **`.text`**: El texto transcrito (cadena)
- **`.finished`**: Booleano indicando si la transcripción está completa (True) o parcial (False)

**Cómo se Entregan las Transcripciones**:

Las transcripciones llegan como campos separados en el flujo de eventos, no como partes de contenido. Siempre usa verificación defensiva de nulos al acceder datos de transcripción:

**Procesando Transcripciones:**

```python
from google.adk.runners import Runner

# ... código de configuración del runner ...

async for event in runner.run_live(...):
    # Transcripción del habla del usuario (de audio de entrada)
    if event.input_transcription:  # Primera verificación: el objeto de transcripción existe
        # Acceder al texto de transcripción y estado
        user_text = event.input_transcription.text
        is_finished = event.input_transcription.finished

        # Segunda verificación: el texto no es None o vacío
        # Esto maneja casos donde la transcripción está en progreso o vacía
        if user_text and user_text.strip():
            print(f"User said: {user_text} (finished: {is_finished})")

            # Tu lógica de actualización de subtítulos
            update_caption(user_text, is_user=True, is_final=is_finished)

    # Transcripción del habla del modelo (de audio de salida)
    if event.output_transcription:  # Primera verificación: el objeto de transcripción existe
        model_text = event.output_transcription.text
        is_finished = event.output_transcription.finished

        # Segunda verificación: el texto no es None o vacío
        # Esto maneja casos donde la transcripción está en progreso o vacía
        if model_text and model_text.strip():
            print(f"Model said: {model_text} (finished: {is_finished})")

            # Tu lógica de actualización de subtítulos
            update_caption(model_text, is_user=False, is_final=is_finished)
```

!!! tip "Mejor Práctica para Verificación de Nulos en Transcripción"

    Siempre usa verificación de nulos de dos niveles para transcripciones:

    1. Verifica si el objeto de transcripción existe (`if event.input_transcription`)
    2. Verifica si el texto no está vacío (`if user_text and user_text.strip()`)

    Este patrón previene errores de valores `None` y maneja transcripciones parciales que pueden estar vacías.

### Manejando Transcripción de Audio en el Cliente

En aplicaciones web, los eventos de transcripción necesitan reenviarse del servidor al navegador y renderizarse en la interfaz de usuario. El bidi-demo demuestra un patrón donde el servidor reenvía todos los eventos de ADK (incluyendo eventos de transcripción) al cliente WebSocket, y el cliente maneja mostrar transcripciones como burbujas de habla con indicadores visuales para transcripciones parciales vs. terminadas.

**Arquitectura:**

1. **Lado del servidor**: Reenviar eventos de transcripción a través de WebSocket (ya mostrado en sección anterior)
2. **Lado del cliente**: Procesar eventos `inputTranscription` y `outputTranscription` del WebSocket
3. **Renderizado de interfaz de usuario**: Mostrar transcripciones parciales con indicadores de escritura, finalizar cuando `finished: true`

```javascript title='Demo implementation: <a href="https://github.com/google/adk-samples/blob/2f7b82f182659e0990bfb86f6ef400dd82633c07/python/agents/bidi-demo/app/static/js/app.js#L532-L655" target="_blank">app.js:530-653</a>'
// Manejar transcripción de entrada (palabras habladas del usuario)
if (adkEvent.inputTranscription && adkEvent.inputTranscription.text) {
    const transcriptionText = adkEvent.inputTranscription.text;
    const isFinished = adkEvent.inputTranscription.finished;

    if (transcriptionText) {
        if (currentInputTranscriptionId == null) {
            // Crear nueva burbuja de transcripción
            currentInputTranscriptionId = Math.random().toString(36).substring(7);
            currentInputTranscriptionElement = createMessageBubble(
                transcriptionText,
                true,  // isUser
                !isFinished  // isPartial
            );
            currentInputTranscriptionElement.id = currentInputTranscriptionId;
            currentInputTranscriptionElement.classList.add("transcription");
            messagesDiv.appendChild(currentInputTranscriptionElement);
        } else {
            // Actualizar burbuja de transcripción existente
            if (currentOutputTranscriptionId == null && currentMessageId == null) {
                // Acumular texto de transcripción de entrada (Live API envía piezas incrementales)
                const existingText = currentInputTranscriptionElement
                    .querySelector(".bubble-text").textContent;
                const cleanText = existingText.replace(/\.\.\.$/, '');
                const accumulatedText = cleanText + transcriptionText;
                updateMessageBubble(
                    currentInputTranscriptionElement,
                    accumulatedText,
                    !isFinished
                );
            }
        }

        // Si la transcripción está terminada, restablecer el estado
        if (isFinished) {
            currentInputTranscriptionId = null;
            currentInputTranscriptionElement = null;
        }
    }
}

// Manejar transcripción de salida (palabras habladas del modelo)
if (adkEvent.outputTranscription && adkEvent.outputTranscription.text) {
    const transcriptionText = adkEvent.outputTranscription.text;
    const isFinished = adkEvent.outputTranscription.finished;

    if (transcriptionText) {
        // Finalizar cualquier transcripción de entrada activa cuando el modelo comienza a responder
        if (currentInputTranscriptionId != null && currentOutputTranscriptionId == null) {
            const textElement = currentInputTranscriptionElement
                .querySelector(".bubble-text");
            const typingIndicator = textElement.querySelector(".typing-indicator");
            if (typingIndicator) {
                typingIndicator.remove();
            }
            currentInputTranscriptionId = null;
            currentInputTranscriptionElement = null;
        }

        if (currentOutputTranscriptionId == null) {
            // Crear nueva burbuja de transcripción para modelo
            currentOutputTranscriptionId = Math.random().toString(36).substring(7);
            currentOutputTranscriptionElement = createMessageBubble(
                transcriptionText,
                false,  // isUser
                !isFinished  // isPartial
            );
            currentOutputTranscriptionElement.id = currentOutputTranscriptionId;
            currentOutputTranscriptionElement.classList.add("transcription");
            messagesDiv.appendChild(currentOutputTranscriptionElement);
        } else {
            // Actualizar burbuja de transcripción existente
            const existingText = currentOutputTranscriptionElement
                .querySelector(".bubble-text").textContent;
            const cleanText = existingText.replace(/\.\.\.$/, '');
            updateMessageBubble(
                currentOutputTranscriptionElement,
                cleanText + transcriptionText,
                !isFinished
            );
        }

        // Si la transcripción está terminada, restablecer el estado
        if (isFinished) {
            currentOutputTranscriptionId = null;
            currentOutputTranscriptionElement = null;
        }
    }
}
```

**Patrones Clave de Implementación:**

1. **Acumulación de Texto Incremental**: Live API puede enviar transcripciones en múltiples fragmentos. Acumula texto agregando nuevas piezas al contenido existente:
   ```javascript
   const accumulatedText = cleanText + transcriptionText;
   ```

2. **Estados Parcial vs. Terminado**: Usa la bandera `finished` para determinar si mostrar indicadores de escritura:
   - `finished: false` → Mostrar indicador de escritura (ej., "...")
   - `finished: true` → Eliminar indicador de escritura, finalizar burbuja

3. **Gestión de Estado de Burbujas**: Rastrea burbujas de transcripción actuales separadamente para entrada y salida usando IDs. Crea nuevas burbujas solo al iniciar transcripciones frescas:
   ```javascript
   if (currentInputTranscriptionId == null) {
       // Crear nueva burbuja
   } else {
       // Actualizar burbuja existente
   }
   ```

4. **Coordinación de Turnos**: Cuando el modelo comienza a responder (primera transcripción de salida llega), finaliza cualquier transcripción de entrada activa para prevenir actualizaciones superpuestas.

Este patrón asegura visualización de transcripción en tiempo real suave con manejo adecuado de actualizaciones de transmisión, transiciones de turnos y retroalimentación visual para usuarios.

### Requisitos de Transcripción Multi-Agente

Para escenarios multi-agente (agentes con `sub_agents`), ADK automáticamente habilita transcripción de audio independientemente de tu configuración `RunConfig`. Este comportamiento automático es requerido para funcionalidad de transferencia de agente, donde las transcripciones de texto se usan para pasar contexto de conversación entre agentes.

**Comportamiento de Habilitación Automática:**

Cuando un agente tiene `sub_agents` definidos, el método `run_live()` de ADK automáticamente habilita transcripción de audio tanto de entrada como salida **incluso si las estableces explícitamente a `None`**. Esto asegura que las transferencias de agente funcionen correctamente proporcionando contexto de texto al siguiente agente.

**Por Qué Esto Importa:**

1. **No puede deshabilitarse**: No puedes desactivar la transcripción en escenarios multi-agente
2. **Requerido para funcionalidad**: La transferencia de agente se rompe sin contexto de texto
3. **Transparente para desarrolladores**: Los eventos de transcripción están automáticamente disponibles
4. **Plan