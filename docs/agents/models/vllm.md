# Host de modelos vLLM para agentes ADK

<div class="language-support-tag">
    <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span>
</div>

Herramientas como [vLLM](https://github.com/vllm-project/vllm) te permiten alojar
modelos de manera eficiente y servirlos como un endpoint de API compatible con OpenAI. Puedes
usar modelos vLLM a través de la biblioteca [LiteLLM](/agents/models/litellm/)
para Python.

## Configuración

1. **Desplegar Modelo:** Despliega tu modelo elegido usando vLLM (o una herramienta similar).
   Anota la URL base de la API (ej., `https://your-vllm-endpoint.run.app/v1`).
    * *Importante para Herramientas ADK:* Al desplegar, asegúrate de que la herramienta de servicio
      soporte y habilite el llamado de herramientas/funciones compatible con OpenAI. Para vLLM,
      esto podría involucrar flags como `--enable-auto-tool-choice` y potencialmente
      un `--tool-call-parser` específico, dependiendo del modelo. Consulta la documentación
      de vLLM sobre Uso de Herramientas.
2. **Autenticación:** Determina cómo tu endpoint maneja la autenticación (ej.,
   clave API, token bearer).

## Ejemplo de Integración

El siguiente ejemplo muestra cómo usar un endpoint vLLM con agentes ADK.

```python
import subprocess
from google.adk.agents import LlmAgent
from google.adk.models.lite_llm import LiteLlm

# --- Agente de ejemplo usando un modelo alojado en un endpoint vLLM ---

# URL del endpoint proporcionada por tu despliegue vLLM
api_base_url = "https://your-vllm-endpoint.run.app/v1"

# Nombre del modelo como es reconocido por *tu* configuración de endpoint vLLM
model_name_at_endpoint = "hosted_vllm/google/gemma-3-4b-it" # Ejemplo de vllm_test.py

# Autenticación (Ejemplo: usando token de identidad de gcloud para un despliegue en Cloud Run)
# Adapta esto según la seguridad de tu endpoint
try:
    gcloud_token = subprocess.check_output(
        ["gcloud", "auth", "print-identity-token", "-q"]
    ).decode().strip()
    auth_headers = {"Authorization": f"Bearer {gcloud_token}"}
except Exception as e:
    print(f"Warning: Could not get gcloud token - {e}. Endpoint might be unsecured or require different auth.")
    auth_headers = None # O maneja el error apropiadamente

agent_vllm = LlmAgent(
    model=LiteLlm(
        model=model_name_at_endpoint,
        api_base=api_base_url,
        # Pasa encabezados de autenticación si es necesario
        extra_headers=auth_headers
        # Alternativamente, si el endpoint usa una clave API:
        # api_key="YOUR_ENDPOINT_API_KEY"
    ),
    name="vllm_agent",
    instruction="You are a helpful assistant running on a self-hosted vLLM endpoint.",
    # ... otros parámetros del agente
)
```