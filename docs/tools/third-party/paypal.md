---
catalog_title: Paypal
catalog_description: Manage payments, send invoices, and handle subscriptions
catalog_icon: /assets/tools-paypal.png
---

# PayPal

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python v0.1.0</span><span class="lst-typescript">TypeScript v0.2.0</span>
</div>

El [Servidor MCP de PayPal](https://github.com/paypal/paypal-mcp-server) conecta
tu agente ADK al ecosistema de [PayPal](https://www.paypal.com/). Esta
integración le da a tu agente la capacidad de gestionar pagos, facturas,
suscripciones y disputas usando lenguaje natural, habilitando flujos de trabajo
de comercio automatizado e información empresarial.

## Casos de uso

- **Simplifica Operaciones Financieras**: Crea órdenes, envía facturas y procesa
  reembolsos directamente a través del chat sin cambiar de contexto. Puedes instruir a tu
  agente para "facturar al Cliente X" o "reembolsar la orden Y" inmediatamente.

- **Gestiona Suscripciones y Productos**: Maneja el ciclo de vida completo de
  facturación recurrente creando productos, configurando planes de suscripción y gestionando
  detalles de suscriptores usando lenguaje natural.

- **Resuelve Problemas y Rastrea el Rendimiento**: Resume y acepta reclamos de disputas,
  rastrea estados de envío y obtén información de comerciantes para tomar decisiones
  basadas en datos sobre la marcha.

## Prerrequisitos

- Crea una [cuenta de desarrollador de PayPal](https://developer.paypal.com/)
- Crea una aplicación y obtén tus credenciales desde el
  [Panel de Desarrollador de PayPal](https://developer.paypal.com/)
- [Genera un token de acceso](https://developer.paypal.com/reference/get-an-access-token/)
  desde tus credenciales

## Uso con agente

=== "Python"

    === "Local MCP Server"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
        from mcp import StdioServerParameters

        PAYPAL_ENVIRONMENT = "SANDBOX"  # Opciones: "SANDBOX" o "PRODUCTION"
        PAYPAL_ACCESS_TOKEN = "YOUR_PAYPAL_ACCESS_TOKEN"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="paypal_agent",
            instruction="Help users manage their PayPal account",
            tools=[
                McpToolset(
                    connection_params=StdioConnectionParams(
                        server_params=StdioServerParameters(
                            command="npx",
                            args=[
                                "-y",
                                "@paypal/mcp",
                                "--tools=all",
                                # (Opcional) Especifica qué herramientas habilitar
                                # "--tools=subscriptionPlans.list,subscriptionPlans.show",
                            ],
                            env={
                                "PAYPAL_ACCESS_TOKEN": PAYPAL_ACCESS_TOKEN,
                                "PAYPAL_ENVIRONMENT": PAYPAL_ENVIRONMENT,
                            }
                        ),
                        timeout=300,
                    ),
                )
            ],
        )
        ```

    === "Remote MCP Server"

        ```python
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import SseConnectionParams

        PAYPAL_MCP_ENDPOINT = "https://mcp.sandbox.paypal.com/sse"  # Producción: https://mcp.paypal.com/sse
        PAYPAL_ACCESS_TOKEN = "YOUR_PAYPAL_ACCESS_TOKEN"

        root_agent = Agent(
            model="gemini-2.5-pro",
            name="paypal_agent",
            instruction="Help users manage their PayPal account",
            tools=[
                McpToolset(
                    connection_params=SseConnectionParams(
                        url=PAYPAL_MCP_ENDPOINT,
                        headers={
                            "Authorization": f"Bearer {PAYPAL_ACCESS_TOKEN}",
                        },
                    ),
                )
            ],
        )
        ```

=== "TypeScript"

    === "Local MCP Server"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const PAYPAL_ENVIRONMENT = "SANDBOX"; // Opciones: "SANDBOX" o "PRODUCTION"
        const PAYPAL_ACCESS_TOKEN = "YOUR_PAYPAL_ACCESS_TOKEN";

        const rootAgent = new LlmAgent({
            model: "gemini-2.5-pro",
            name: "paypal_agent",
            instruction: "Help users manage their PayPal account",
            tools: [
                new MCPToolset({
                    type: "StdioConnectionParams",
                    serverParams: {
                        command: "npx",
                        args: [
                            "-y",
                            "@paypal/mcp",
                            "--tools=all",
                            // (Opcional) Especifica qué herramientas habilitar
                            // "--tools=subscriptionPlans.list,subscriptionPlans.show",
                        ],
                        env: {
                            PAYPAL_ACCESS_TOKEN: PAYPAL_ACCESS_TOKEN,
                            PAYPAL_ENVIRONMENT: PAYPAL_ENVIRONMENT,
                        },
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

!!! note

    **Expiración del Token**: Los tokens de acceso de PayPal tienen una vida útil limitada de 3-8
    horas. Si tu agente deja de funcionar, asegúrate de que tu token no haya expirado y
    genera uno nuevo si es necesario. Debes implementar lógica de actualización de tokens para
    manejar la expiración del token.

## Herramientas disponibles

### Gestión de catálogo

Herramienta | Descripción
---- | -----------
`create_product` | Crea un nuevo producto en el catálogo de PayPal
`list_products` | Lista productos del catálogo de PayPal
`show_product_details` | Muestra detalles de un producto específico del catálogo de PayPal
`update_product` | Actualiza un producto existente en el catálogo de PayPal

### Gestión de disputas

Herramienta | Descripción
---- | -----------
`list_disputes` | Obtiene un resumen de todas las disputas con filtrado opcional
`get_dispute` | Obtiene información detallada sobre una disputa específica
`accept_dispute_claim` | Acepta un reclamo de disputa, resolviéndolo a favor del comprador

### Facturas

Herramienta | Descripción
---- | -----------
`create_invoice` | Crea una nueva factura en el sistema de PayPal
`list_invoices` | Lista facturas
`get_invoice` | Obtiene detalles sobre una factura específica
`send_invoice` | Envía una factura existente al destinatario especificado
`send_invoice_reminder` | Envía un recordatorio para una factura existente
`cancel_sent_invoice` | Cancela una factura enviada
`generate_invoice_qr_code` | Genera un código QR para una factura

### Pagos

Herramienta | Descripción
---- | -----------
`create_order` | Crea una orden en el sistema de PayPal basada en los detalles proporcionados
`create_refund` | Procesa un reembolso para un pago capturado
`get_order` | Obtiene detalles de un pago específico
`get_refund` | Obtiene los detalles de un reembolso específico
`pay_order` | Captura el pago para una orden autorizada

### Reportes e información

Herramienta | Descripción
---- | -----------
`get_merchant_insights` | Obtiene métricas de inteligencia empresarial y analítica para un comerciante
`list_transactions` | Lista todas las transacciones

### Rastreo de envío

Herramienta | Descripción
---- | -----------
`create_shipment_tracking` | Crea información de rastreo de envío para una transacción de PayPal
`get_shipment_tracking` | Obtiene información de rastreo de envío para un envío específico
`update_shipment_tracking` | Actualiza información de rastreo de envío para un envío específico

### Gestión de suscripciones

Herramienta | Descripción
---- | -----------
`cancel_subscription` | Cancela una suscripción activa
`create_subscription` | Crea una nueva suscripción
`create_subscription_plan` | Crea un nuevo plan de suscripción
`update_subscription` | Actualiza una suscripción existente
`list_subscription_plans` | Lista planes de suscripción
`show_subscription_details` | Muestra detalles de una suscripción específica
`show_subscription_plan_details` | Muestra detalles de un plan de suscripción específico

## Configuración

Puedes controlar qué herramientas están habilitadas usando el argumento de línea de comandos
`--tools`. Esto es útil para limitar el alcance de los permisos del agente.

Puedes habilitar todas las herramientas con `--tools=all` o especificar una lista separada por comas de
identificadores de herramientas específicas.

**Nota**: Los identificadores de configuración a continuación usan notación de puntos (p. ej.,
`invoices.create`) que difiere de los nombres de herramientas expuestos al agente (p. ej.,
`create_invoice`).

**Productos**: `products.create`, `products.list`, `products.update`,
`products.show`

**Disputas**:
`disputes.list`, `disputes.get`, `disputes.create`

**Facturas**: `invoices.create`, `invoices.list`, `invoices.get`,
`invoices.send`, `invoices.sendReminder`, `invoices.cancel`,
`invoices.generateQRC`

**Órdenes y Pagos**: `orders.create`, `orders.get`, `orders.capture`,
`payments.createRefund`, `payments.getRefunds`

**Transacciones**:
`transactions.list`

**Envío**:
`shipment.create`, `shipment.get`

**Suscripciones**: `subscriptionPlans.create`, `subscriptionPlans.list`,
`subscriptionPlans.show`, `subscriptions.create`, `subscriptions.show`,
`subscriptions.cancel`

## Recursos adicionales

- [Documentación del Servidor MCP de PayPal](https://docs.paypal.ai/developer/tools/ai/mcp-quickstart)
- [Repositorio del Servidor MCP de PayPal](https://github.com/paypal/paypal-mcp-server)
- [Referencia de Herramientas de Agente de PayPal](https://docs.paypal.ai/developer/tools/ai/agent-tools-ref)