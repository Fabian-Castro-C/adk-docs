---
catalog_title: Stripe
catalog_description: Manage payments, customers, subscriptions, and invoices
catalog_icon: /adk-docs/assets/tools-stripe.png
---

# Stripe

<div class="language-support-tag">
  <span class="lst-supported">Soportado en ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de Stripe](https://docs.stripe.com/mcp) conecta tu agente ADK al
ecosistema de [Stripe](https://stripe.com/). Esta integración le otorga a tu agente
la capacidad de gestionar pagos, clientes, suscripciones y facturas usando
lenguaje natural, habilitando flujos de trabajo de comercio automatizados y
operaciones financieras.

## Casos de uso

- **Automatizar Operaciones de Pago**: Crea enlaces de pago, procesa reembolsos y
  lista intenciones de pago mediante comandos conversacionales.

- **Simplificar Facturación**: Genera y finaliza facturas, añade elementos de línea y
  rastrea pagos pendientes sin salir de tu entorno de desarrollo.

- **Acceder a Información Empresarial**: Consulta saldos de cuenta, lista productos y
  precios, y busca a través de recursos de Stripe para tomar decisiones basadas en datos.

## Requisitos previos

- Crear una [cuenta de Stripe](https://dashboard.stripe.com/register)
- Generar una [clave API restringida](https://dashboard.stripe.com/apikeys) desde el
  Panel de Stripe

## Usar con agente

=== "Python"

    === "Servidor MCP Local"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        STRIPE_SECRET_KEY = "YOUR_STRIPE_SECRET_KEY"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="stripe_agent",
            instruction="Help users manage their Stripe account",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="npx",
                            args=[
                                "-y",
                                "@stripe/mcp",
                                "--tools=all",
                                # (Opcional) Especificar qué herramientas habilitar
                                # "--tools=customers.read,invoices.read,products.read",
                            ],
                            env={
                                "STRIPE_SECRET_KEY": STRIPE_SECRET_KEY,
                            }
                        ),
                        timeout=30,
                    ),
                )
            ],
        )
        ```

    === "Servidor MCP Remoto"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StreamableHTTPServerParams

        STRIPE_SECRET_KEY = "YOUR_STRIPE_SECRET_KEY"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="stripe_agent",
            instruction="Help users manage their Stripe account",
            tools=[
                McpToolset(
                    connection_params=StreamableHTTPServerParams(
                        url="https://mcp.stripe.com",
                        headers={
                            "Authorization": f"Bearer {STRIPE_SECRET_KEY}",
                        },
                    ),
                )
            ],
        )
        ```

=== "TypeScript"

    === "Servidor MCP Local"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const STRIPE_SECRET_KEY = "YOUR_STRIPE_SECRET_KEY";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "stripe_agent",
            instruction: "Help users manage their Stripe account",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "npx",
                        args: [
                            "-y",
                            "@stripe/mcp",
                            "--tools=all",
                            // (Opcional) Especificar qué herramientas habilitar
                            // "--tools=customers.read,invoices.read,products.read",
                        ],
                        env: {
                            STRIPE_SECRET_KEY: STRIPE_SECRET_KEY,
                        },
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

    === "Servidor MCP Remoto"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const STRIPE_SECRET_KEY = "YOUR_STRIPE_SECRET_KEY";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "stripe_agent",
            instruction: "Help users manage their Stripe account",
            tools: [
                new MCPToolset({
                    type: "StreamableHTTPConnectionParams",
                    url: "https://mcp.stripe.com",
                    header: {
                        Authorization: `Bearer ${STRIPE_SECRET_KEY}`,
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

!!! tip "Mejores prácticas"

    Habilita la confirmación humana de las acciones de herramientas y ejerce precaución al usar
    el servidor MCP de Stripe junto con otros servidores MCP para mitigar
    riesgos de inyección de prompts.

## Herramientas disponibles

Recurso | Herramienta | API
-------- | ---- | ----
Cuenta | `get_stripe_account_info` | Recuperar cuenta
Saldo | `retrieve_balance` | Recuperar saldo
Cupón | `create_coupon` | Crear cupón
Cupón | `list_coupons` | Listar cupones
Cliente | `create_customer` | Crear cliente
Cliente | `list_customers` | Listar clientes
Disputa | `list_disputes` | Listar disputas
Disputa | `update_dispute` | Actualizar disputa
Factura | `create_invoice` | Crear factura
Factura | `create_invoice_item` | Crear elemento de factura
Factura | `finalize_invoice` | Finalizar factura
Factura | `list_invoices` | Listar facturas
Enlace de Pago | `create_payment_link` | Crear enlace de pago
IntenciónDePago | `list_payment_intents` | Listar IntencionesDePago
Precio | `create_price` | Crear precio
Precio | `list_prices` | Listar precios
Producto | `create_product` | Crear producto
Producto | `list_products` | Listar productos
Reembolso | `create_refund` | Crear reembolso
Suscripción | `cancel_subscription` | Cancelar suscripción
Suscripción | `list_subscriptions` | Listar suscripciones
Suscripción | `update_subscription` | Actualizar suscripción
Otros | `search_stripe_resources` | Buscar recursos de Stripe
Otros | `fetch_stripe_resources` | Obtener objeto de Stripe
Otros | `search_stripe_documentation` | Buscar conocimiento de Stripe

## Recursos adicionales

- [Documentación del Servidor MCP de Stripe](https://docs.stripe.com/mcp)
- [Servidor MCP de Stripe en GitHub](https://github.com/stripe/ai/tree/main/tools/modelcontextprotocol)
- [Construir en Stripe con LLMs](https://docs.stripe.com/building-with-llms)
- [Añadir Stripe a tus flujos de trabajo agénticos](https://docs.stripe.com/agents)