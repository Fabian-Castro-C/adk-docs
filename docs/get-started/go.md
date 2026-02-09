# Inicio Rápido de Go para ADK

Esta guía te muestra cómo empezar a trabajar con el Kit de Desarrollo de Agentes
para Go. Antes de comenzar, asegúrate de tener lo siguiente instalado:

*   Go 1.24.4 o posterior
*   ADK Go v0.2.0 o posterior

## Crear un proyecto de agente

Crea un proyecto de agente con los siguientes archivos y estructura de directorios:

```none
my_agent/
    agent.go    # código principal del agente
    .env        # claves API o IDs de proyecto
```

??? tip "Crea esta estructura de proyecto usando la línea de comandos"

    === "Windows"

        ```console
        mkdir my_agent\
        type nul > my_agent\agent.go
        type nul > my_agent\env.bat
        ```

    === "MacOS / Linux"

        ```bash
        mkdir -p my_agent/ && \
            touch my_agent/agent.go && \
            touch my_agent/.env
        ```

### Definir el código del agente

Crea el código para un agente básico que utiliza la
[herramienta de Búsqueda de Google](/adk-docs/tools/built-in-tools/#google-search) integrada. Agrega el
siguiente código al archivo `my_agent/agent.go` en el directorio de tu proyecto:

```go title="my_agent/agent.go"
package main

import (
	"context"
	"log"
	"os"

	"google.golang.org/adk/agent"
	"google.golang.org/adk/agent/llmagent"
	"google.golang.org/adk/cmd/launcher"
	"google.golang.org/adk/cmd/launcher/full"
	"google.golang.org/adk/model/gemini"
	"google.golang.org/adk/tool"
	"google.golang.org/adk/tool/geminitool"
	"google.golang.org/genai"
)

func main() {
	ctx := context.Background()

	model, err := gemini.NewModel(ctx, "gemini-3-pro-preview", &genai.ClientConfig{
		APIKey: os.Getenv("GOOGLE_API_KEY"),
	})
	if err != nil {
		log.Fatalf("Failed to create model: %v", err) // Error al crear el modelo
	}

	timeAgent, err := llmagent.New(llmagent.Config{
		Name:        "hello_time_agent",
		Model:       model,
		Description: "Tells the current time in a specified city.", // Indica la hora actual en una ciudad especificada
		Instruction: "You are a helpful assistant that tells the current time in a city.", // Eres un asistente útil que indica la hora actual en una ciudad
		Tools: []tool.Tool{
			geminitool.GoogleSearch{},
		},
	})
	if err != nil {
		log.Fatalf("Failed to create agent: %v", err) // Error al crear el agente
	}

	config := &launcher.Config{
		AgentLoader: agent.NewSingleLoader(timeAgent),
	}

	l := full.NewLauncher()
	if err = l.Execute(ctx, config, os.Args[1:]); err != nil {
		log.Fatalf("Run failed: %v\n\n%s", err, l.CommandLineSyntax()) // Ejecución fallida
	}
}
```

### Configurar el proyecto y las dependencias

Usa el comando `go mod` para inicializar los módulos del proyecto e instalar los
paquetes requeridos basándose en la declaración `import` en tu archivo de código del agente:

```console
go mod init my-agent/main
go mod tidy
```

### Configurar tu clave API

Este proyecto utiliza la API de Gemini, que requiere una clave API. Si no
tienes ya una clave API de Gemini, crea una clave en Google AI Studio en la
página de [Claves API](https://aistudio.google.com/app/apikey).

En una ventana de terminal, escribe tu clave API en el archivo `.env` o `env.bat` de
tu proyecto para establecer las variables de entorno:

=== "MacOS / Linux"

    ```bash title="Actualizar: my_agent/.env"
    echo 'export GOOGLE_API_KEY="YOUR_API_KEY"' > .env
    ```

=== "Windows"

    ```console title="Actualizar: my_agent/env.bat"
    echo 'set GOOGLE_API_KEY="YOUR_API_KEY"' > env.bat
    ```

??? tip "Usar otros modelos de IA con ADK"
    ADK soporta el uso de muchos modelos de IA generativa. Para más
    información sobre cómo configurar otros modelos en agentes ADK, consulta
    [Modelos y Autenticación](/adk-docs/agents/models).


## Ejecutar tu agente

Puedes ejecutar tu agente ADK usando la interfaz de línea de comandos interactiva
que definiste o la interfaz de usuario web de ADK proporcionada por
la herramienta de línea de comandos de ADK Go. Ambas opciones te permiten probar e
interactuar con tu agente.

### Ejecutar con interfaz de línea de comandos

Ejecuta tu agente usando el siguiente comando de Go:

```console title="Ejecutar desde: directorio my_agent/"
# Recuerda cargar las claves y configuraciones: source .env O env.bat
go run agent.go
```

![adk-run.png](/adk-docs/assets/adk-run.png)

### Ejecutar con interfaz web

Ejecuta tu agente con la interfaz web de ADK usando el siguiente comando de Go:

```console title="Ejecutar desde: directorio my_agent/"
# Recuerda cargar las claves y configuraciones: source .env O env.bat
go run agent.go web api webui
```

Este comando inicia un servidor web con una interfaz de chat para tu agente. Puedes
acceder a la interfaz web en (http://localhost:8080). Selecciona tu agente en la
esquina superior izquierda y escribe una solicitud.

![adk-web-dev-ui-chat.png](/adk-docs/assets/adk-web-dev-ui-chat.png)

!!! warning "Precaución: ADK Web solo para desarrollo"

    ADK Web ***no está diseñado para uso en implementaciones de producción***. Deberías
    usar ADK Web solo para propósitos de desarrollo y depuración.

## Siguiente: Construir tu agente

Ahora que tienes ADK instalado y tu primer agente ejecutándose, intenta construir
tu propio agente con nuestras guías de construcción:

*  [Construir tu agente](/adk-docs/tutorials/)