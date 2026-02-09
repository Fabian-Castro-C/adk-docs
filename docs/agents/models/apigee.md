# Apigee AI Gateway para agentes ADK

<div class="language-support-tag">
   <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v1.18.0</span><span class="lst-java">Java v0.4.0</span>
</div>

[Apigee](https://docs.cloud.google.com/apigee/docs/api-platform/get-started/what-apigee)
proporciona un poderoso [AI Gateway](https://cloud.google.com/solutions/apigee-ai),
transformando la forma en que administras y gobiernas el tráfico de tus modelos de IA generativa. Al
exponer el endpoint de tu modelo de IA (como Vertex AI o la API de Gemini) a través de un
proxy de Apigee, obtienes inmediatamente capacidades de nivel empresarial:

- **Seguridad del Modelo:** Implementa políticas de seguridad como Model Armor para protección contra amenazas.

- **Gobernanza de Tráfico:** Aplica Rate Limiting y Token Limiting para gestionar costos y prevenir abusos.

- **Rendimiento:** Mejora los tiempos de respuesta y la eficiencia usando Semantic Caching y enrutamiento avanzado de modelos.

- **Monitoreo y Visibilidad:** Obtén monitoreo granular, análisis y auditoría de todas tus solicitudes de IA.

!!! note

    El wrapper `ApigeeLLM` está actualmente diseñado para uso con Vertex AI
    y la API de Gemini (generateContent). Estamos expandiendo continuamente el soporte para
    otros modelos e interfaces.

## Ejemplo de implementación

Integra la gobernanza de Apigee en el flujo de trabajo de tu agente instanciando el
objeto wrapper `ApigeeLlm` y pasándolo a un `LlmAgent` u otro tipo de agente.

=== "Python"

    ```python

    from google.adk.agents import LlmAgent
    from google.adk.models.apigee_llm import ApigeeLlm

    # Instancia el wrapper ApigeeLlm
    model = ApigeeLlm(
        # Especifica la ruta de Apigee hacia tu modelo. Para más información, consulta la documentación de ApigeeLlm (https://github.com/google/adk-python/tree/main/contributing/samples/hello_world_apigeellm).
        model="apigee/gemini-2.5-flash",
        # La URL del proxy de tu proxy Apigee desplegado incluyendo la ruta base
        proxy_url=f"https://{APIGEE_PROXY_URL}",
        # Pasa los encabezados necesarios de autenticación/autorización (como una clave API)
        custom_headers={"foo": "bar"}
    )

    # Pasa el wrapper del modelo configurado a tu LlmAgent
    agent = LlmAgent(
        model=model,
        name="my_governed_agent",
        instruction="You are a helpful assistant powered by Gemini and governed by Apigee.",
        # ... otros parámetros del agente
    )

    ```

=== "Java"

    ```java
    import com.google.adk.agents.LlmAgent;
    import com.google.adk.models.ApigeeLlm;
    import com.google.common.collect.ImmutableMap;

    ApigeeLlm apigeeLlm =
            ApigeeLlm.builder()
                .modelName("apigee/gemini-2.5-flash") // Especifica la ruta de Apigee hacia tu modelo. Para más información, consulta la documentación de ApigeeLlm
                .proxyUrl(APIGEE_PROXY_URL) //La URL del proxy de tu proxy Apigee desplegado incluyendo la ruta base
                .customHeaders(ImmutableMap.of("foo", "bar")) //Pasa los encabezados necesarios de autenticación/autorización (como una clave API)
                .build();
    LlmAgent agent =
        LlmAgent.builder()
            .model(apigeeLlm)
            .name("my_governed_agent")
            .description("my_governed_agent")
            .instruction("You are a helpful assistant powered by Gemini and governed by Apigee.")
            // las herramientas se añadirán a continuación
            .build();
    ```

Con esta configuración, cada llamada a la API desde tu agente será enrutada primero a través de
Apigee, donde se ejecutan todas las políticas necesarias (seguridad, limitación de tasa, registro)
antes de que la solicitud sea reenviada de forma segura al endpoint del modelo de IA
subyacente. Para un ejemplo de código completo usando el proxy de Apigee, consulta
[Hello World Apigee LLM](https://github.com/google/adk-python/tree/main/contributing/samples/hello_world_apigeellm).