# Configurando el comportamiento de streaming

<div class="language-support-tag">
    <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.5.0</span><span class="lst-preview">Experimental</span>
</div>

Hay algunas configuraciones que puedes establecer para agentes en vivo (streaming).

Se configura mediante [RunConfig](https://github.com/google/adk-python/blob/main/src/google/adk/agents/run_config.py). Debes usar RunConfig con tu [Runner.run_live(...)](https://github.com/google/adk-python/blob/main/src/google/adk/runners.py).

Por ejemplo, si quieres configurar la configuración de voz, puedes aprovechar speech_config.

```python
voice_config = genai_types.VoiceConfig(
    prebuilt_voice_config=genai_types.PrebuiltVoiceConfigDict(
        voice_name='Aoede'
    )
)
speech_config = genai_types.SpeechConfig(voice_config=voice_config)
run_config = RunConfig(speech_config=speech_config)

runner.run_live(
    ...,
    run_config=run_config,
)
```