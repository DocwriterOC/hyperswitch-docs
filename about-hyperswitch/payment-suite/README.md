---
description: Open, Modular, Self-Hostable Payment Infrastructure
icon: container-storage
---

# Juspay hyperswitch Payment Suite

## TL;DR

Juspay hyperswitch is an open-source payment orchestration platform that reduces payment failures and processing costs by intelligently routing transactions across multiple payment processors. Engineering teams use it to increase authorisation rates by 5-15%, reduce processor downtime impact, and maintain full control over their payment infrastructure. The platform consists of four modular components (SDK, Orchestration, Connectors, Vault) that can be owned by Hyperswitch, self-hosted, or provided by third parties. Choose **Client-Side SDK** integration for frontend-driven flows with minimal backend work, or **Server-to-Server (S2S)** integration for backend-controlled execution and pre-payment tokenisation. This guide covers architecture, integration models, PCI scope implications, and code examples to help you implement the right solution for your compliance posture and engineering capabilities.

---

## Prerequisites

Before integrating with Juspay hyperswitch, ensure you have:

| Requirement | Description |
|-------------|-------------|
| **API Key** | Obtain your API key from the [Hyperswitch Control Centre](https://app.hyperswitch.io) |
| **Base URL** | Production: `https://api.hyperswitch.io/v1` or Sandbox: `https://sandbox.hyperswitch.io/v1` |
| **Authentication** | Include your API key in the `api-key` header for all requests |
| **Webhook Endpoint** | Configure a HTTPS endpoint to receive asynchronous payment status updates |
| **HTTPS** | Your checkout pages must be served over HTTPS for secure data transmission |

### Quick Authentication Test

```bash
curl -X GET https://sandbox.hyperswitch.io/v1/health \
  -H "api-key: YOUR_API_KEY"
```

---

## Quick Start: Your First Payment

Get a test payment working in under 5 minutes:

### Step 1: Get Your API Key
1. Sign up at [Hyperswitch Control Centre](https://app.hyperswitch.io)
2. Navigate to **Developers → API Keys**
3. Copy your sandbox API key (starts with `snd_`)

### Step 2: Create a Payment
```bash
curl -X POST https://sandbox.hyperswitch.io/v1/payments \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_SANDBOX_API_KEY" \
  -d '{
    "amount": 10000,
    "currency": "USD",
    "customer_id": "test_customer_001",
    "description": "Test payment"
  }'
```

### Step 3: Complete the Payment
Use the `client_secret` from the response with our test checkout:
```bash
# The payment is now in "requires_confirmation" state
# Use the client_secret to complete it via SDK or API
```

### Test Card Numbers
Use these test cards for sandbox transactions:

| Card Number | Brand | Scenario |
|-------------|-------|----------|
| `4111111111111111` | Visa | Successful payment |
| `4000000000000002` | Visa | Declined (generic) |
| `4000000000000127` | Visa | Incorrect CVC |
| `4000000000000069` | Visa | Expired card |
| `5555555555554444` | Mastercard | Successful payment |

See the [full test card reference](https://docs.hyperswitch.io/explore-hyperswitch/test-cards) for more scenarios.

---

## The Four Core Components

Juspay hyperswitch is built for teams that want engineering-grade control over payments. The ecosystem can be viewed as four independent building blocks. By defining ownership of each block — Hyperswitch-managed, self-hosted, or third-party — you can design an architecture aligned with your compliance posture, performance requirements, and internal engineering capabilities.

### Component 1: The SDK (Frontend)

The Software Development Kit (SDK) is the entry point for your payment flow. It resides in your frontend and is responsible for securely capturing sensitive payment information.

**Ownership Options:**

| Ownership Model | Best For |
|-----------------|----------|
| **Hyperswitch-managed** | Rapid implementation, minimal maintenance overhead |
| **Self-hosted** | Full customisation of UI/UX, specific compliance requirements |
| **Third-party** | Existing frontend investments, hybrid architectures |

### Component 2: Intelligent Routing and Orchestration (Backend)

The core of the operation manages the payment lifecycle, executes routing logic, and handles post-payment operations like refunds and voids.

**What Makes Routing "Intelligent":**

Juspay hyperswitch's routing engine evaluates multiple real-time criteria to select the optimal Payment Service Provider (PSP) for each transaction:

- **Cost optimisation** — Routes to the lowest-cost processor based on fee structures
- **Success rate scoring** — Directs traffic to PSPs with historically higher authorisation rates for specific card types or regions
- **Latency-based selection** — Chooses connectors with the fastest response times
- **Custom business rules** — Implements merchant-defined logic (region-based routing, volume caps, failover thresholds)
- **Dynamic retries** — Automatically retries failed transactions through alternate PSPs with smart backoff strategies

**Ownership Options:**

| Ownership Model | Best For |
|-----------------|----------|
| **Hyperswitch Cloud** | Managed infrastructure, automatic scaling, minimal DevOps |
| **Self-hosted** | Data residency requirements, custom compliance frameworks |

### Component 3: Acquirer and Processor Connectivity (Connectors)

The actual pipelines that translate transactions between your system and payment processors (e.g., Stripe, Adyen, Worldpay).

**Ownership Options:**

| Ownership Model | Best For |
|-----------------|----------|
| **Hyperswitch-managed connectors** | Pre-built integrations, automatic updates |
| **Custom connectors** | Proprietary PSP relationships, specialised regional processors |

See the [full list of supported payment processors](https://docs.hyperswitch.io/explore-hyperswitch/payment-flows-and-management/payment-methods) in our documentation.

### Component 4: Vault (Card Data Storage)

The secure locker for sensitive card data to enable "One-Click" recurring payments without the user re-entering details.

**Ownership Options:**

| Ownership Model | Best For |
|-----------------|----------|
| **Hyperswitch Vault** | PCI DSS scope reduction, managed security, tokenisation included |
| **External Vault** | Existing vault investments, specific compliance requirements |
| **Hybrid** | Gradual migration from legacy vault infrastructure |

Learn how to [connect external vaults to Juspay hyperswitch orchestration](https://docs.hyperswitch.io/explore-hyperswitch/workflows/vault/connect-external-vaults-to-hyperswitch-orchestration).

---

## Integration Architecture

With the components defined, the next step is to select your integration architecture. This choice hinges on a single question: **Who controls the payment execution?**

Choose the integration method that best aligns with your payment flow requirements.

---

## Integration Model 1: Client-Side SDK Payments

**Tokenise Post-Payment | SDK-Initiated Execution**

Use this model when you want dynamic, frontend-driven payment experiences with minimal backend orchestration logic.

### When to Choose This Model

| Choose Model 1 If | Reason |
|-------------------|--------|
| You want dynamic, frontend-driven payment experiences | SDK handles UI state and user interactions natively |
| You prefer minimal backend orchestration logic | SDK triggers payment confirmation directly |
| You want SDK-triggered payment confirmation | Reduces server-side complexity |
| You are optimising for rapid checkout implementation | Fastest time-to-market for standard flows |

### High-Level Flow

1. Merchant calls the [Create Payment API](https://api-reference.hyperswitch.io/v1/payments/payments--create) and loads the [Payment SDK](https://docs.hyperswitch.io/explore-hyperswitch/payment-experience/payment)
2. SDK securely collects payment details
3. SDK triggers payment confirmation
4. SDK communicates with Hyperswitch backend
5. Juspay hyperswitch:
   - Applies [intelligent routing logic](https://docs.hyperswitch.io/explore-hyperswitch/workflows/intelligent-routing)
   - Sends request to configured PSP
   - Manages authorisation and capture
   - Returns final payment status

### Code Example: Client-Side SDK Integration

#### Step 1: Create a Payment Session (Backend)

```bash
curl -X POST https://sandbox.hyperswitch.io/v1/payments \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{
    "amount": 10000,
    "currency": "USD",
    "customer_id": "cust_12345",
    "description": "Premium subscription",
    "metadata": {
      "order_id": "order_789"
    }
  }'
```

**Response:**

```json
{
  "payment_id": "pay_abcdefghijklmnopqrstuvwxyz",
  "client_secret": "pay_abcdefghijklmnopqrstuvwxyz_secret_1234567890",
  "status": "requires_confirmation",
  "amount": 10000,
  "currency": "USD"
}
```

#### Step 2: Initialise the SDK (Frontend)

```html
<script src="https://checkout.hyperswitch.io/v1/checkout.js"></script>

<div id="payment-form"></div>

<script>
  const hyper = window.Hyper('YOUR_PUBLISHABLE_KEY');
  
  const widgets = hyper.widgets({
    appearance: {
      theme: 'default'
    }
  });
  
  const payment = widgets.create('payment', {
    clientSecret: 'pay_abcdefghijklmnopqrstuvwxyz_secret_1234567890'
  });
  
  payment.mount('#payment-form');
</script>
```

#### Step 3: Confirm Payment (SDK-Triggered)

The SDK automatically handles confirmation when the customer submits the form. Your backend receives the final status via webhook.

---

## Integration Model 2: Server-to-Server (S2S) Payments

**Tokenise Pre-Payment | Backend-Controlled Execution**

Use this model when you want granular control over transaction timing and require backend-driven orchestration logic.

### When to Choose This Model

| Choose Model 2 If | Reason |
|-------------------|--------|
| You want granular control over transaction timing | Execute payments at precise business moments |
| You require backend-driven orchestration logic | Complex business rules, multi-step flows |
| You want to tokenise credentials before execution | Decouple vaulting from transaction processing |
| You prefer decoupling vaulting from transaction processing | Reuse stored payment methods across transactions |

### High-Level Flow

#### Step 1: Tokenise Card

Tokenise payment credentials using the [Vault SDK](https://docs.hyperswitch.io/explore-hyperswitch/payment-experience/payment-method/web) or a backend call to the [Create Payment Method API](https://api-reference.hyperswitch.io/v2/payment-methods/payment-method--create-v1).

Juspay hyperswitch securely stores the credential and returns a reusable identifier: `payment_method_id`.

#### Step 2: Trigger Payment Execution

**Option A: Process via Hyperswitch Orchestration**

Use this option if you want Juspay hyperswitch to:
- Apply intelligent routing logic
- Select the optimal connector
- Manage smart retries and failover
- Handle authorisation and capture lifecycle

This is the recommended model for merchants adopting Juspay hyperswitch orchestration.

**Option B: Process via Proxy API**

Use this option if:
- You do not want to change your existing PSP integration immediately
- You want Juspay hyperswitch to act as a passthrough layer
- You are incrementally migrating to full orchestration

In this mode:
- Your existing integration contract remains unchanged
- Juspay hyperswitch forwards requests to the configured processor
- You can progressively enable routing and orchestration features

### Code Example: Server-to-Server Integration

#### Step 1: Tokenise Payment Method (Backend)

```bash
curl -X POST https://sandbox.hyperswitch.io/v1/payment_methods \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{
    "type": "card",
    "card": {
      "number": "4111111111111111",
      "exp_month": "12",
      "exp_year": "2027",
      "cvc": "123"
    },
    "customer_id": "cust_12345"
  }'
```

**Response:**

```json
{
  "payment_method_id": "pm_abcdefghijklmnopqrstuvwxyz",
  "customer_id": "cust_12345",
  "payment_method": "card",
  "card": {
    "scheme": "Visa",
    "last4_digits": "1111",
    "exp_month": "12",
    "exp_year": "2027"
  },
  "created": "2026-03-10T12:00:00Z"
}
```

#### Step 2: Create Payment Using Token (Backend)

```bash
curl -X POST https://sandbox.hyperswitch.io/v1/payments \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{
    "amount": 10000,
    "currency": "USD",
    "customer_id": "cust_12345",
    "payment_method": "card",
    "payment_method_id": "pm_abcdefghijklmnopqrstuvwxyz",
    "payment_method_type": "credit",
    "confirm": true,
    "description": "Premium subscription"
  }'
```

**Response:**

```json
{
  "payment_id": "pay_zyxwvutsrqponmlkjihgfedcba",
  "status": "succeeded",
  "amount": 10000,
  "currency": "USD",
  "connector": "stripe",
  "payment_method_id": "pm_abcdefghijklmnopqrstuvwxyz"
}
```

#### Alternative: Proxy API (Incremental Migration)

```bash
curl -X POST https://sandbox.hyperswitch.io/v1/proxy/payments \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{
    "connector": "stripe",
    "payload": {
      "amount": 10000,
      "currency": "usd",
      "source": "tok_visa"
    }
  }'
```

---

### Error Handling

Handle these common error scenarios in your integration:

#### Payment Declined
```json
{
  "payment_id": "pay_abcdefghijklmnopqrstuvwxyz",
  "status": "failed",
  "error_code": "card_declined",
  "error_message": "Your card was declined.",
  "decline_code": "insufficient_funds"
}
```

**Action:** Prompt customer to use a different payment method or contact their bank.

#### Authentication Required (3D Secure)
```json
{
  "payment_id": "pay_abcdefghijklmnopqrstuvwxyz",
  "status": "requires_customer_action",
  "next_action": {
    "type": "three_ds_authentication",
    "url": "https://acs.bank.com/authenticate"
  }
}
```

**Action:** Redirect customer to the authentication URL and handle the callback.

#### Network/Timeout Errors
```json
{
  "error": {
    "type": "api_connection_error",
    "message": "Network error communicating with upstream processor"
  }
}
```

**Action:** Retry with exponential backoff. Use idempotency keys to prevent duplicate charges.

---

### Webhook Handling

Configure your endpoint to receive asynchronous payment status updates.

#### Webhook Payload Example
```json
{
  "event_type": "payment.succeeded",
  "event_id": "evt_abcdefghijklmnopqrstuvwxyz",
  "created": "2026-03-10T12:30:00Z",
  "data": {
    "payment_id": "pay_abcdefghijklmnopqrstuvwxyz",
    "status": "succeeded",
    "amount": 10000,
    "currency": "USD",
    "connector": "stripe",
    "customer_id": "cust_12345"
  }
}
```

#### Signature Verification
Verify webhook authenticity using the signature header:

```python
import hmac
import hashlib

# Your webhook signing secret from Control Centre
webhook_secret = "whsec_..."

# From the webhook request headers
signature = request.headers.get("x-webhook-signature")
payload = request.body

expected_signature = hmac.new(
    webhook_secret.encode(),
    payload.encode(),
    hashlib.sha256
).hexdigest()

if not hmac.compare_digest(signature, expected_signature):
    raise ValueError("Invalid webhook signature")
```

#### Idempotency
Store processed `event_id` values to prevent duplicate processing:

```python
# Check if event already processed
if event_id in processed_events:
    return  # Skip duplicate

# Process the event
process_payment_update(data)

# Mark as processed
processed_events.add(event_id)
```

---

## Decision Framework: Model 1 vs Model 2

Use this framework to select the right integration model for your use case.

| Decision Factor | Choose Model 1 (Client-Side SDK) | Choose Model 2 (Server-to-Server) |
|-----------------|----------------------------------|-----------------------------------|
| **Development Speed** | Faster — minimal backend work required | Slower — requires backend integration |
| **Control Level** | Standard — SDK handles execution | Granular — you control timing and logic |
| **PCI Scope** | Reduced — sensitive data never touches your servers | Higher — you handle card data (unless using Vault SDK) |
| **Recurring Payments** | Supported via saved payment methods | Ideal — pre-tokenisation enables one-click checkout |
| **Complex Business Logic** | Limited — frontend-driven | Full support — backend orchestration |
| **Retry Handling** | Managed by Hyperswitch | Configurable — custom retry strategies |
| **Migration Path** | Clean slate implementation | Proxy API enables incremental adoption |

### Recommended Default

For most new implementations, **Model 1 (Client-Side SDK)** provides the fastest time-to-value with reduced compliance burden. Choose **Model 2** when you have specific requirements for backend control, complex orchestration, or gradual migration from existing PSP integrations.

---

## PCI DSS Scope Discussion

Understanding your PCI DSS (Payment Card Industry Data Security Standard) scope is critical when designing your payment architecture.

### PCI DSS Scope by Component Ownership

| Component | Hyperswitch-Managed | Self-Hosted | Third-Party |
|-----------|---------------------|-------------|-------------|
| **SDK** | SAQ A eligible | SAQ A-EP or SAQ D | Depends on provider |
| **Orchestration** | No card data exposure | Requires assessment | Depends on integration |
| **Connectors** | PCI compliance managed by PSPs | PCI compliance managed by PSPs | Verify provider compliance |
| **Vault** | SAQ A eligible | SAQ D required | Verify provider attestation |

### Integration Model Impact on PCI Scope

#### Model 1: Client-Side SDK (Reduced Scope)

When using the Hyperswitch-managed SDK:
- Sensitive card data is captured directly by the SDK and transmitted to Hyperswitch
- Your servers never handle raw card numbers
- You may be eligible for **SAQ A** (Self-Assessment Questionnaire A)
- Requires: HTTPS, no server-side card data handling

#### Model 2: Server-to-Server (Higher Scope)

When handling card data on your backend:
- Your servers process raw card numbers before tokenisation
- You are responsible for securing card data in transit and at rest
- Typically requires **SAQ D** or full PCI DSS assessment
- Mitigation: Use the Vault SDK to tokenise client-side, then process payments server-side using tokens

### Reducing Your PCI Scope

1. **Use Hyperswitch Vault** — Store all card data in our PCI DSS Level 1 compliant vault
2. **Tokenise early** — Convert card data to tokens at the earliest opportunity
3. **Avoid logging** — Never log full card numbers or CVV codes
4. **Use HTTPS everywhere** — Ensure all payment-related traffic is encrypted

For detailed PCI guidance, consult the [PCI Security Standards Council documentation](https://www.pcisecuritystandards.org/) or speak with your compliance advisor.

---

## Competitive Differentiation

Juspay hyperswitch stands apart from other payment orchestration solutions through these key differentiators.

### Open Source

Unlike proprietary platforms, Juspay hyperswitch is fully open-source. You benefit from:
- **No vendor lock-in** — Full access to source code, deploy anywhere
- **Community contributions** — Continuous improvements from a global developer community
- **Transparency** — Audit the code that handles your payments
- **Customisation** — Modify and extend for your specific needs

### Unified API Across 100+ Processors

Write once, route anywhere. Our single API abstracts differences across:
- Major global processors (Stripe, Adyen, Worldpay)
- Regional specialists (local payment methods, country-specific providers)
- Alternative payment methods (wallets, bank transfers, buy-now-pay-later)

### Intelligent Routing with Business Impact

Our routing engine goes beyond simple failover:
- **Revenue optimisation** — Route to processors with higher approval rates
- **Cost reduction** — Automatically select lowest-cost routing paths
- **Latency improvement** — Reduce checkout abandonment with faster processing
- **Custom strategies** — Implement your own routing logic via our rule engine

### Modular Architecture

Deploy only what you need:
- Use our SDK with your existing backend
- Use our orchestration with your existing vault
- Self-host components requiring data residency
- Mix and match to meet your compliance requirements

### Cost Efficiency

- **Transparent pricing** — No hidden fees or volume commitments
- **Reduced processor costs** — Competition between connected PSPs drives better rates
- **Lower engineering overhead** — Single integration instead of multiple PSP implementations

---

## When NOT to Use Juspay hyperswitch

Juspay hyperswitch is not the right fit for every organisation. Consider alternatives if:

| Scenario | Reason |
|----------|--------|
| **You process very low transaction volumes** (<100/month) | The integration overhead may exceed the benefits of orchestration |
| **You require a single PSP relationship** | If you have no plans to add redundancy or optimise routing, a direct integration may be simpler |
| **You need immediate, custom support SLAs** | While we offer support, enterprise SLA requirements may be better served by proprietary vendors |
| **Your compliance framework requires specific certifications** | Verify that our current certifications meet your regulatory requirements |
| **You require real-time, cross-processor reconciliation** | Our reconciliation features may not meet specialised accounting requirements |

### Alternatives to Consider

- **Direct PSP integration** — Stripe, Adyen, or Braintree for simple, single-provider setups
- **Enterprise proprietary platforms** — Spreedly or Payoneer for managed service with guaranteed SLAs
- **In-house build** — For organisations with specialised requirements and significant engineering resources

---

## Next Steps / Implementation Checklist

Use this checklist to guide your Juspay hyperswitch implementation.

### Phase 1: Preparation

- [ ] Obtain API credentials from the [Hyperswitch Control Centre](https://app.hyperswitch.io)
- [ ] Configure your webhook endpoint for asynchronous status updates
- [ ] Identify your compliance requirements and desired PCI scope
- [ ] Select your component ownership model (Hyperswitch-managed, self-hosted, or hybrid)

### Phase 2: Architecture Decision

- [ ] Review the Decision Framework (Model 1 vs Model 2)
- [ ] Select your integration model based on control requirements and PCI scope
- [ ] Identify required payment processors and verify connector availability
- [ ] Define your routing strategy (cost-optimised, success-rate-optimised, or custom rules)

### Phase 3: Integration

- [ ] Implement the Create Payment API (Model 1) or Payment Method Tokenisation (Model 2)
- [ ] Integrate the SDK into your frontend (if using Model 1)
- [ ] Build backend payment confirmation logic (if using Model 2)
- [ ] Configure webhook handlers for payment status updates
- [ ] Implement error handling and retry logic

### Phase 4: Testing

- [ ] Test successful payment flows using test card numbers
- [ ] Test failure scenarios (insufficient funds, expired cards, network errors)
- [ ] Verify webhook delivery and handling
- [ ] Test retry and failover behaviour (if using multiple connectors)
- [ ] Validate PCI scope with your compliance team

### Phase 5: Go-Live

- [ ] Switch from sandbox to production API keys
- [ ] Configure production webhook endpoints
- [ ] Enable monitoring and alerting for payment flows
- [ ] Set up reconciliation processes
- [ ] Document your implementation for your operations team

### Getting Help

- [API Reference](https://api-reference.hyperswitch.io/) — Complete endpoint documentation
- [Documentation](https://docs.hyperswitch.io/) — Guides, tutorials, and best practices
- [GitHub](https://github.com/juspay/hyperswitch) — Source code and issue tracking
- [Community Discord](https://discord.gg/hyperswitch) — Ask questions and connect with other developers

---

## Key Terms and Acronyms

| Term | Definition |
|------|------------|
| **SDK** | Software Development Kit — A set of tools and libraries for building applications. In this context, the frontend components that capture payment information. |
| **S2S** | Server-to-Server — Integration pattern where your backend communicates directly with the payment provider's API. |
| **PSP** | Payment Service Provider — A company that enables merchants to accept electronic payments (also called processor, acquirer, or gateway). |
| **PCI DSS** | Payment Card Industry Data Security Standard — Security standards for organisations that handle credit card information. |
| **SAQ** | Self-Assessment Questionnaire — A validation tool for merchants and service providers to evaluate their PCI DSS compliance. |
| **Tokenisation** | The process of replacing sensitive card data with a non-sensitive equivalent (token) that can be used for subsequent transactions. |
| **Orchestration** | The coordination and management of payment flows across multiple processors and services. |
