---
catalog_title: GKE Code Executor
catalog_description: Run AI-generated code in a secure and scalable GKE environment
catalog_icon: /adk-docs/assets/tools-gke.png
---

# Herramienta GKE Code Executor para ADK

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v1.14.0</span>
</div>

El GKE Code Executor (`GkeCodeExecutor`) proporciona un método seguro y escalable
para ejecutar código generado por LLM aprovechando el entorno Sandbox de GKE 
(Google Kubernetes Engine), que utiliza gVisor para el aislamiento de cargas de 
trabajo. Para cada solicitud de ejecución de código, crea dinámicamente un Job de 
Kubernetes efímero y aislado con una configuración de Pod reforzada. Deberías usar 
este ejecutor para entornos de producción en GKE donde la seguridad y el aislamiento 
son críticos.

## Cómo Funciona

Cuando se realiza una solicitud para ejecutar código, el `GkeCodeExecutor` realiza los siguientes pasos:

1.  **Crea un ConfigMap:** Se crea un ConfigMap de Kubernetes para almacenar el código Python que necesita ser ejecutado.
2.  **Crea un Pod Aislado:** Se crea un nuevo Job de Kubernetes, que a su vez crea un Pod con un contexto de seguridad reforzado y el runtime gVisor habilitado. El código del ConfigMap se monta en este Pod.
3.  **Ejecuta el Código:** El código se ejecuta dentro del Pod aislado, aislado del nodo subyacente y otras cargas de trabajo.
4.  **Recupera el Resultado:** Los flujos de salida estándar y error de la ejecución se capturan de los logs del Pod.
5.  **Limpia los Recursos:** Una vez que la ejecución se completa, el Job y el ConfigMap asociado se eliminan automáticamente, asegurando que no queden artefactos.

## Beneficios Clave

*   **Seguridad Mejorada:** El código se ejecuta en un entorno aislado por gVisor con aislamiento a nivel de kernel.
*   **Entornos Efímeros:** Cada ejecución de código se ejecuta en su propio Pod efímero, para prevenir la transferencia de estado entre ejecuciones.
*   **Control de Recursos:** Puedes configurar límites de CPU y memoria para los Pods de ejecución para prevenir el abuso de recursos.
*   **Escalabilidad:** Te permite ejecutar un gran número de ejecuciones de código en paralelo, con GKE manejando la programación y escalado de los nodos subyacentes.

## Requisitos del Sistema

Los siguientes requisitos deben cumplirse para desplegar exitosamente tu proyecto ADK
con la herramienta GKE Code Executor:

- Clúster GKE con un **node pool habilitado con gVisor**.
- La cuenta de servicio del agente requiere **permisos RBAC** específicos, que le permiten:
    - Crear, observar y eliminar **Jobs** para cada solicitud de ejecución.
    - Administrar **ConfigMaps** para inyectar código en el pod del Job.
    - Listar **Pods** y leer sus **logs** para recuperar el resultado de la ejecución
- Instalar la biblioteca cliente con extras de GKE: `pip install google-adk[gke]`

Para una configuración completa y lista para usar, consulta la muestra
[deployment_rbac.yaml](https://github.com/google/adk-python/blob/main/contributing/samples/gke_agent_sandbox/deployment_rbac.yaml). 
Para más información sobre el despliegue de flujos de trabajo ADK en GKE, consulta
[Desplegar en Google Kubernetes Engine (GKE)](/adk-docs/deploy/gke/).

=== "Python"

    ```python
    from google.adk.agents import LlmAgent
    from google.adk.code_executors import GkeCodeExecutor

    # Inicializa el ejecutor, apuntando al namespace donde su ServiceAccount
    # tiene los permisos RBAC requeridos.
    # Este ejemplo también establece un timeout personalizado y límites de recursos.
    gke_executor = GkeCodeExecutor(
        namespace="agent-sandbox",
        timeout_seconds=600,
        cpu_limit="1000m",  # 1 núcleo de CPU
        mem_limit="1Gi",
    )

    # El agente ahora usa este ejecutor para cualquier código que genere.
    gke_agent = LlmAgent(
        name="gke_coding_agent",
        model="gemini-2.0-flash",
        instruction="You are a helpful AI agent that writes and executes Python code.",
        code_executor=gke_executor,
    )
    ```

## Parámetros de Configuración

El `GkeCodeExecutor` puede ser configurado con los siguientes parámetros:

| Parámetro            | Tipo   | Descripción                                                                             |
| -------------------- | ------ | --------------------------------------------------------------------------------------- |
| `namespace`          | `str`  | Namespace de Kubernetes donde se crearán los Jobs de ejecución. Por defecto es `"default"`. |
| `image`              | `str`  | Imagen de contenedor a usar para el Pod de ejecución. Por defecto es `"python:3.11-slim"`.         |
| `timeout_seconds`    | `int`  | Timeout en segundos para la ejecución de código. Por defecto es `300`.                           |
| `cpu_requested`      | `str`  | Cantidad de CPU a solicitar para el Pod de ejecución. Por defecto es `"200m"`.                   |
| `mem_requested`      | `str`  | Cantidad de memoria a solicitar para el Pod de ejecución. Por defecto es `"256Mi"`.               |
| `cpu_limit`          | `str`  | Cantidad máxima de CPU que puede usar el Pod de ejecución. Por defecto es `"500m"`.                  |
| `mem_limit`          | `str`  | Cantidad máxima de memoria que puede usar el Pod de ejecución. Por defecto es `"512Mi"`.              |
| `kubeconfig_path`    | `str`  | Ruta a un archivo kubeconfig para usar en la autenticación. Recurre a la configuración in-cluster o al kubeconfig local por defecto. |
| `kubeconfig_context` | `str`  | El contexto de `kubeconfig` a usar.  |