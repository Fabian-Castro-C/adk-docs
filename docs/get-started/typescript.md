# Inicio Rápido de TypeScript para ADK

Esta guía te muestra cómo comenzar a trabajar con Agent Development Kit
para TypeScript. Antes de comenzar, asegúrate de tener lo siguiente instalado:

*   Node.js 24.13.0 o posterior
*   Node Package Manager (npm) 11.8.0 o posterior

## Crear un proyecto de agente

Crea un directorio vacío `my-agent` para tu proyecto:

```none
my-agent/
```

??? tip "Crea esta estructura de proyecto usando la línea de comandos"

    === "MacOS / Linux"

        ```bash
        mkdir -p my-agent/
        ```

    === "Windows"

        ```console
        mkdir my-agent
        ```

### Configurar proyecto y dependencias

Usa la herramienta `npm` para instalar y configurar dependencias para tu proyecto,
incluyendo el archivo de paquete, la biblioteca principal de ADK TypeScript
y las herramientas de desarrollo. Ejecuta los siguientes comandos desde tu
directorio `my-agent/` para crear el archivo `package.json` e instalar las
dependencias del proyecto:

```console
cd my-agent/
# initialize a project as an ES module
npm init --yes
npm pkg set type="module"
npm pkg set main="agent.ts"
# install ADK libraries
npm install @google/adk
# install dev tools as a dev dependency
npm install -D @google/adk-devtools
```

### Definir el código del agente

Crea el código para un agente básico, incluyendo una implementación simple de una
[Function Tool](/adk-docs/tools/function-tools/) de ADK, llamada `getCurrentTime`.
Crea un archivo `agent.ts` en el directorio de tu proyecto y agrega el siguiente código:

```typescript title="my-agent/agent.ts"
import {FunctionTool, LlmAgent} from '@google/adk';
import {z} from 'zod';

/* Implementación simulada de herramienta */
const getCurrentTime = new FunctionTool({
  name: 'get_current_time',
  description: 'Returns the current time in a specified city.',
  parameters: z.object({
    city: z.string().describe("The name of the city for which to retrieve the current time."),
  }),
  execute: ({city}) => {
    return {status: 'success', report: `The current time in ${city} is 10:30 AM`};
  },
});

export const rootAgent = new LlmAgent({
  name: 'hello_time_agent',
  model: 'gemini-2.5-flash',
  description: 'Tells the current time in a specified city.',
  instruction: `You are a helpful assistant that tells the current time in a city.
                Use the 'getCurrentTime' tool for this purpose.`,
  tools: [getCurrentTime],
});
```

### Configurar tu clave API

Este proyecto utiliza la API de Gemini, que requiere una clave API. Si
aún no tienes una clave API de Gemini, crea una clave en Google AI Studio en la
página de [API Keys](https://aistudio.google.com/app/apikey).

En una ventana de terminal, escribe tu clave API en el archivo `.env` de tu proyecto
para establecer las variables de entorno:

```bash title="Update: my-agent/.env"
echo 'GEMINI_API_KEY="YOUR_API_KEY"' > .env
```

??? tip "Usar otros modelos de IA con ADK"
    ADK soporta el uso de muchos modelos de IA generativa. Para más
    información sobre cómo configurar otros modelos en agentes ADK, consulta
    [Models & Authentication](/adk-docs/agents/models).

## Ejecutar tu agente

Puedes ejecutar tu agente ADK con la biblioteca `@google/adk-devtools` como una
interfaz de línea de comandos interactiva usando el comando `run` o la interfaz de usuario
web de ADK usando el comando `web`. Ambas opciones te permiten probar e
interactuar con tu agente.

### Ejecutar con interfaz de línea de comandos

Ejecuta tu agente con la herramienta de interfaz de línea de comandos de ADK TypeScript
usando el siguiente comando:

```console
npx adk run agent.ts
```

![adk-run.png](/adk-docs/assets/adk-run.png)

### Ejecutar con interfaz web

Ejecuta tu agente con la interfaz web de ADK usando el siguiente comando:

```console
npx adk web
```

Este comando inicia un servidor web con una interfaz de chat para tu agente. Puedes
acceder a la interfaz web en (http://localhost:8000). Selecciona tu agente en la
esquina superior derecha y escribe una solicitud.

![adk-web-dev-ui-chat.png](/adk-docs/assets/adk-web-dev-ui-chat.png)

!!! warning "Precaución: ADK Web solo para desarrollo"

    ADK Web ***no está destinado para uso en implementaciones de producción***. Deberías
    usar ADK Web solo para propósitos de desarrollo y depuración.

## Siguiente: Construye tu agente

Ahora que tienes ADK instalado y tu primer agente ejecutándose, intenta construir
tu propio agente con nuestras guías de construcción:

*  [Build your agent](/adk-docs/tutorials/)