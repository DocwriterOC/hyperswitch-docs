---
description: Open, Modular, Self-Hostable Payment Infrastructure
cover: ../../.gitbook/assets/payments-suite-cover.png
coverY: 0
icon: container-storage
---

# Payments Suite

Hyperswitch is an **open-source payment orchestration platform** that gives you complete control over your payment stack. Whether you're a startup processing your first transaction or an enterprise handling millions in volume, Hyperswitch lets you **own your payment infrastructure** without the complexity of building it from scratch.

## Why Hyperswitch?

- **Reduce payment costs** by routing transactions to the most cost-effective processor
- **Increase authorization rates** with intelligent routing and smart retries
- **Eliminate vendor lock-in** with a unified API across 100+ payment providers
- **Stay PCI compliant** with built-in tokenization and secure vaulting
- **Deploy anywhere** - self-hosted, cloud, or hybrid

***

## Quick Start

Get your first payment running in under 15 minutes:

1. **[Create a Hyperswitch account](https://app.hyperswitch.io)** (Cloud) or **[deploy locally](https://docs.hyperswitch.io/hyperswitch-open-source/overview/unified-local-setup-using-docker)** (Self-hosted)
2. **Get your API credentials** from the Dashboard
3. **Choose your integration model** below
4. **Run your first test payment** using our sandbox environment

***

## Prerequisites

Before integrating, ensure you have:

| Requirement | Description | Quick Link |
|-------------|-------------|------------|
| Hyperswitch Account | Active merchant account with API access | [Sign up](https://app.hyperswitch.io) |
| API Key | Secret key for server-side API calls | [Dashboard → API Keys](https://app.hyperswitch.io/api-keys) |
| Publishable Key | Public key for client-side SDK | [Dashboard → Developers](https://app.hyperswitch.io/developers) |
| Profile ID | Identifier for your business profile | [Dashboard → Profiles](https://app.hyperswitch.io/profiles) |
| Test Credentials | Sandbox API endpoints and test cards | See [Testing](#testing-your-integration) section |

***

## The Four Core Components

Hyperswitch is architected as four independent building blocks. You can **mix and match** ownership models for each component based on your compliance requirements, performance needs, and engineering capabilities.

```
┌─────────────────────────────────────────────────────────────────┐
│                     HYPERSWITCH ECOSYSTEM                        │
├─────────────┬─────────────┬─────────────┬───────────────────────┤
│     SDK     │  Backend    │ Connectors  │        Vault          │
│  (Frontend) │(Orchestration│  (PSPs)    │  (Card Storage)       │
├─────────────┼─────────────┼─────────────┼───────────────────────┤
│ • Web SDK   │ • Routing   │ • Stripe    │ • Tokenization        │
│ • Mobile SDK│ • Retries   │ • Adyen     │ • Network tokens      │
│ • Headless  │ • Refunds   │ • Worldpay  │ • PCI compliance      │
│             │ • Analytics │ • 100+ more │ • External vaults     │
└─────────────┴─────────────┴─────────────┴───────────────────────┘
         ↑
    Ownership Options:
    ┌─────────────┬──────────────┬─────────────┐
    │ Hyperswitch │ Self-Hosted  │  Third-Party│
    │   Managed   │   (Your Infra)│   (External)│
    └─────────────┴──────────────┴─────────────┘
```

### Component 1: The SDK (Frontend)
The entry point for your payment flow. Securely captures sensitive payment information directly in the browser or mobile app, ensuring card data never touches your servers.

- **Web SDK**: Drop-in checkout with customizable UI
- **React SDK**: Native React components
- **Mobile SDKs**: iOS and Android native SDKs
- **Headless SDK**: Build your own UI while keeping PCI scope minimal

### Component 2: Intelligent Routing & Orchestration (Backend)
The brain of your payment operations. Manages the entire payment lifecycle, applies routing logic, and handles post-payment operations.

- **Smart Routing**: Route by cost, success rate, or custom rules
- **Auto Retries**: ML-powered retry logic for failed payments
- **Unified API**: Single API across all payment providers
- **Real-time Analytics**: Monitor performance and costs

### Component 3: Acquirer & Processor Connectivity (Connectors)
Pre-built integrations to 100+ payment service providers (PSPs) including Stripe, Adyen, Worldpay, and local payment methods worldwide.

- **100+ Connectors**: Global and local payment methods
- **Universal Tokens**: Use the same token across different PSPs
- **Instant Migration**: Switch PSPs without changing your integration

### Component 4: Vault (Card Data Storage)
PCI-compliant secure storage for sensitive card data. Enable one-click checkout and recurring payments while reducing your compliance scope.

- **PCI DSS Level 1**: Certified secure storage
- **Network Tokenization**: Automatic network token provisioning
- **External Vault Support**: Connect your existing vault (Spreedly, TokenEx, etc.)
- **Zero Data Exposure**: Your servers never handle raw card data

***

## Choosing Your Integration Model

Select the integration approach that best fits your architecture and requirements:

| Factor | SDK Payments (Model 1) | Server-to-Server (Model 2) |
|--------|------------------------|---------------------------|
| **Control** | SDK handles tokenization | You control the entire flow |
| **PCI Scope** | Minimal (SAQ A) | Higher (SAQ D) |
| **Complexity** | Lower - quick to implement | Higher - more flexibility |
| **Best For** | Standard checkout flows | Complex workflows, pre-auth |
| **Timing** | Tokenize at payment time | Tokenize before payment |

***

## Integration Model 1: Client-Side SDK Payments

**Tokenize at Payment Time | SDK-Initiated Execution**

Use this model when you want a quick, secure checkout with minimal backend complexity. The SDK handles payment confirmation directly.

### When to Choose This Model

- ✅ You want dynamic, frontend-driven payment experiences
- ✅ You prefer minimal backend orchestration logic
- ✅ You need SDK-triggered payment confirmation
- ✅ You're optimizing for rapid checkout implementation
- ✅ You want minimal PCI compliance scope

### Installation

```bash
# npm
npm install @hyperswitch/react-js

# yarn
yarn add @hyperswitch/react-js
```

### Integration Flow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Merchant   │────▶│  Hyperswitch │────▶│     SDK      │────▶│  Customer    │
│   Server     │     │   Backend    │     │   (Frontend) │     │  Browser     │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
       │                    │                    │                    │
       │ 1. Create Payment  │                    │                    │
       │───────────────────▶│                    │                    │
       │                    │ 2. Return client_secret                     │
       │◀───────────────────│                    │                    │
       │ 3. Load SDK with client_secret            │                    │
       │────────────────────────────────────────▶│                    │
       │                    │                    │ 4. Collect card    │
       │                    │                    │───────────────────▶│
       │                    │ 5. Confirm payment │◀───────────────────│
       │                    │◀───────────────────│                    │
       │                    │ 6. Route & process │                    │
       │                    │────┬───────────────┘                    │
       │                    │    │                                    │
       │                    │    ▼                                    │
       │                    │ ┌──────────┐                            │
       │                    │ │   PSP    │                            │
       │                    │ │(Stripe,  │                            │
       │                    │ │ Adyen...)│                            │
       │                    │ └──────────┘                            │
       │ 7. Return status   │                    │                    │
       │◀───────────────────│                    │                    │
```

### Step-by-Step Implementation

#### Step 1: Create a Payment (Server-Side)

```bash
curl --request POST \
  --url https://sandbox.hyperswitch.io/v1/payments \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "amount": 10000,
    "currency": "USD",
    "customer_id": "cust_12345",
    "profile_id": "pro_xxxxxxxx",
    "description": "Payment for Order #12345",
    "capture_method": "automatic"
  }'
```

**Response:**
```json
{
  "payment_id": "pay_1234567890abcdef",
  "client_secret": "pay_1234567890abcdef_secret_xxxxxxxx",
  "status": "requires_confirmation",
  "amount": 10000,
  "currency": "USD"
}
```

#### Step 2: Initialize the SDK (Client-Side)

```jsx
import { HyperProvider, Checkout } from '@hyperswitch/react-js';

function PaymentCheckout({ clientSecret }) {
  const hyperOptions = {
    publishableKey: 'YOUR_PUBLISHABLE_KEY',
    profileId: 'YOUR_PROFILE_ID',
  };

  return (
    <HyperProvider hyperOptions={hyperOptions}>
      <Checkout clientSecret={clientSecret} />
    </HyperProvider>
  );
}
```

#### Step 3: Handle Payment Completion

```javascript
// The SDK automatically handles:
// 1. Secure card collection
// 2. Tokenization
// 3. Payment confirmation
// 4. Routing to optimal PSP
// 5. Returning final status

// Listen for payment completion
hyper.on('payment_completed', (event) => {
  console.log('Payment status:', event.detail.status);
  // Redirect to success page
});
```

***

## Integration Model 2: Server-to-Server (S2S) Payments

**Tokenize Before Payment | Backend-Controlled Execution**

Use this model when you need granular control over transaction timing, or when you want to vault cards separately from payment execution.

### When to Choose This Model

- ✅ You want granular control over transaction timing
- ✅ You require backend-driven orchestration logic
- ✅ You want to tokenize credentials before execution
- ✅ You prefer decoupling vaulting from transaction processing
- ✅ You have existing PSP integrations to maintain

### Option A: Full Hyperswitch Orchestration (Recommended)

Let Hyperswitch handle routing, retries, and failover while you control the timing.

#### Step 1: Create a Customer (Optional)

```bash
curl --request POST \
  --url https://sandbox.hyperswitch.io/v1/customers \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "email": "customer@example.com",
    "name": "John Doe",
    "phone": "+1234567890"
  }'
```

#### Step 2: Tokenize the Card (Server-Side)

```bash
curl --request POST \
  --url https://sandbox.hyperswitch.io/v2/payment-methods \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "customer_id": "cust_12345",
    "card": {
      "card_number": "4242424242424242",
      "card_exp_month": "12",
      "card_exp_year": "2027",
      "card_holder_name": "John Doe"
    }
  }'
```

**Response:**
```json
{
  "payment_method_id": "pm_0196f252baa1736190bf0fc81b9651ea",
  "customer_id": "cust_12345",
  "payment_method": "card",
  "card": {
    "scheme": "Visa",
    "last4_digits": "4242",
    "expiry_month": "12",
    "expiry_year": "2027"
  }
}
```

#### Step 3: Create and Confirm Payment

```bash
curl --request POST \
  --url https://sandbox.hyperswitch.io/v2/payments/create-confirm \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "amount": 10000,
    "currency": "USD",
    "customer_id": "cust_12345",
    "payment_method_id": "pm_0196f252baa1736190bf0fc81b9651ea",
    "confirm": true,
    "profile_id": "pro_xxxxxxxx",
    "capture_method": "automatic"
  }'
```

### Option B: Proxy API (Incremental Migration)

Use the Proxy API when you want to keep your existing PSP integration while gradually adopting Hyperswitch features.

```bash
curl --request POST \
  --url https://sandbox.hyperswitch.io/proxy \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'X-Profile-Id: YOUR_PROFILE_ID' \
  --header 'Content-Type: application/json' \
  --data '{
    "request_body": {
      "source": {
        "type": "card",
        "number": "{{$card_number}}",
        "expiry_month": "{{$card_exp_month}}",
        "expiry_year": "{{$card_exp_year}}"
      },
      "amount": 10000,
      "currency": "USD"
    },
    "destination_url": "https://api.stripe.com/v1/charges",
    "headers": {
      "Authorization": "Bearer YOUR_STRIPE_KEY"
    },
    "token": "pm_0196f252baa1736190bf0fc81b9651ea",
    "token_type": "payment_method_id",
    "method": "POST"
  }'
```

**How it works:**
1. Hyperswitch receives the request with a `payment_method_id`
2. Vault detokenizes the card data
3. Hyperswitch replaces `{{$card_number}}` placeholders with actual values
4. Request is forwarded to your specified PSP
5. PSP response is returned unchanged

***

## Testing Your Integration

### Sandbox Environment

All examples use the Hyperswitch Sandbox. Use these base URLs:

| Environment | Base URL |
|-------------|----------|
| Sandbox | `https://sandbox.hyperswitch.io` |
| Production | `https://api.hyperswitch.io` |

### Test Card Numbers

Use these cards in the sandbox environment:

| Brand | Card Number | Expiry | CVC | Result |
|-------|-------------|--------|-----|--------|
| Visa | `4242424242424242` | Any future | Any 3 digits | Success |
| Visa (Declined) | `4000000000000002` | Any future | Any 3 digits | Generic decline |
| Mastercard | `5555555555554444` | Any future | Any 3 digits | Success |
| Amex | `378282246310005` | Any future | Any 4 digits | Success |
| 3DS Required | `4000002500003155` | Any future | Any 3 digits | 3D Secure challenge |

### Test Scenarios

#### Scenario 1: Successful Payment Flow

```bash
# 1. Create payment
curl -X POST https://sandbox.hyperswitch.io/v1/payments \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"amount":10000,"currency":"USD"}'

# Expected: Returns payment_id and client_secret with status "requires_confirmation"

# 2. Confirm with test card
# (Use SDK or S2S tokenization with 4242424242424242)

# Expected: Final status "succeeded"
```

#### Scenario 2: Declined Payment

```bash
# Use card number: 4000000000000002

# Expected: Status "failed" with error code "card_declined"
```

#### Scenario 3: 3D Secure Authentication

```bash
# Use card number: 4000002500003155

# Expected: Status "requires_customer_action" with 3DS URL
# Complete 3DS simulation at the provided URL
# Final status: "succeeded"
```

### Webhook Testing

Set up webhooks to receive async notifications:

1. Go to [Dashboard → Webhooks](https://app.hyperswitch.io/webhooks)
2. Add your endpoint URL
3. Select events: `payment_succeeded`, `payment_failed`, `refund_succeeded`
4. Use [webhook.site](https://webhook.site) for testing

***

## Error Handling

### Common Error Codes

| Code | Description | Resolution |
|------|-------------|------------|
| `card_declined` | Card was declined by issuer | Ask customer to use different card or contact bank |
| `insufficient_funds` | Card has insufficient funds | Ask customer to use different card |
| `expired_card` | Card has expired | Ask customer to update card details |
| `incorrect_cvc` | CVC check failed | Ask customer to check CVC and retry |
| `processing_error` | Temporary processing error | Retry the payment |
| `authentication_required` | 3D Secure required | Complete authentication flow |

### Error Response Format

```json
{
  "error": {
    "type": "card_error",
    "code": "card_declined",
    "message": "Your card was declined.",
    "decline_code": "insufficient_funds",
    "payment_id": "pay_1234567890abcdef"
  }
}
```

***

## Next Steps

- **[Explore Payment Methods](../../explore-hyperswitch/payment-flows-and-management/quickstart/payment-methods-setup/)** - Configure cards, wallets, and local payment methods
- **[Set up Routing](../../explore-hyperswitch/payment-orchestration/smart-router.md)** - Configure intelligent payment routing
- **[Enable Smart Retries](../../explore-hyperswitch/workflows/smart-retries/)** - Automatically recover failed payments
- **[Go Live](../../check-list-for-production/going-live/)** - Production deployment checklist
- **[API Reference](https://api-reference.hyperswitch.io/)** - Complete API documentation

***

## Need Help?

- 💬 [Join our Slack Community](https://join.slack.com/t/hyperswitch-io/shared_invite) - Get help from the team and community
- 📧 [Contact Support](mailto:support@hyperswitch.io) - Direct email support
- 🐛 [GitHub Issues](https://github.com/juspay/hyperswitch/issues) - Report bugs or request features
- 📖 [GitHub Repository](https://github.com/juspay/hyperswitch) - Star us and explore the code
