---
catalog_title: AG-UI
catalog_description: Crea interfaces de usuario de chat interactivas con streaming, sincronización de estado y acciones agénticas
catalog_icon: /adk-docs/assets/tools-ag-ui.png
---

# Crea experiencias de chat con AG-UI y CopilotKit

Convierte tus agentes ADK en aplicaciones completas con interfaces de usuario ricas y responsivas.
[AG-UI](https://docs.ag-ui.com/) es un protocolo abierto que maneja eventos de
streaming, estado del cliente y comunicación bidireccional entre tus agentes y
usuarios.

[AG-UI](https://github.com/ag-ui-protocol/ag-ui) proporciona una interfaz consistente
para potenciar clientes ricos a través de pilas tecnológicas, desde móviles hasta la web e
incluso la línea de comandos. Hay varios clientes diferentes que soportan
AG-UI:

- [CopilotKit](https://copilotkit.ai) proporciona herramientas y componentes para integrar
  estrechamente tu agente con aplicaciones web
- Clientes para
  [Kotlin](https://github.com/ag-ui-protocol/ag-ui/tree/main/sdks/community/kotlin),
  [Java](https://github.com/ag-ui-protocol/ag-ui/tree/main/sdks/community/java),
  [Go](https://github.com/ag-ui-protocol/ag-ui/tree/main/sdks/community/go/example/client),
  e [implementaciones
  CLI](https://github.com/ag-ui-protocol/ag-ui/tree/main/apps/client-cli-example/src)
  en TypeScript

Este tutorial usa CopilotKit para crear una aplicación de ejemplo respaldada por un agente ADK que
demuestra algunas de las características soportadas por AG-UI.

## Inicio rápido

Para empezar, vamos a crear una aplicación de ejemplo con un agente ADK y un cliente
web simple:

1. Crea la aplicación:

    ```bash
    npx copilotkit@latest create -f adk
    ```

2. Establece tu clave de API de Google:

    ```bash
    export GOOGLE_API_KEY="your-api-key"
    ```

3. Instala las dependencias y ejecuta:

    ```bash
    npm install && npm run dev
    ```

Esto inicia dos servidores:

- **http://localhost:3000** - La interfaz de usuario web (abre esto en tu navegador)
- **http://localhost:8000** - La API del agente ADK (solo backend)

Abre [http://localhost:3000](http://localhost:3000) en tu navegador para chatear con
tu agente.

## Características

### Chat

El chat es una interfaz familiar para exponer tu agente, y AG-UI maneja
el streaming de mensajes entre tus usuarios y agentes:

```tsx title="src/app/page.tsx"
<CopilotSidebar
  clickOutsideToClose={false}
  defaultOpen={true}
  labels={{
    title: "Popup Assistant",
    initial: "👋 Hi, there! You're chatting with an agent. This agent comes with a few tools to get you started..."
  }}
/>
```

Aprende más sobre la interfaz de usuario de chat
[en los documentos de CopilotKit](https://docs.copilotkit.ai/adk/agentic-chat-ui).

### Interfaz de Usuario Generativa

AG-UI te permite compartir información de herramientas con una interfaz de usuario generativa para que pueda ser
mostrada a los usuarios:

```tsx title="src/app/page.tsx"
useRenderToolCall(
  {
    name: "get_weather",
    description: "Get the weather for a given location.",
    parameters: [{ name: "location", type: "string", required: true }],
    render: ({ args }) => {
      return <WeatherCard location={args.location} themeColor={themeColor} />;
    },
  },
  [themeColor],
);
```

Aprende más sobre la interfaz de usuario generativa
[en los documentos de CopilotKit](https://docs.copilotkit.ai/adk/generative-ui).

### Estado compartido

Los agentes ADK pueden tener estado, y sincronizar ese estado entre tus agentes y
tus interfaces de usuario habilita experiencias de usuario poderosas y fluidas. El estado puede ser sincronizado
en ambas direcciones para que los agentes sean automáticamente conscientes de los cambios realizados por tu usuario u
otras partes de tu aplicación:

```tsx title="src/app/page.tsx"
const { state, setState } = useCoAgent<AgentState>({
  name: "my_agent",
  initialState: {
    proverbs: [
      "A journey of a thousand miles begins with a single step.",
    ],
  },
})
```

Aprende más sobre el estado compartido
[en los documentos de CopilotKit](https://docs.copilotkit.ai/adk/shared-state).

## Recursos

Para ver qué otras características puedes construir en tu interfaz de usuario con AG-UI, consulta los
documentos de CopilotKit:

- [Interfaz de Usuario Generativa Agéntica](https://docs.copilotkit.ai/adk/generative-ui/agentic)
- [Humano en el Bucle](https://docs.copilotkit.ai/adk/human-in-the-loop)
- [Acciones de Frontend](https://docs.copilotkit.ai/adk/frontend-actions)

O pruébalas en el [AG-UI Dojo](https://dojo.ag-ui.com).