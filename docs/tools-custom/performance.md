# Mejora el rendimiento de las herramientas con ejecución paralela

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v1.10.0</span>
</div>

A partir de la versión 1.10.0 del Agent Development Kit (ADK) para Python, el framework
intenta ejecutar cualquier 
[herramienta de función](/adk-docs/tools-custom/function-tools/) 
solicitada por el agente en paralelo. Este comportamiento puede mejorar significativamente el rendimiento y
la capacidad de respuesta de tus agentes, particularmente para agentes que dependen de múltiples
APIs externas o tareas de larga duración. Por ejemplo, si tienes 3 herramientas que cada una
tarda 2 segundos, al ejecutarlas en paralelo, el tiempo total de ejecución será
más cercano a 2 segundos, en lugar de 6 segundos. La capacidad de ejecutar funciones de herramientas
en paralelo puede mejorar el rendimiento de tus agentes, particularmente en los
siguientes escenarios:

-   **Tareas de investigación:** Donde el agente recopila información de múltiples
    fuentes antes de proceder a la siguiente etapa del flujo de trabajo.
-   **Llamadas a API:** Donde el agente accede a varias APIs de forma independiente, como
    buscar vuelos disponibles usando APIs de múltiples aerolíneas.
-   **Tareas de publicación y comunicación:** Cuando el agente necesita publicar
    o comunicarse a través de múltiples canales independientes o múltiples destinatarios.

Sin embargo, tus herramientas personalizadas deben estar construidas con soporte de ejecución asíncrona para
habilitar esta mejora de rendimiento. Esta guía explica cómo funciona la ejecución paralela de herramientas
en el ADK y cómo construir tus herramientas para aprovechar al máximo esta
característica de procesamiento.

!!! warning
    Cualquier herramienta de ADK que use procesamiento síncrono en un conjunto de llamadas a funciones de herramientas
    bloqueará la ejecución paralela de otras herramientas, incluso si las otras
    herramientas permiten la ejecución paralela.

## Construye herramientas listas para paralelización

Habilita la ejecución paralela de tus funciones de herramientas definiéndolas como
funciones asíncronas. En código Python, esto significa usar la sintaxis `async def` y `await`
que permite al ADK ejecutarlas concurrentemente en un bucle de eventos `asyncio`.
Las siguientes secciones muestran ejemplos de herramientas de agentes construidas para procesamiento
paralelo y operaciones asíncronas.

### Ejemplo de llamada web http

El siguiente ejemplo de código muestra cómo modificar la función `get_weather()` para
operar de forma asíncrona y permitir la ejecución paralela:

```python
 async def get_weather(city: str) -> dict:
      async with aiohttp.ClientSession() as session:
          async with session.get(f"http://api.weather.com/{city}") as response:
              return await response.json()
```

### Ejemplo de llamada a base de datos

El siguiente ejemplo de código muestra cómo escribir una función de llamada a base de datos para
operar de forma asíncrona:

```python
async def query_database(query: str) -> list:
      async with asyncpg.connect("postgresql://...") as conn:
          return await conn.fetch(query)
```

### Ejemplo de comportamiento de cesión para bucles largos

En casos donde una herramienta está procesando múltiples solicitudes o numerosas solicitudes
de larga duración, considera agregar código de cesión para permitir que otras herramientas se ejecuten, como
se muestra en el siguiente ejemplo de código:

```python
async def process_data(data: list) -> dict:
      results = []
      for i, item in enumerate(data):
          processed = await process_item(item)  # Punto de cesión
          results.append(processed)

          # Agregar puntos de cesión periódicos para bucles largos
          if i % 100 == 0:
              await asyncio.sleep(0)  # Ceder el control
      return {"results": results}
```

!!! tip "Importante"
    Usa la función `asyncio.sleep()` para pausas para evitar bloquear la
    ejecución de otras funciones.

### Ejemplo de pools de hilos para operaciones intensivas

Cuando realices funciones intensivas en procesamiento, considera crear pools de hilos
para una mejor gestión de los recursos de computación disponibles, como se muestra en el
siguiente ejemplo:

```python
async def cpu_intensive_tool(data: list) -> dict:
      loop = asyncio.get_event_loop()

      # Usar pool de hilos para trabajo intensivo de CPU
      with ThreadPoolExecutor() as executor:
          result = await loop.run_in_executor(
              executor,
              expensive_computation,
              data
          )
      return {"result": result}
```

### Ejemplo de división en fragmentos del procesamiento

Cuando realices procesos en listas largas o grandes cantidades de datos, considera
combinar una técnica de pool de hilos con dividir el procesamiento en fragmentos de
datos, y ceder tiempo de procesamiento entre los fragmentos, como se muestra en el siguiente
ejemplo:

```python
 async def process_large_dataset(dataset: list) -> dict:
      results = []
      chunk_size = 1000

      for i in range(0, len(dataset), chunk_size):
          chunk = dataset[i:i + chunk_size]

          # Procesar fragmento en pool de hilos
          loop = asyncio.get_event_loop()
          with ThreadPoolExecutor() as executor:
              chunk_result = await loop.run_in_executor(
                  executor, process_chunk, chunk
              )

          results.extend(chunk_result)

          # Ceder el control entre fragmentos
          await asyncio.sleep(0)

      return {"total_processed": len(results), "results": results}
```

## Escribe prompts y descripciones de herramientas listas para paralelización

Al construir prompts para modelos de IA, considera especificar explícitamente o insinuar
que las llamadas a funciones se realicen en paralelo. El siguiente ejemplo de un prompt de IA
dirige al modelo a usar herramientas en paralelo:

```none
When users ask for multiple pieces of information, always call functions in
parallel.

  Examples:
  - "Get weather for London and currency rate USD to EUR" → Call both functions
    simultaneously
  - "Compare cities A and B" → Call get_weather, get_population, get_distance in 
    parallel
  - "Analyze multiple stocks" → Call get_stock_price for each stock in parallel

  Always prefer multiple specific function calls over single complex calls.
```

El siguiente ejemplo muestra una descripción de función de herramienta que insinúa un uso más
eficiente a través de la ejecución paralela:

```python
 async def get_weather(city: str) -> dict:
      """Get current weather for a single city.

      This function is optimized for parallel execution - call multiple times for different cities.

      Args:
          city: Name of the city, for example: 'London', 'New York'

      Returns:
          Weather data including temperature, conditions, humidity
      """
      await asyncio.sleep(2)  # Simular llamada a API
      return {"city": city, "temp": 72, "condition": "sunny"}
```

## Próximos pasos

Para más información sobre la construcción de herramientas para agentes y llamadas a funciones, consulta
[Herramientas de Función](/adk-docs/tools-custom/function-tools/). Para
ejemplos más detallados de herramientas que aprovechan el procesamiento paralelo, consulta
las muestras en el repositorio
[adk-python](https://github.com/google/adk-python/tree/main/contributing/samples/parallel_functions).