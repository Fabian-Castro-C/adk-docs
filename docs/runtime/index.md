# Tiempo de Ejecución del Agente

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span><span class="lst-go">Go v0.1.0</span><span class="lst-java">Java v0.1.0</span>
</div>

ADK proporciona varias formas de ejecutar y probar tus agentes durante el desarrollo. Elige
el método que mejor se adapte a tu flujo de trabajo de desarrollo.

## Formas de ejecutar agentes

<div class="grid cards" markdown>

-   :material-web:{ .lg .middle } **Interfaz de Desarrollo**

    ---

    Usa `adk web` para lanzar una interfaz basada en navegador para interactuar con tus
    agentes.

    [:octicons-arrow-right-24: Usar la Interfaz Web](web-interface.md)

-   :material-console:{ .lg .middle } **Línea de Comandos**

    ---

    Usa `adk run` para interactuar con tus agentes directamente en la terminal.

    [:octicons-arrow-right-24: Usar la Línea de Comandos](command-line.md)

-   :material-api:{ .lg .middle } **Servidor API**

    ---

    Usa `adk api_server` para exponer tus agentes a través de una API RESTful.

    [:octicons-arrow-right-24: Usar el Servidor API](api-server.md)

</div>

## Referencia técnica

Para obtener información más detallada sobre la configuración y el comportamiento del tiempo de ejecución, consulta estas
páginas:

- **[Bucle de Eventos](event-loop.md)**: Comprende el bucle de eventos central que impulsa
  ADK, incluyendo el ciclo de yield/pause/resume.
- **[Reanudar Agentes](resume.md)**: Aprende cómo reanudar la ejecución del agente desde un
  estado anterior.
- **[Configuración de Tiempo de Ejecución](runconfig.md)**: Configura el comportamiento del tiempo de ejecución con
  RunConfig.