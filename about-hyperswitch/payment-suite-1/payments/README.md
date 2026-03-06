---
icon: file-invoice-dollar
---

# Payments (cards)

{% hint style="success" %}
## TL;DR

Hyperswitch Payments provides a unified API for processing card payments across 200+ processors. You can implement one-time payments, save payment methods for future use, and set up recurring billing with minimal code changes. The platform handles 3D Secure authentication, PCI compliance, and intelligent routing automatically. Integrate via our REST API or SDKs to start accepting payments in minutes.
{% endhint %}

Hyperswitch provides flexible payment processing with multiple flow patterns to accommodate different business needs. The system supports one-time payments, saved payment methods, and recurring billing through a comprehensive API design.

## Prerequisites

Before you start integrating card payments, ensure you have the following:

### Account Setup
- [ ] Create a Hyperswitch account at [app.hyperswitch.io](https://app.hyperswitch.io/)
- [ ] Generate API keys from the dashboard (Settings → API Keys)
- [ ] Configure at least one payment processor connector

### Technical Requirements
- [ ] API base URL: `https://api.hyperswitch.io/v1` (current stable version)
- [ ] Authentication: Include your API key in the `api-key` header
- [ ] Webhook endpoint configured to receive event notifications

### Compliance Checklist
{% hint style="warning" %}
**PCI DSS Compliance**

Hyperswitch is PCI DSS Level 1 compliant. When you use our hosted checkout or vault services, you reduce your compliance scope significantly. For self-hosted card data collection, you remain responsible for your own PCI compliance. Contact our support team for guidance on SAQ A, SAQ A-EP, or full DSS assessments.
{% endhint %}

## Integration Models Compared

| Aspect | Model 1: SDK Payments | Model 2: Server-to-Server |
|--------|----------------------|---------------------------|
| **Tokenisation** | SDK tokenises card data client-side | You tokenise via API call |
| **PCI Scope** | Minimal (SAQ A eligible) | Higher (SAQ A-EP or full DSS) |
| **Implementation** | Drop-in UI components | Custom UI + API integration |
| **Best For** | Web/mobile apps | Headless/complex workflows |
| **3D Secure** | Handled automatically | Manual integration required |

## SDK Integration Example

```javascript
// Initialise Hyperswitch SDK
const hyper = await Hyper("your_publishable_key");

// Mount payment element
const elements = hyper.elements();
const cardElement = elements.create("card");
cardElement.mount("#card-element");

// Tokenise and confirm payment
const { paymentIntent, error } = await hyper.confirmPayment({
  elements,
  clientSecret: "pi_xxx_secret_yyy",
  confirmParams: {
    return_url: "https://your-site.com/payment-complete"
  }
});

if (error) {
  console.error("Payment failed:", error.message);
} else {
  console.log("Payment succeeded:", paymentIntent.id);
}
```

{% hint style="info" %}
### When Should I Use the SDK?

Choose the **SDK integration** when you want to minimise PCI compliance burden and implement payments quickly. The SDK securely collects card details in the browser, tokenises them, and communicates directly with Hyperswitch servers. This approach keeps sensitive data away from your servers entirely.
{% endhint %}

## One-Time Payment Patterns

### How Do I Process an Instant Payment?

**Use Case:** Simple, immediate payment processing where funds are captured automatically upon authorisation.

**Endpoint:** `POST /v1/payments`

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

**Required Fields:**

* `confirm: true`
* `capture_method: "automatic"`
* `payment_method`

**Final Status:** `succeeded`

### How Do I Authorise Now and Capture Later?

**Use Case:** Deferred capture for scenarios like shipping goods before charging the customer.

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

**Flow:**

1. **Authorise:** `POST /v1/payments` with `capture_method: "manual"`
2. **Status:** `requires_capture`
3. **Capture:** `POST /v1/payments/{payment_id}/capture`
4. **Final Status:** `succeeded`

Read more in our [manual capture documentation](https://docs.hyperswitch.io/about-hyperswitch/payment-suite-1/payments-cards/manual-capture).

### How Do I Build a Multi-Step Checkout?

**Use Case:** Complex checkout journeys with progressive data collection, such as headless checkouts or B2B portals.

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

**Endpoints:**

* **Create:** `POST /v1/payments`
* **Update:** `POST /v1/payments/{payment_id}`
* **Confirm:** `POST /v1/payments/{payment_id}/confirm`
* **Capture:** `POST /v1/payments/{payment_id}/capture` (if manual)

### How Does 3D Secure Authentication Work?

**Use Case:** Enhanced security with customer authentication for high-value transactions or regulatory requirements.

<figure><img src="../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

**Additional Fields:**

* `authentication_type: "three_ds"`

**Status Progression:** `processing` → `requires_customer_action` → `succeeded`

Read more about [3D Secure decision management](https://docs.hyperswitch.io/explore-hyperswitch/workflows/3ds-decision-manager).

## Recurring Payments and Payment Storage

### How Do I Save a Payment Method?

**During Payment Creation:**

* Add `setup_future_usage: "off_session"` or `"on_session"`
* Include `customer_id`
* **Result:** `payment_method_id` returned on success

**Understanding `setup_future_usage`:**

* **`on_session`**: Use when the customer is actively present during the transaction. This is typical for scenarios like saving card details for faster checkouts in subsequent sessions where the customer will still be present to initiate the payment (e.g., card vaulting for e-commerce sites).
* **`off_session`**: Use when you intend to charge the customer later without their active involvement at the time of charge. This is suitable for subscriptions, recurring billing, or merchant-initiated transactions (MITs) where the customer has pre-authorised future charges.

### How Do I Use Saved Payment Methods?

<figure><img src="../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

**Steps:**

1. **Initiate:** Create payment with `customer_id`
2. **List:** Get saved cards via `GET /v1/customers/{customer_id}/payment_methods`
3. **Confirm:** Use selected `payment_token` in confirm call

{% hint style="info" %}
**PCI Compliance and `payment_method_id`**

Storing `payment_method_id` (which is a token representing the actual payment instrument, which could be a payment token, network token, or payment processor token) significantly reduces your PCI DSS scope. Hyperswitch securely stores the sensitive card details and provides you with this token. While you still need to ensure your systems handle `payment_method_id` and related customer data securely, you avoid the complexities of storing raw card numbers. Always consult with a PCI QSA to understand your specific compliance obligations.
{% endhint %}

## Recurring Payment Flows

### Customer-Initiated Transaction (CIT) Setup

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Read more about [recurring payments](https://docs.hyperswitch.io/about-hyperswitch/payment-suite-1/payments-cards/recurring-payments).

### Merchant-Initiated Transaction (MIT) Execution

<figure><img src="../../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

Read more about [recurring payments](https://docs.hyperswitch.io/about-hyperswitch/payment-suite-1/payments-cards/recurring-payments).

## Webhooks and Error Handling

### What Webhook Events Should I Handle?

Configure your webhook endpoint to receive these essential events:

| Event | Description | Action Required |
|-------|-------------|-----------------|
| `payment_intent.succeeded` | Payment completed successfully | Fulfill order, send confirmation |
| `payment_intent.payment_failed` | Payment failed | Notify customer, offer retry |
| `payment_intent.requires_action` | 3D Secure or other authentication needed | Redirect customer if applicable |
| `charge.dispute.created` | Chargeback initiated | Respond with evidence |

### How Do I Handle Common Errors?

```json
{
  "error": {
    "type": "invalid_request",
    "code": "card_declined",
    "decline_code": "insufficient_funds",
    "message": "Your card was declined.",
    "payment_intent": {
      "id": "pi_xxx",
      "status": "requires_payment_method"
    }
  }
}
```

**Retry Strategies:**
- **Temporary failures** (network timeouts): Retry with exponential backoff
- **Card declines**: Prompt customer for different payment method
- **Authentication required**: Redirect to 3D Secure flow

{% hint style="info" %}
**Idempotency for Safe Retries**

Always include an `Idempotency-Key` header (UUID v4) for `POST` requests. This ensures duplicate requests with the same key return the same response without creating duplicate payments.
{% endhint %}

## Status Flow Summary

<figure><img src="../../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

### What Are the Terminal States?

- **Terminal States:** `succeeded`, `failed`, `cancelled`, `partially_captured` are terminal states requiring no further action
- **Capture Methods:** System supports `automatic` (funds captured immediately), `manual` (funds captured in a separate step), `manual_multiple` (funds captured in multiple partial amounts via separate steps), and `scheduled` (funds captured automatically at a future predefined time) capture methods
- **Authentication:** 3D Secure authentication automatically resumes payment processing after customer completion
- **MIT Compliance:** Off-session recurring payments follow industry standards for merchant-initiated transactions

## Next Steps

- [Integrate the SDK](https://docs.hyperswitch.io/hyperswitch-cloud/integration-guide)
- [Explore the API Reference](https://api-reference.hyperswitch.io/)
- [Join our Slack Community](https://inviter.co/hyperswitch-slack) for support
