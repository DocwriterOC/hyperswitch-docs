---
icon: file-invoice-dollar
---

# Payments (Cards)

Process card payments with flexibility and security. Hyperswitch supports multiple payment flows—from simple one-time charges to complex recurring billing—while keeping your PCI compliance scope minimal through secure tokenization.

{% hint style="success" %}
**Why Hyperswitch for Card Payments?**
- **Reduced PCI Scope**: Card data never touches your servers
- **Multiple Capture Options**: Automatic, manual, partial, and scheduled captures
- **3D Secure Ready**: Built-in support for customer authentication
- **Smart Routing**: Automatic failover between payment processors
{% endhint %}

## Table of Contents

- [Quick Start](#quick-start)
- [Prerequisites](#prerequisites)
- [Core Concepts](#core-concepts)
- [Payment Flows](#payment-flows)
  - [Instant Payment (Automatic Capture)](#1-instant-payment-automatic-capture)
  - [Two-Step Manual Capture](#2-two-step-manual-capture)
  - [Fully Decoupled Flow](#3-fully-decoupled-flow)
  - [3D Secure Authentication](#4-3d-secure-authentication-flow)
- [Recurring Payments](#recurring-payments-and-payment-storage)
- [Saved Payment Methods](#using-saved-payment-methods)
- [Testing](#testing)
- [API Reference](#api-reference)
- [Webhooks](#webhooks)
- [Troubleshooting](#troubleshooting)

---

## Quick Start

Get your first card payment processed in under 5 minutes:

### Step 1: Create a Payment

```bash
curl --location 'https://sandbox.hyperswitch.io/payments' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--header 'X-Idempotency-Key: unique-key-123' \
--data-raw '{
    "amount": 10000,
    "currency": "USD",
    "customer_id": "cust_123",
    "profile_id": "pro_abc123",
    "email": "customer@example.com",
    "description": "Test payment",
    "confirm": false,
    "capture_method": "automatic",
    "return_url": "https://example.com/payment/complete"
}'
```

**Response:**

```json
{
    "payment_id": "pay_jXzPoRHB9PiYBZc5TD3A",
    "client_secret": "pay_jXzPoRHB9PiYBZc5TD3A_secret",
    "status": "requires_confirmation",
    "amount": 10000,
    "currency": "USD",
    "customer_id": "cust_123",
    "created": "2024-03-06T10:30:00.000Z"
}
```

### Step 2: Confirm with Card Details (SDK)

```javascript
// Initialize Hyperswitch SDK
const hyper = window.Hyper("YOUR_PUBLISHABLE_KEY");
const widgets = hyper.widgets({ 
    clientSecret: "pay_jXzPoRHB9PiYBZc5TD3A_secret" 
});

// Create and mount payment element
const checkout = widgets.create("payment", {
    layout: "tabs",
    wallets: {
        walletReturnUrl: "https://example.com/payment/complete"
    }
});
checkout.mount("#payment-element");

// Confirm payment
const { error } = await hyper.confirmPayment({
    elements: widgets,
    confirmParams: {
        return_url: "https://example.com/payment/complete"
    }
});
```

### Step 3: Handle the Result

The customer is redirected to your `return_url` with the payment status. Check the status using:

```bash
curl --location 'https://sandbox.hyperswitch.io/payments/pay_jXzPoRHB9PiYBZc5TD3A' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY'
```

---

## Prerequisites

Before integrating card payments, ensure you have:

| Requirement | Description |
|-------------|-------------|
| **Hyperswitch Account** | Sign up at [app.hyperswitch.io](https://app.hyperswitch.io) |
| **API Keys** | Generate API key and Publishable key from Dashboard → Developers → API Keys |
| **Business Profile** | Create a business profile and configure at least one card processor (Stripe, Adyen, etc.) |
| **PCI Compliance** | For SDK integration: SAQ A eligible. For direct API: SAQ D required. |

### Environment URLs

| Environment | Base URL |
|-------------|----------|
| Sandbox | `https://sandbox.hyperswitch.io` |
| Production | `https://api.hyperswitch.io` |

---

## Core Concepts

### Terminology

| Term | Description | Example |
|------|-------------|---------|
| `payment_method` | Type of payment instrument | `card`, `wallet`, `bank_transfer` |
| `payment_method_type` | Specific variant | `credit`, `debit` |
| `payment_method_id` | Token representing stored payment method | `pm_abc123xyz` |
| `payment_token` | Short-lived token for confirming payment | `token_xyz789` |
| `client_secret` | Secret key for client-side operations | `pay_xxxxx_secret` |
| `setup_future_usage` | Intent for storing payment method | `on_session`, `off_session` |
| `CIT` | Customer-Initiated Transaction | Customer actively present |
| `MIT` | Merchant-Initiated Transaction | Background/recurring charge |

### Capture Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| `automatic` | Funds captured immediately | Simple e-commerce |
| `manual` | Authorization only, capture later | Ship before charging |
| `manual_multiple` | Multiple partial captures | Installment releases |
| `scheduled` | Auto-capture at future time | Pre-ordered items |

### Payment Status Flow

```
created → requires_confirmation → processing → succeeded
                     ↓
              requires_customer_action (3DS)
                     ↓
              requires_capture (manual capture)
                     ↓
              failed / cancelled / partially_captured
```

**Terminal States** (no further action required):
- `succeeded` - Payment completed successfully
- `failed` - Payment failed, can retry with different method
- `cancelled` - Payment was cancelled
- `partially_captured` - Partial capture completed (manual_multiple)

---

## Payment Flows

Choose the right flow based on your business requirements:

| Flow | Capture | Customer Present | Complexity | Best For |
|------|---------|------------------|------------|----------|
| **Instant** | Automatic | Yes | Low | Standard checkout |
| **Manual** | Manual | Yes | Medium | Delayed shipping |
| **Decoupled** | Either | Yes | High | Progressive checkout |
| **3D Secure** | Either | Yes | Medium | High-risk transactions |

### 1. Instant Payment (Automatic Capture)

**Use Case:** Standard e-commerce checkout where goods are shipped immediately or services delivered instantly.

<figure><img src="../../../.gitbook/assets/image (39).png" alt="Instant Payment Flow"><figcaption>Instant payment with automatic capture</figcaption></figure>

**API Flow:**

```bash
# Create and confirm in one request
curl --location 'https://sandbox.hyperswitch.io/payments' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--header 'X-Idempotency-Key: instant-pay-001' \
--data-raw '{
    "amount": 5000,
    "currency": "USD",
    "confirm": true,
    "capture_method": "automatic",
    "customer_id": "cust_123",
    "profile_id": "pro_abc123",
    "payment_method": "card",
    "payment_method_type": "credit",
    "payment_method_data": {
        "card": {
            "card_number": "4242424242424242",
            "card_exp_month": "12",
            "card_exp_year": "2027",
            "card_holder_name": "John Doe",
            "card_cvc": "123"
        }
    },
    "return_url": "https://example.com/complete"
}'
```

**Response:**

```json
{
    "payment_id": "pay_abc123",
    "status": "succeeded",
    "amount": 5000,
    "amount_received": 5000,
    "currency": "USD",
    "capture_method": "automatic",
    "customer_id": "cust_123"
}
```

**Required Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `amount` | integer | Yes | Amount in smallest currency unit (cents) |
| `currency` | string | Yes | Three-letter ISO currency code |
| `confirm` | boolean | Yes | Set to `true` for instant payment |
| `capture_method` | string | Yes | Set to `"automatic"` |
| `payment_method` | string | Yes | `"card"` for card payments |
| `payment_method_data` | object | Yes* | Card details (*or use SDK) |

---

### 2. Two-Step Manual Capture

**Use Case:** When you need to authorize funds but capture only after shipping, service completion, or inventory confirmation.

<figure><img src="../../../.gitbook/assets/image (42).png" alt="Manual Capture Flow"><figcaption>Authorize now, capture later</figcaption></figure>

**Step 1: Create Authorization**

```bash
curl --location 'https://sandbox.hyperswitch.io/payments' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--data-raw '{
    "amount": 10000,
    "currency": "USD",
    "confirm": true,
    "capture_method": "manual",
    "customer_id": "cust_123",
    "profile_id": "pro_abc123",
    "payment_method": "card",
    "payment_method_data": {
        "card": {
            "card_number": "4242424242424242",
            "card_exp_month": "12",
            "card_exp_year": "2027",
            "card_holder_name": "John Doe",
            "card_cvc": "123"
        }
    }
}'
```

**Response:**

```json
{
    "payment_id": "pay_manual_123",
    "status": "requires_capture",
    "amount": 10000,
    "amount_capturable": 10000,
    "amount_received": 0,
    "capture_method": "manual"
}
```

**Step 2: Capture Funds**

```bash
# Full capture
curl --location 'https://sandbox.hyperswitch.io/payments/pay_manual_123/capture' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--data-raw '{
    "amount_to_capture": 10000
}'
```

**Partial Capture:**

```bash
# Capture partial amount
curl --location 'https://sandbox.hyperswitch.io/payments/pay_manual_123/capture' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--data-raw '{
    "amount_to_capture": 7500
}'
```

**Response:**

```json
{
    "payment_id": "pay_manual_123",
    "status": "partially_captured",
    "amount": 10000,
    "amount_capturable": 2500,
    "amount_received": 7500
}
```

{% hint style="info" %}
Learn more about [Manual Capture](manual-capture/) and [Overcapture](manual-capture/overcapture.md) for capturing amounts greater than the original authorization.
{% endhint %}

---

### 3. Fully Decoupled Flow

**Use Case:** Complex checkout journeys where payment data is collected progressively, such as headless checkout, B2B portals, or multi-step forms.

<figure><img src="../../../.gitbook/assets/image (43).png" alt="Decoupled Flow"><figcaption>Create → Update → Confirm → Capture</figcaption></figure>

**Step 1: Create Payment Intent**

```bash
curl --location 'https://sandbox.hyperswitch.io/payments' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--data-raw '{
    "amount": 15000,
    "currency": "USD",
    "confirm": false,
    "customer_id": "cust_123",
    "profile_id": "pro_abc123"
}'
```

**Response:**

```json
{
    "payment_id": "pay_decoupled_123",
    "client_secret": "pay_decoupled_123_secret",
    "status": "requires_confirmation",
    "amount": 15000,
    "currency": "USD"
}
```

**Step 2: Update Payment (Optional)**

```bash
curl --location 'https://sandbox.hyperswitch.io/payments/pay_decoupled_123' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--data-raw '{
    "amount": 16000,
    "shipping": {
        "address": {
            "line1": "123 Main St",
            "city": "San Francisco",
            "state": "CA",
            "zip": "94105",
            "country": "US"
        }
    }
}'
```

**Step 3: Confirm Payment**

```bash
curl --location 'https://sandbox.hyperswitch.io/payments/pay_decoupled_123/confirm' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--data-raw '{
    "payment_method": "card",
    "payment_method_data": {
        "card": {
            "card_number": "4242424242424242",
            "card_exp_month": "12",
            "card_exp_year": "2027",
            "card_holder_name": "John Doe",
            "card_cvc": "123"
        }
    },
    "return_url": "https://example.com/complete"
}'
```

**Step 4: Capture (if manual)**

```bash
curl --location 'https://sandbox.hyperswitch.io/payments/pay_decoupled_123/capture' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--data-raw '{
    "amount_to_capture": 16000
}'
```

**Endpoints Summary:**

| Endpoint | Purpose |
|----------|---------|
| `POST /payments` | Create a new payment intent |
| `POST /payments/{id}` | Update payment details |
| `POST /payments/{id}/confirm` | Confirm and process payment |
| `POST /payments/{id}/capture` | Capture authorized funds |

---

### 4. 3D Secure Authentication Flow

**Use Case:** Enhanced security for high-risk transactions or regulatory compliance (PSD2 SCA).

<figure><img src="../../../.gitbook/assets/image (44).png" alt="3DS Flow"><figcaption>3D Secure customer authentication</figcaption></figure>

**Request:**

```bash
curl --location 'https://sandbox.hyperswitch.io/payments' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--data-raw '{
    "amount": 20000,
    "currency": "USD",
    "confirm": true,
    "capture_method": "automatic",
    "authentication_type": "three_ds",
    "customer_id": "cust_123",
    "profile_id": "pro_abc123",
    "payment_method": "card",
    "payment_method_data": {
        "card": {
            "card_number": "4242424242424242",
            "card_exp_month": "12",
            "card_exp_year": "2027",
            "card_holder_name": "John Doe",
            "card_cvc": "123"
        }
    },
    "return_url": "https://example.com/3ds-complete"
}'
```

**Status Progression:**

```
processing → requires_customer_action → succeeded
                    ↓
              (Customer completes 3DS)
                    ↓
              redirect to return_url
```

**Handling 3DS Response:**

```javascript
// Check payment status after 3DS redirect
const result = await hyper.retrievePaymentIntent(clientSecret);

if (result.paymentIntent.status === 'succeeded') {
    // Payment successful
} else if (result.paymentIntent.status === 'requires_customer_action') {
    // Additional action needed
} else if (result.paymentIntent.status === 'failed') {
    // Handle failure
}
```

{% hint style="info" %}
Learn more about [3DS Decision Manager](../../explore-hyperswitch/workflows/3ds-decision-manager.md) for configuring authentication rules.
{% endhint %}

---

## Recurring Payments and Payment Storage

Store payment methods for future use with customer consent.

### Saving Payment Methods

**During Payment Creation:**

```bash
curl --location 'https://sandbox.hyperswitch.io/payments' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--data-raw '{
    "amount": 5000,
    "currency": "USD",
    "confirm": true,
    "customer_id": "cust_123",
    "profile_id": "pro_abc123",
    "setup_future_usage": "off_session",
    "payment_method": "card",
    "payment_method_data": {
        "card": {
            "card_number": "4242424242424242",
            "card_exp_month": "12",
            "card_exp_year": "2027",
            "card_holder_name": "John Doe",
            "card_cvc": "123"
        }
    },
    "customer_acceptance": {
        "acceptance_type": "online",
        "accepted_at": "2024-03-06T10:30:00Z",
        "online": {
            "ip_address": "192.168.1.1",
            "user_agent": "Mozilla/5.0..."
        }
    }
}'
```

**Response:**

```json
{
    "payment_id": "pay_recurring_123",
    "status": "succeeded",
    "payment_method_id": "pm_stored_456",
    "network_transaction_id": "ntid_789"
}
```

### Understanding `setup_future_usage`

| Value | Use When | Example |
|-------|----------|---------|
| `on_session` | Customer will be present for future payments | Express checkout for returning customers |
| `off_session` | Charging without customer present | Subscriptions, auto-renewals |

### Customer Consent Capture

For compliance with card network rules, you must capture explicit customer consent before storing payment methods:

```json
"customer_acceptance": {
    "acceptance_type": "online",
    "accepted_at": "2024-03-06T10:30:00Z",
    "online": {
        "ip_address": "192.168.1.1",
        "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
    }
}
```

{% hint style="info" %}
When using Hyperswitch SDK, `customer_acceptance` is automatically sent when the customer checks "Save card for future use".
{% endhint %}

---

## Using Saved Payment Methods

<figure><img src="../../../.gitbook/assets/image (45).png" alt="Saved Payment Methods"><figcaption>Retrieve and use stored payment methods</figcaption></figure>

### Step 1: List Saved Payment Methods

```bash
curl --location 'https://sandbox.hyperswitch.io/customers/cust_123/payment_methods' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY'
```

**Response:**

```json
{
    "customer_payment_methods": [
        {
            "payment_method_id": "pm_stored_456",
            "payment_method": "card",
            "payment_method_type": "credit",
            "card": {
                "scheme": "visa",
                "last4_digits": "4242",
                "expiry_month": "12",
                "expiry_year": "2027"
            }
        }
    ]
}
```

### Step 2: Use Saved Payment Method

```bash
curl --location 'https://sandbox.hyperswitch.io/payments' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--data-raw '{
    "amount": 3000,
    "currency": "USD",
    "confirm": true,
    "customer_id": "cust_123",
    "profile_id": "pro_abc123",
    "payment_method": "card",
    "payment_token": "pm_stored_456"
}'
```

### PCI Compliance and `payment_method_id`

Storing `payment_method_id` (a secure token) instead of raw card data significantly reduces your PCI DSS scope. Hyperswitch handles sensitive card storage in a PCI-compliant vault.

| Approach | PCI Scope | Description |
|----------|-----------|-------------|
| Hyperswitch SDK | SAQ A | Minimal compliance - no card data touches your systems |
| Direct API Integration | SAQ D | Full compliance required - card data passes through your servers |
| Token Only (`payment_method_id`) | SAQ A-EP | Reduced scope - only tokens stored |

Always consult with a PCI QSA for your specific compliance requirements.

---

## Recurring Payment Flows

### Customer-Initiated Transaction (CIT) Setup

<figure><img src="../../../.gitbook/assets/image (48).png" alt="CIT Flow"><figcaption>Customer-initiated transaction with card storage</figcaption></figure>

**Use Case:** First-time subscription signup with immediate charge or free trial setup.

{% hint style="info" %}
Detailed CIT implementation guide available in [Recurring Payments documentation](recurring-payments.md).
{% endhint %}

### Merchant-Initiated Transaction (MIT) Execution

<figure><img src="../../../.gitbook/assets/image (50).png" alt="MIT Flow"><figcaption>Merchant-initiated recurring charge</figcaption></figure>

**MIT with Stored Payment Method:**

```bash
curl --location 'https://sandbox.hyperswitch.io/payments' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--data-raw '{
    "amount": 2999,
    "currency": "USD",
    "confirm": true,
    "customer_id": "cust_123",
    "profile_id": "pro_abc123",
    "off_session": true,
    "recurring_details": {
        "type": "payment_method_id",
        "data": "pm_stored_456"
    }
}'
```

**MIT with Network Transaction ID:**

```bash
curl --location 'https://sandbox.hyperswitch.io/payments' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: YOUR_API_KEY' \
--data-raw '{
    "amount": 2999,
    "currency": "USD",
    "confirm": true,
    "customer_id": "cust_123",
    "profile_id": "pro_abc123",
    "off_session": true,
    "recurring_details": {
        "type": "network_transaction_id_and_card_details",
        "data": {
            "card_number": "4242424242424242",
            "card_exp_month": "12",
            "card_exp_year": "2027",
            "network_transaction_id": "ntid_789"
        }
    }
}'
```

{% hint style="info" %}
Learn more about [Recurring Payments](recurring-payments.md), including connector-agnostic MIT routing.
{% endhint %}

---

## Testing

### Test Card Numbers

Use these cards in the sandbox environment:

| Card Number | Brand | Scenario |
|-------------|-------|----------|
| `4242424242424242` | Visa | Successful payment |
| `4000000000000002` | Visa | Generic decline |
| `4000000000000127` | Visa | Insufficient funds |
| `4000000000000069` | Visa | Expired card |
| `4000000000000119` | Visa | Incorrect CVC |
| `4000002500003155` | Visa | 3D Secure required |
| `4000002760003184` | Visa | 3D Secure frictionless |
| `5555555555554444` | Mastercard | Successful payment |
| `378282246310005` | Amex | Successful payment |

### Testing Payment Flows

**Test Instant Payment:**

```bash
# Should return status: succeeded
curl -X POST 'https://sandbox.hyperswitch.io/payments' \
-H 'Content-Type: application/json' \
-H 'api-key: YOUR_API_KEY' \
-d '{
    "amount": 1000,
    "currency": "USD",
    "confirm": true,
    "capture_method": "automatic",
    "payment_method": "card",
    "payment_method_data": {
        "card": {
            "card_number": "4242424242424242",
            "card_exp_month": "12",
            "card_exp_year": "2027",
            "card_cvc": "123"
        }
    }
}'
```

**Test 3D Secure Flow:**

Use card `4000002500003155` and verify the status progresses through `requires_customer_action`.

**Test Saved Payment Method:**

1. Create payment with `setup_future_usage: "off_session"`
2. Store the returned `payment_method_id`
3. Create new payment using `payment_token: "pm_xxxxx"`

**Test MIT Flow:**

1. Complete a CIT with `setup_future_usage: "off_session"`
2. Wait for payment to succeed
3. Create MIT with `off_session: true` and the stored `payment_method_id`

---

## API Reference

### POST /payments

Create a new payment intent.

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `amount` | integer | Yes | Amount in smallest currency unit |
| `currency` | string | Yes | Three-letter ISO currency code |
| `confirm` | boolean | No | Confirm immediately (default: false) |
| `capture_method` | string | No | `automatic`, `manual`, `manual_multiple`, `scheduled` |
| `customer_id` | string | No | Customer identifier |
| `profile_id` | string | Yes* | Business profile ID (*required if multiple profiles) |
| `payment_method` | string | No | Payment method type |
| `payment_method_data` | object | No* | Required if `confirm: true` and not using SDK |
| `setup_future_usage` | string | No | `on_session` or `off_session` |
| `authentication_type` | string | No | `no_three_ds` or `three_ds` |
| `return_url` | string | No* | Required for 3DS and redirect flows |
| `email` | string | No | Customer email |
| `description` | string | No | Payment description |
| `metadata` | object | No | Custom key-value pairs |

**Response:**

| Field | Type | Description |
|-------|------|-------------|
| `payment_id` | string | Unique payment identifier |
| `client_secret` | string | Secret for client-side confirmation |
| `status` | string | Current payment status |
| `amount` | integer | Payment amount |
| `amount_capturable` | integer | Amount available to capture |
| `amount_received` | integer | Amount successfully captured |
| `currency` | string | Currency code |
| `payment_method_id` | string | Stored payment method token |

### POST /payments/{id}/confirm

Confirm a pending payment intent.

### POST /payments/{id}/capture

Capture authorized funds.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `amount_to_capture` | integer | No | Amount to capture (defaults to full amount) |

### GET /customers/{id}/payment_methods

List stored payment methods for a customer.

---

## Webhooks

Subscribe to webhook events to receive real-time payment status updates.

### Payment Events

| Event | Description |
|-------|-------------|
| `payment_intent.requires_action` | Payment requires customer action (3DS) |
| `payment_intent.succeeded` | Payment completed successfully |
| `payment_intent.failed` | Payment failed |
| `payment_intent.captured` | Funds captured |
| `payment_intent.partially_captured` | Partial capture completed |
| `payment_intent.cancelled` | Payment cancelled |

### Webhook Payload Example

```json
{
    "event_type": "payment_intent.succeeded",
    "payment_id": "pay_abc123",
    "status": "succeeded",
    "amount": 10000,
    "currency": "USD",
    "created": "2024-03-06T10:30:00Z"
}
```

Configure webhooks in Dashboard → Developers → Webhooks.

---

## Troubleshooting

### Common Errors

| Error Code | Cause | Solution |
|------------|-------|----------|
| `payment_method_not_available` | Payment method not enabled for profile | Enable card payment method in Dashboard |
| `connector_processing_failure` | Processor declined | Check card details or use different card |
| `requires_customer_action` | 3DS or redirect required | Handle authentication flow |
| `amount_exceeds_capturable` | Capture amount > authorized | Check `amount_capturable` before capture |
| `payment_already_captured` | Duplicate capture request | Check payment status before capturing |
| `idempotency_key_reused` | Same idempotency key, different payload | Use unique idempotency keys per request |

### Debug Steps

1. **Check Payment Status:**
   ```bash
   curl 'https://sandbox.hyperswitch.io/payments/{payment_id}' \
   -H 'api-key: YOUR_API_KEY'
   ```

2. **Verify Profile Configuration:**
   - Dashboard → Business Profiles → {profile} → Payment Methods
   - Ensure "card" is enabled

3. **Test with SDK:**
   - Use Hyperswitch SDK to isolate integration issues

4. **Check Webhook Logs:**
   - Dashboard → Developers → Webhooks → Logs

### Support

- **Documentation:** [docs.hyperswitch.io](https://docs.hyperswitch.io)
- **API Reference:** [api-reference.hyperswitch.io](https://api-reference.hyperswitch.io)
- **Support Email:** support@hyperswitch.io
- **Discord Community:** [Join here](https://discord.gg/hyperswitch)

---

## Status Flow Summary

<figure><img src="../../../.gitbook/assets/image (81).png" alt="Status Flow"><figcaption>Complete payment status lifecycle</figcaption></figure>

### Status Quick Reference

| Status | Action Required | Terminal |
|--------|-----------------|----------|
| `requires_confirmation` | Confirm payment | No |
| `requires_customer_action` | Complete 3DS/redirect | No |
| `requires_capture` | Capture funds | No |
| `processing` | Wait for completion | No |
| `succeeded` | None | Yes |
| `failed` | Retry or alternate method | Yes |
| `cancelled` | None | Yes |
| `partially_captured` | Capture remaining (optional) | Yes |
