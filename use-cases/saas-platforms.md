---
description: All the payment use-cases for SaaS providers
icon: desktop
---

# SaaS Platforms

**TL;DR:** Juspay Hyperswitch provides SaaS platforms with a multi-tenant payment infrastructure that supports merchant-specific processors, hierarchical isolation, and unified operations.

---

## What payment challenges do SaaS platforms face?

SaaS platforms in commerce, bookings, and professional services must manage payments for thousands of distinct merchants. The core challenge is balancing standardisation with flexibility.

| Challenge | Description |
|-----------|-------------|
| Processor lock-in | Merchants have pre-negotiated rates with specific providers |
| Tenant isolation | Each merchant's data and routing rules must stay separate |
| Onboarding scale | Manual provisioning creates operational bottlenecks |
| Fragmented flows | Different PSPs handle 3DS, refunds, and errors differently |
| Vendor lock-in | PSP-specific vaults trap saved customer cards |

---

## What is the Connector Abstraction Layer?

The Connector Abstraction Layer (also called BYOP — Bring Your Own Processor) decouples your platform from specific payment providers. You integrate once, and transactions route to each merchant's preferred processor based on configuration.

### How does it work?

| Feature | Description |
|---------|-------------|
| Unified API | Normalises 300+ processor APIs into a single [Payment Intent Flow](https://api-reference.hyperswitch.io/v1/payments/payments--create#payments-create) |
| Zero-Code Integration | Add new processors via configuration, not code. See [Supported Connectors](https://juspay.io/integrations) |
| Deployment Models | Full-Stack Mode (Juspay Hyperswitch handles UI and tokenisation) or Backend-Only (you own the UI) |

### What problems does it solve?

| Problem | Solution |
|---------|----------|
| Merchants refuse to migrate from existing providers | Support their current processor without custom integration work |
| Maintaining dozens of PSP integrations | Single integration covers all processors |
| Per-merchant processor preferences | Route dynamically based on merchant configuration |

---

## How does hierarchical tenant isolation work?

Juspay Hyperswitch provides a built-in hierarchy: **Organisation → Account → Profile**. This model ensures each merchant's routing rules, API keys, and customer data remain isolated.

Refer to [Multiple Accounts and Profiles](https://docs.hyperswitch.io/explore-hyperswitch/account-management/multiple-accounts-and-profiles) for the complete data model.

### What does each level control?

| Level | Purpose |
|-------|---------|
| Organisation | Top-level entity, typically the SaaS platform |
| Account | Individual merchant, isolated API keys and routing rules |
| Profile | Business unit or regional split (e.g., "Account A - US Store" vs "Account A - EU Store") |

### How do I manage team access?

Use [User Management](https://docs.hyperswitch.io/explore-hyperswitch/account-management/manage-your-team) controls to map dashboard users to specific levels of the hierarchy.

---

## How do I onboard merchants programmatically?

Use the [Management APIs](https://api-reference.hyperswitch.io/v1/merchant-account/merchant-account--create#merchant-account-create) to automate the full merchant lifecycle. Treat onboarding as an API call rather than a manual process.

### What APIs are available?

| API | Purpose |
|-----|---------|
| [Merchant Account Create](https://api-reference.hyperswitch.io/v1/merchant-account/merchant-account--create#merchant-account-create) | Create a new merchant entity |
| [Connector Configuration API](https://api-reference.hyperswitch.io/v1/merchant-connector-account/merchant-connector--create#merchant-connector-create) | Inject merchant's Stripe/Adyen credentials |

### What liability models are supported?

| Model | Description |
|-------|-------------|
| Merchant of Record (MoR) | Platform holds funds |
| Connected Account | Merchant holds funds |

---

## How does Juspay Hyperswitch handle 3DS and payment flows?

Juspay Hyperswitch normalises complex payment flows into a standard state machine. Your frontend handles a single response type regardless of the underlying processor's behaviour.

### What flows are unified?

| Flow | Description |
|------|-------------|
| 3D Secure (3DS) | Automatic handling across all processors. See [3DS documentation](https://docs.hyperswitch.io/explore-hyperswitch/merchant-controls/payment-features/3d-secure-3ds) |
| Auth, Capture, Void | Single API syntax across PSPs. See [Connector Payment Flows](https://docs.hyperswitch.io/learn-more/hyperswitch-architecture/connector-payment-flows) |

### Why does this matter?

Different verticals require different flows: `$0 Auth` for hotels, `3DS` for EU retail, `Recurring` for subscriptions. Without normalisation, platforms write separate handling logic for each PSP's quirks.

---

## What is the Payment Vault?

The [Payment Vault](https://docs.hyperswitch.io/about-hyperswitch/payments-modules/vault) is a neutral tokenisation service that exists independently of any processor.

### How does portable tokenisation work?

| Feature | Description |
|---------|-------------|
| Token Ownership | You or the account own the tokens, not the PSP |
| Interoperability | A card saved via Stripe can be charged via Adyen using [Network Tokenisation](https://docs.hyperswitch.io/explore-hyperswitch/payment-orchestration/quickstart/tokenization-and-saved-cards) |
| PCI-DSS Compliance | Offload compliance using certified secure storage |

### What problem does this solve?

When a merchant stores cards in a PSP-specific vault (e.g., Stripe Customer ID), switching providers means losing all saved cards. This locks in recurring revenue to a single vendor.

---

## How does error code mapping work?

Juspay Hyperswitch maps thousands of PSP error codes into a [Standardised Error Reference](https://docs.hyperswitch.io/explore-hyperswitch/payment-experience/payment/web/error-codes). Your UI displays consistent messages regardless of the upstream provider.

### Example error mapping

| PSP Error | Standardised Code |
|-----------|-------------------|
| "Do Not Honor" | `card_declined` |
| "Refusal" | `card_declined` |
| "Error 402" | `card_declined` |
| "Card Expired" | `card_expired` |

### Where can I view transaction logs?

Use the [Operations Dashboard](https://docs.hyperswitch.io/explore-hyperswitch/account-management/analytics-and-operations) to view transaction logs, refunds, and disputes across all accounts and processors in one view.

---

## How do normalised event streams work?

Juspay Hyperswitch standardises Day-2 operations (refunds, disputes, webhooks) into a unified interface. Build one refund handler and one webhook listener — it works for all connected processors.

### What is standardised?

| Event Type | Documentation |
|------------|---------------|
| Webhooks | [Standardised Webhook Schema](https://docs.hyperswitch.io/explore-hyperswitch/payment-orchestration/quickstart/webhooks) |
| Disputes | [Disputes Lifecycle](https://docs.hyperswitch.io/explore-hyperswitch/account-management/disputes) |
| Refunds/Voids | [Relay APIs](https://api-reference.hyperswitch.io/v1/relay/relay#relay-create) — trigger operations using `connector_resource_id` |

---

## How does high-availability failover work?

Juspay Hyperswitch monitors processor health and can automatically failover traffic to healthy alternatives.

### What observability features are available?

| Feature | Description |
|---------|-------------|
| Connector Health | Continuous monitoring of success rates and latency per processor |
| Smart Router | Automatic failover when a provider degrades. See [Intelligent Routing](https://docs.hyperswitch.io/explore-hyperswitch/workflows/intelligent-routing) |
| Open Telemetry | Standard [OTel Traces](https://github.com/juspay/hyperswitch/blob/main/docs/architecture.md#monitoring) for Datadog, Prometheus, or Grafana |
| System Health API | Build internal status pages using the [System Health API](https://live.hyperswitch.io/api/health) |

---

## What's next?

- [Set up multiple accounts and profiles](https://docs.hyperswitch.io/explore-hyperswitch/account-management/multiple-accounts-and-profiles)
- [Configure intelligent routing](https://docs.hyperswitch.io/explore-hyperswitch/workflows/intelligent-routing)
- [Implement webhooks](https://docs.hyperswitch.io/explore-hyperswitch/payment-orchestration/quickstart/webhooks)
- [View supported connectors](https://juspay.io/integrations)
