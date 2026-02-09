# Introducción a A2A

A medida que construyes sistemas agénticos más complejos, descubrirás que un solo agente
a menudo no es suficiente. Querrás crear agentes especializados que puedan
colaborar para resolver un problema. El [**Protocolo Agent2Agent (A2A)**](https://a2a-protocol.org) es el
estándar que permite que estos agentes se comuniquen entre sí.

## Cuándo Usar A2A vs. Sub-Agentes Locales

- **Sub-Agentes Locales:** Estos son agentes que se ejecutan *dentro del mismo proceso de aplicación*
  que tu agente principal. Son como módulos internos o bibliotecas, usados
  para organizar tu código en componentes lógicos y reutilizables. La comunicación entre
  un agente principal y sus sub-agentes locales es muy rápida porque ocurre
  directamente en memoria, sin sobrecarga de red.

- **Agentes Remotos (A2A):** Estos son agentes independientes que se ejecutan como servicios
  separados, comunicándose a través de una red. A2A define el protocolo estándar
  para esta comunicación.

Considera usar **A2A** cuando:

- El agente con el que necesitas comunicarte es un **servicio separado e independiente** (por ejemplo, un
  agente especializado en modelado financiero).
- El agente es mantenido por un **equipo u organización diferente**.
- Necesitas conectar agentes escritos en **diferentes lenguajes de programación o
  frameworks de agentes**.
- Quieres imponer un **contrato fuerte y formal** (el protocolo A2A) entre
  los componentes de tu sistema.

### Cuándo Usar A2A: Ejemplos Concretos

- **Integración con un Servicio de Terceros:** Tu agente principal necesita obtener
  precios de acciones en tiempo real de un proveedor externo de datos financieros. Este
  proveedor expone sus datos a través de un agente compatible con A2A.
- **Arquitectura de Microservicios:** Tienes un sistema grande dividido en
  servicios más pequeños e independientes (por ejemplo, un Agente de Procesamiento de Pedidos, un Agente de
  Gestión de Inventario, un Agente de Envío). A2A es ideal para que estos servicios
  se comuniquen entre sí a través de límites de red.
- **Comunicación Entre Lenguajes:** Tu lógica de negocio principal está en un agente
  Python, pero tienes un sistema heredado o un componente especializado escrito en Java
  que quieres integrar como un agente. A2A proporciona la capa de
  comunicación estandarizada.
- **Aplicación de API Formal:** Estás construyendo una plataforma donde diferentes equipos
  contribuyen agentes, y necesitas un contrato estricto sobre cómo estos agentes
  interactúan para asegurar compatibilidad y estabilidad.

### Cuándo NO Usar A2A: Ejemplos Concretos (Prefiere Sub-Agentes Locales)

- **Organización de Código Interno:** Estás dividiendo una tarea compleja dentro de un
  solo agente en funciones o módulos más pequeños y manejables (por ejemplo, un
  sub-agente `DataValidator` que limpia datos de entrada antes de procesarlos). Estos se
  manejan mejor como sub-agentes locales por rendimiento y simplicidad.
- **Operaciones Internas Críticas de Rendimiento:** Un sub-agente es responsable de una
  operación de alta frecuencia y baja latencia que está estrechamente acoplada con la
  ejecución del agente principal (por ejemplo, un sub-agente `RealTimeAnalytics` que procesa flujos de datos
  dentro de la misma aplicación).
- **Memoria/Contexto Compartido:** Cuando los sub-agentes necesitan acceso directo al
  estado interno del agente principal o memoria compartida por eficiencia, la sobrecarga de red de A2A y la
  serialización/deserialización serían contraproducentes.
- **Funciones Auxiliares Simples:** Para piezas pequeñas y reutilizables de lógica que no
  requieren despliegue independiente o gestión de estado compleja, una simple función
  o clase dentro del mismo agente es más apropiada que un agente A2A separado.

## El Flujo de Trabajo A2A en ADK: Una Vista Simplificada

Agent Development Kit (ADK) simplifica el proceso de construir y conectar
agentes usando el protocolo A2A. Aquí hay un desglose sencillo de cómo
funciona:

1. **Hacer un Agente Accesible (Exponer):** Comienzas con un agente ADK
    existente con el que quieres que otros agentes puedan interactuar. El ADK
    proporciona una forma simple de "exponer" este agente, convirtiéndolo en un
    **A2AServer**. Este servidor actúa como una interfaz pública, permitiendo que otros agentes
    envíen solicitudes a tu agente a través de una red. Piensa en ello como configurar un
    servidor web para tu agente.

2. **Conectarse a un Agente Accesible (Consumir):** En un agente separado
    (que podría estar ejecutándose en la misma máquina o en una diferente), usarás
    un componente especial de ADK llamado `RemoteA2aAgent`. Este `RemoteA2aAgent` actúa
    como un cliente que sabe cómo comunicarse con el **A2AServer** que
    expusiste anteriormente. Maneja todas las complejidades de comunicación de red,
    autenticación y formateo de datos detrás de escena.

Desde tu perspectiva como desarrollador, una vez que has configurado esta conexión,
interactuar con el agente remoto se siente como interactuar con una herramienta
o función local. El ADK abstrae la capa de red, haciendo que los sistemas de agentes distribuidos
sean tan fáciles de trabajar como los locales.

## Visualizando el Flujo de Trabajo A2A

Para aclarar aún más el flujo de trabajo A2A, veamos el "antes y después" tanto para
exponer como consumir agentes, y luego el sistema combinado.

### Exponiendo un Agente

**Antes de Exponer:**
Tu código de agente se ejecuta como un componente independiente, pero en este escenario, quieres
exponerlo para que otros agentes remotos puedan interactuar con tu agente.

```text
+-------------------+
| Your Agent Code   |
|   (Standalone)    |
+-------------------+
```

**Después de Exponer:**
Tu código de agente se integra con un `A2AServer` (un componente ADK), haciéndolo
accesible a través de una red a otros agentes remotos.

```text
+-----------------+
|   A2A Server    |
| (ADK Component) |<--------+
+-----------------+         |
        |                   |
        v                   |
+-------------------+       |
| Your Agent Code   |       |
| (Now Accessible)  |       |
+-------------------+       |
                            |
                            | (Network Communication)
                            v
+-----------------------------+
|       Remote Agent(s)       |
|    (Can now communicate)    |
+-----------------------------+
```

### Consumiendo un Agente

**Antes de Consumir:**
Tu agente (referido como el "Agente Raíz" en este contexto) es la aplicación
que estás desarrollando que necesita interactuar con un agente remoto. Antes de
consumir, carece del mecanismo directo para hacerlo.

```text
+----------------------+         +-------------------------------------------------------------+
|      Root Agent      |         |                        Remote Agent                         |
| (Your existing code) |         | (External Service that you want your Root Agent to talk to) |
+----------------------+         +-------------------------------------------------------------+
```

**Después de Consumir:**
Tu Agente Raíz usa un `RemoteA2aAgent` (un componente ADK que actúa como un
proxy del lado del cliente para el agente remoto) para establecer comunicación con el
agente remoto.

```text
+----------------------+         +-----------------------------------+
|      Root Agent      |         |         RemoteA2aAgent            |
| (Your existing code) |<------->|         (ADK Client Proxy)        |
+----------------------+         |                                   |
                                 |  +-----------------------------+  |
                                 |  |         Remote Agent        |  |
                                 |  |      (External Service)     |  |
                                 |  +-----------------------------+  |
                                 +-----------------------------------+
      (Now talks to remote agent via RemoteA2aAgent)
```

### Sistema Final (Vista Combinada)

Este diagrama muestra cómo las partes de consumo y exposición se conectan para formar un
sistema A2A completo.

```text
Consuming Side:
+----------------------+         +-----------------------------------+
|      Root Agent      |         |         RemoteA2aAgent            |
| (Your existing code) |<------->|         (ADK Client Proxy)        |
+----------------------+         |                                   |
                                 |  +-----------------------------+  |
                                 |  |         Remote Agent        |  |
                                 |  |      (External Service)     |  |
                                 |  +-----------------------------+  |
                                 +-----------------------------------+
                                                 |
                                                 | (Network Communication)
                                                 v
Exposing Side:
                                               +-----------------+
                                               |   A2A Server    |
                                               | (ADK Component) |
                                               +-----------------+
                                                       |
                                                       v
                                               +-------------------+
                                               | Your Agent Code   |
                                               | (Exposed Service) |
                                               +-------------------+
```

## Caso de Uso Concreto: Agentes de Servicio al Cliente y Catálogo de Productos

Consideremos un ejemplo práctico: un **Agente de Servicio al Cliente** que necesita
recuperar información de productos de un **Agente de Catálogo de Productos** separado.

### Antes de A2A

Inicialmente, tu Agente de Servicio al Cliente podría no tener una forma directa y estandarizada
de consultar el Agente de Catálogo de Productos, especialmente si es un servicio separado
o administrado por un equipo diferente.

```text
+-------------------------+         +--------------------------+
| Customer Service Agent  |         |  Product Catalog Agent   |
| (Needs Product Info)    |         | (Contains Product Data)  |
+-------------------------+         +--------------------------+
      (No direct, standardized communication)
```

### Después de A2A

Al usar el Protocolo A2A, el Agente de Catálogo de Productos puede exponer su
funcionalidad como un servicio A2A. Tu Agente de Servicio al Cliente puede entonces fácilmente
consumir este servicio usando `RemoteA2aAgent` de ADK.

```text
+-------------------------+         +-----------------------------------+
| Customer Service Agent  |         |         RemoteA2aAgent            |
| (Your Root Agent)       |<------->|         (ADK Client Proxy)        |
+-------------------------+         |                                   |
                                    |  +-----------------------------+  |
                                    |  |     Product Catalog Agent   |  |
                                    |  |      (External Service)     |  |
                                    |  +-----------------------------+  |
                                    +-----------------------------------+
                                                 |
                                                 | (Network Communication)
                                                 v
                                               +-----------------+
                                               |   A2A Server    |
                                               | (ADK Component) |
                                               +-----------------+
                                                       |
                                                       v
                                               +------------------------+
                                               | Product Catalog Agent  |
                                               | (Exposed Service)      |
                                               +------------------------+
```

En esta configuración, primero, el Agente de Catálogo de Productos necesita ser expuesto a través de un
Servidor A2A. Luego, el Agente de Servicio al Cliente puede simplemente llamar métodos en el
`RemoteA2aAgent` como si fuera una herramienta, y el ADK maneja toda la
comunicación subyacente con el Agente de Catálogo de Productos. Esto permite una clara separación de
responsabilidades y una fácil integración de agentes especializados.

## Próximos Pasos

Ahora que entiendes el "por qué" de A2A, vamos a profundizar en el "cómo."

- **Continúa con la siguiente guía:**
  [Inicio Rápido: Exponiendo Tu Agente](./quickstart-exposing.md)