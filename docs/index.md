---
hide:
  - toc
---

<div style="text-align: center;">
  <div class="centered-logo-text-group">
    <img src="assets/agent-development-kit.png" alt="Logo de Agent Development Kit" width="100">
    <h1>Agent Development Kit</h1>
  </div>
</div>

Agent Development Kit (ADK) es un framework flexible y modular para **desarrollar
y desplegar agentes de IA**. Si bien está optimizado para Gemini y el ecosistema de Google,
ADK es **agnóstico del modelo**, **agnóstico del despliegue**, y está construido para
**compatibilidad con otros frameworks**. ADK fue diseñado para hacer que el
desarrollo de agentes se sienta más como desarrollo de software, para facilitar a los
desarrolladores crear, desplegar y orquestar arquitecturas agénticas que van
desde tareas simples hasta flujos de trabajo complejos.

<div id="centered-install-tabs" class="install-command-container" markdown="1">

<p class="get-started-text" style="text-align: center;">Comienza:</p>

=== "Python"
    <br>
    <p style="text-align: center;">
    <code>pip install google-adk</code>
    </p>

=== "TypeScript"
    <br>
    <p style="text-align: center;">
    <code>npm install @google/adk</code>
    </p>

=== "Go"
    <br>
    <p style="text-align: center;">
    <code>go get google.golang.org/adk</code>
    </p>

=== "Java"

    ```xml title="pom.xml"
    <dependency>
        <groupId>com.google.adk</groupId>
        <artifactId>google-adk</artifactId>
        <version>0.5.0</version>
    </dependency>
    ```

    ```gradle title="build.gradle"
    dependencies {
        implementation 'com.google.adk:google-adk:0.5.0'
    }
    ```

</div>

<p style="text-align:center;">
  <a href="/adk-docs/get-started/python/" class="md-button" style="margin:3px">Comenzar con Python</a>
  <a href="/adk-docs/get-started/typescript/" class="md-button" style="margin:3px">Comenzar con TypeScript</a>
  <a href="/adk-docs/get-started/go/" class="md-button" style="margin:3px">Comenzar con Go</a>
  <a href="/adk-docs/get-started/java/" class="md-button" style="margin:3px">Comenzar con Java</a>
</p>

---

## Aprende más

[:fontawesome-brands-youtube:{.youtube-red-icon} ¡Mira "Presentando Agent Development Kit"!](https://www.youtube.com/watch?v=zgrOwow_uTQ){:target="_blank" rel="noopener noreferrer"}

<div class="grid cards" markdown>

-   :material-transit-connection-variant: **Orquestación Flexible**

    ---

    Define flujos de trabajo usando agentes de flujo de trabajo (`Sequential`, `Parallel`, `Loop`)
    para pipelines predecibles, o aprovecha el enrutamiento dinámico impulsado por LLM
    (transferencia `LlmAgent`) para comportamiento adaptativo.

    [**Aprende sobre agentes**](agents/index.md)

-   :material-graph: **Arquitectura Multi-Agente**

    ---

    Construye aplicaciones modulares y escalables componiendo múltiples agentes especializados
    en una jerarquía. Habilita coordinación y delegación complejas.

    [**Explora sistemas multi-agente**](agents/multi-agents.md)

-   :material-toolbox-outline: **Ecosistema Rico de Herramientas**

    ---

    Equipa a los agentes con diversas capacidades: usa herramientas pre-construidas (Search, Code
    Exec), crea funciones personalizadas, integra bibliotecas de terceros, o incluso usa
    otros agentes como herramientas.

    [**Explora herramientas**](tools/index.md)

-   :material-rocket-launch-outline: **Listo para Despliegue**

    ---

    Conteneriza y despliega tus agentes en cualquier lugar – ejecuta localmente, escala con
    Vertex AI Agent Engine, o integra en infraestructura personalizada usando Cloud
    Run o Docker.

    [**Despliega agentes**](deploy/index.md)

-   :material-clipboard-check-outline: **Evaluación Integrada**

    ---

    Evalúa sistemáticamente el rendimiento del agente evaluando tanto la calidad
    de la respuesta final como la trayectoria de ejecución paso a paso contra
    casos de prueba predefinidos.

    [**Evalúa agentes**](evaluate/index.md)

-   :material-console-line: **Construcción de Agentes Seguros y Protegidos**

    ---

    Aprende cómo construir agentes poderosos y confiables implementando
    patrones y mejores prácticas de seguridad y protección en el diseño de tu agente.

    [**Seguridad y Protección**](safety/index.md)

</div>