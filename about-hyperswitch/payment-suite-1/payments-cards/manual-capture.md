---
description: >-
  Complete guide to implementing Manual Capture (Auth & Capture) payments with Hyperswitch. Learn to authorize funds, capture later, handle partial captures, overcapture, and webhooks with multi-language SDK examples.
icon: transmission
---

# Manual Capture

{% embed url="https://youtu.be/XtOMZVhvLwQ" %}

## Quick Start (5-Minute Guide)

Get your first Manual Capture payment running in under 5 minutes:

### Step 1: Create a Payment (Server-side)

```bash
curl --location 'https://sandbox.hyperswitch.io/payments' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: <your_api_key>' \
--data '{
    "amount": 6540,
    "currency": "USD",
    "confirm": false,
    "capture_method": "manual",
    "customer_id": "cust_123",
    "return_url": "https://your-site.com/return"
}'
```

Save the `payment_id` and `client_secret` from the response.

### Step 2: Confirm/Authorize (Client-side)

Use [Hyperswitch Unified Checkout](https://docs.hyperswitch.io/hyperswitch-cloud/integration-guide/web) to collect card details and authorize the payment. The payment status becomes `requires_capture`.

### Step 3: Capture Funds (Server-side)

```bash
curl --location 'https://sandbox.hyperswitch.io/payments/{payment_id}/capture' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: <your_api_key>' \
--data '{
    "amount_to_capture": 6540
}'
```

**Done!** You've successfully authorized and captured a payment.

[Try in Sandbox →](https://app.hyperswitch.io/dashboard) | [Contact Sales](https://hyperswitch.io/contact)

---

## Overview

Manual Capture (also known as "Auth and Capture" or "two-step payments") allows merchants to **authorize** funds on a customer's card without immediately charging them. The funds are held (blocked) for a period of time, and the merchant can **capture** (settle) the payment at a later time—either fully, partially, or even over the authorized amount (with overcapture enabled).

### What Makes Manual Capture Different?

| Flow | Automatic Capture | Manual Capture |
|------|-------------------|----------------|
| **Authorization** | ✅ Immediate | ✅ Immediate |
| **Capture** | Automatic after auth | Manual, when you choose |
| **Use Case** | Simple purchases | Deliver-later scenarios |
| **Risk** | Funds guaranteed at sale | Funds held, chargeable on delivery |
| **Customer Trust** | Standard | Higher (pay only on fulfillment) |

### Who Benefits from Manual Capture?

* **🏨 Hotels & Hospitality**: Authorize at booking, capture at checkout (add incidentals)
* **🛒 E-commerce Marketplaces**: Hold funds until seller ships, then capture
* **🚗 Car Rentals**: Block deposit at pickup, capture final amount on return
* **🍕 Food Delivery**: Authorize when order placed, capture when delivered
* **🎫 Event Ticketing**: Hold funds until event confirmation or seat assignment
* **🏠 Vacation Rentals**: Security deposit holds with variable final capture

---

## When to Use Manual Capture

### Decision Framework

**Choose Manual Capture when:**
1. You deliver goods/services after the order is placed
2. Final amount might differ from initial authorization
3. You need to hold security deposits
4. Multiple partial shipments/deliveries are expected

**Choose Automatic Capture when:**
1. Digital goods/services delivered immediately
2. Amount is fixed at checkout
3. No post-purchase adjustments expected

### Industry Use Cases

#### 🏨 Hotels & Hospitality
**Scenario**: Guest books a room for $200/night for 3 nights = $600
* **Authorization**: Block $600 at booking + $150 incidentals = $750
* **Capture**: $600 on checkout (release incidentals hold)
* **Overcapture enabled**: Add room service charges ($75) → Capture $675

**Business Outcome**: Zero chargebacks from unfulfilled bookings, reduced fraud exposure, improved guest satisfaction through transparent holds.

#### 🛒 E-commerce Marketplace
**Scenario**: Customer orders from 3 different sellers
* **Authorization**: Block total $350 at checkout
* **Capture 1**: $100 when Seller A ships
* **Capture 2**: $150 when Seller B ships
* **Capture 3**: $100 when Seller C ships

**Business Outcome**: Seller-agnostic payment flow, reduced platform liability, automated multi-party settlements.

#### 🚗 Car Rentals
**Scenario**: Weekly rental with variable mileage
* **Authorization**: Block $500 (base + $200 security deposit)
* **Capture**: $320 base rate on return
* **Adjustment**: Additional $45 fuel charge via overcapture

**Business Outcome**: Dynamic pricing flexibility, damage protection through holds, streamlined returns process.

### Manual vs Automatic Capture Comparison

| Factor | Manual Capture | Automatic Capture |
|--------|----------------|-------------------|
| **Implementation Complexity** | Medium | Low |
| **Settlement Speed** | When you capture | Immediate |
| **Refund Requirements** | Pre-capture: void only | Full refund process |
| **Chargeback Risk** | Lower (capture on fulfillment) | Higher |
| **Customer Perception** | Trust-building | Standard |
| **Best For** | Deliver-later, variable amounts | Digital goods, immediate delivery |

---

## Prerequisites

Before implementing Manual Capture, ensure you have:

### 1. Hyperswitch Account & API Keys

You need two types of API keys:

| Key Type | Purpose | Security Level | Where Used |
|----------|---------|----------------|------------|
| **API Key** (`api-key`) | Server-side operations | 🔒 High | Create payment, capture funds, refunds |
| **Publishable Key** | Client-side operations | 🔓 Lower | Confirm payment from frontend |

#### Obtaining Your Keys

1. Sign up at [Hyperswitch Dashboard](https://app.hyperswitch.io/)
2. Navigate to **Developer → API Keys**
3. Copy your **API Key** (starts with `snd_` for sandbox)
4. Copy your **Publishable Key** (starts with `pk_` for sandbox)

**⚠️ Never expose your API Key in client-side code or public repositories.** Use the Publishable Key for frontend operations only.

### 2. Supported Payment Methods

Manual Capture is supported for:

| Payment Method | Full Capture | Partial Capture | Over Capture | Multiple Captures |
|----------------|--------------|-----------------|--------------|-------------------|
| **Cards (Visa, Mastercard, Amex)** | ✅ | ✅ | ✅* | ✅* |
| **Wallets (Apple Pay, Google Pay)** | ✅ | ⚠️ Varies | ❌ | ❌ |
| **BNPL (Klarna, Affirm)** | ✅ | ❌ | ❌ | ❌ |

*Connector-dependent. Verify with your specific payment processor.

### 3. Connector Configuration

Ensure your payment processor (Stripe, Adyen, etc.) supports manual capture:

1. Go to **Dashboard → Processors**
2. Select your connector
3. Verify "Manual Capture" is enabled in settings
4. Check authorization hold duration (typically 7-30 days)

### 4. Webhook Endpoint (Recommended)

Set up a webhook endpoint to receive real-time status updates:

```
POST https://your-domain.com/webhooks/hyperswitch
```

Configure in Dashboard: **Developer → Payment Settings → Webhooks**

---

## How It Works

### Payment State Flow

1. **requires_customer_action** - Payment created, awaiting card details
2. **requires_confirmation** - Card details collected, ready to authorize
3. **processing** - Authorization in progress (3DS verification)
4. **requires_capture** - ✅ Authorized, funds held, ready to capture
5. **partially_captured** - Partial amount captured, remainder voided
6. **partially_captured_and_capturable** - Partial captured, more can be captured
7. **succeeded** - Fully captured, funds settled
8. **cancelled** - Authorization voided, hold released

### Key Timestamps & Hold Durations

| Card Network | Standard Hold Duration | Extended Available |
|--------------|------------------------|-------------------|
| **Visa** | 7 days | Up to 30 days |
| **Mastercard** | 7 days | Up to 30 days |
| **American Express** | 7 days | Up to 30 days |
| **Discover** | 10 days | Up to 30 days |

**⚠️ Authorization holds automatically expire after the hold period. Ensure you capture before expiration, or re-authorize the payment.**

---

## Implementation

### Step 1: Create Payment

Create a payment with `capture_method: "manual"` from your server.

**cURL:**

```bash
curl --location 'https://sandbox.hyperswitch.io/payments' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: <your_api_key>' \
--data '{
    "amount": 6540,
    "currency": "USD",
    "confirm": false,
    "capture_method": "manual",
    "authentication_type": "no_three_ds",
    "return_url": "https://your-site.com/return",
    "customer_id": "cust_123456",
    "description": "Hotel booking - Room 302",
    "billing": {
        "address": {
            "line1": "1467 Harrison Street",
            "city": "San Francisco",
            "state": "California",
            "zip": "94122",
            "country": "US",
            "first_name": "John",
            "last_name": "Doe"
        },
        "email": "john.doe@example.com"
    },
    "metadata": {
        "order_id": "ORD-2024-001",
        "hotel_id": "HTL-SF-001",
        "check_in": "2024-03-15"
    }
}'
```

**Node.js:**

```javascript
const Hyperswitch = require('@juspay-tech/hyperswitch-node');
const hyperswitch = new Hyperswitch('<your_api_key>');

async function createManualCapturePayment() {
  const payment = await hyperswitch.paymentIntents.create({
    amount: 6540,
    currency: 'USD',
    confirm: false,
    capture_method: 'manual',
    authentication_type: 'no_three_ds',
    return_url: 'https://your-site.com/return',
    customer_id: 'cust_123456',
    description: 'Hotel booking - Room 302',
    billing: {
      address: {
        line1: '1467 Harrison Street',
        city: 'San Francisco',
        state: 'California',
        zip: '94122',
        country: 'US',
        first_name: 'John',
        last_name: 'Doe'
      },
      email: 'john.doe@example.com'
    },
    metadata: {
      order_id: 'ORD-2024-001',
      hotel_id: 'HTL-SF-001',
      check_in: '2024-03-15'
    }
  });

  console.log('Payment created:', payment.payment_id);
  console.log('Client secret:', payment.client_secret);
  return payment;
}
```

**Python:**

```python
import requests

def create_manual_capture_payment():
    url = "https://sandbox.hyperswitch.io/payments"
    
    headers = {
        "Content-Type": "application/json",
        "Accept": "application/json",
        "api-key": "<your_api_key>"
    }
    
    payload = {
        "amount": 6540,
        "currency": "USD",
        "confirm": False,
        "capture_method": "manual",
        "authentication_type": "no_three_ds",
        "return_url": "https://your-site.com/return",
        "customer_id": "cust_123456",
        "description": "Hotel booking - Room 302",
        "billing": {
            "address": {
                "line1": "1467 Harrison Street",
                "city": "San Francisco",
                "state": "California",
                "zip": "94122",
                "country": "US",
                "first_name": "John",
                "last_name": "Doe"
            },
            "email": "john.doe@example.com"
        },
        "metadata": {
            "order_id": "ORD-2024-001",
            "hotel_id": "HTL-SF-001",
            "check_in": "2024-03-15"
        }
    }
    
    response = requests.post(url, headers=headers, json=payload)
    response.raise_for_status()
    payment = response.json()
    
    print(f"Payment created: {payment['payment_id']}")
    print(f"Client secret: {payment['client_secret']}")
    return payment
```

**Expected Response (Success):**

```json
{
  "payment_id": "pay_At7O43TJJZyP7OmrcdQD",
  "client_secret": "pay_At7O43TJJZyP7OmrcdQD_secret_c2tuaWQ",
  "status": "requires_customer_action",
  "amount": 6540,
  "currency": "USD",
  "capture_method": "manual",
  "amount_capturable": 0,
  "created": "2024-03-08T03:30:00.000Z",
  "expires_on": "2024-03-15T03:30:00.000Z"
}
```

**Error Response Example:**

```json
{
  "error": {
    "type": "invalid_request",
    "code": "IR_01",
    "message": "Invalid api key provided"
  }
}
```

---

### Step 2: Confirm (Authorize)

#### Option A: Frontend with Hyperswitch.js (Recommended)

**JavaScript (Browser):**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Manual Capture Checkout</title>
  <script src="https://checkout.hyperswitch.io/v1/hyper.js"></script>
</head>
<body>
  <form id="payment-form">
    <div id="payment-element"></div>
    <button id="submit-button">Authorize Payment</button>
    <div id="error-message"></div>
  </form>

  <script>
    const hyper = Hyper('<your_publishable_key>');
    const widgets = hyper.widgets({ appearance: { theme: 'default' }});

    const paymentElement = widgets.create('payment', {
      clientSecret: '<client_secret_from_step_1>'
    });
    paymentElement.mount('#payment-element');

    document.getElementById('payment-form').addEventListener('submit', async (e) => {
      e.preventDefault();
      
      const { error, paymentIntent } = await hyper.confirmPayment({
        widgets,
        confirmParams: { return_url: 'https://your-site.com/return' },
      });

      if (error) {
        document.getElementById('error-message').textContent = error.message;
      } else {
        console.log('Payment authorized:', paymentIntent.id);
        console.log('Status:', paymentIntent.status); // 'requires_capture'
      }
    });
  </script>
</body>
</html>
```

**React:**

```jsx
import { HyperElements, PaymentElement, useHyper } from '@juspay-tech/react-hyper-js';
import { loadHyper } from '@juspay-tech/hyper-js';
import { useState } from 'react';

const hyperPromise = loadHyper('<your_publishable_key>');

function CheckoutForm({ clientSecret }) {
  const hyper = useHyper();
  const widgets = hyper.widgets();
  const [message, setMessage] = useState(null);
  const [isProcessing, setIsProcessing] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsProcessing(true);

    const { error, paymentIntent } = await hyper.confirmPayment({
      widgets,
      confirmParams: { return_url: 'https://your-site.com/return' },
    });

    if (error) {
      setMessage(error.message);
    } else {
      setMessage('Payment authorized! Funds are now on hold.');
    }
    setIsProcessing(false);
  };

  return (
    <form onSubmit={handleSubmit}>
      <PaymentElement />
      <button disabled={isProcessing}>
        {isProcessing ? 'Authorizing...' : 'Authorize Payment'}
      </button>
      {message && <div>{message}</div>}
    </form>
  );
}

function App() {
  return (
    <HyperElements hyper={hyperPromise} options={{ clientSecret: '<client_secret>' }}>
      <CheckoutForm />
    </HyperElements>
  );
}
```

#### Option B: Direct API Confirmation (Requires PCI Compliance)

```bash
curl --location 'https://sandbox.hyperswitch.io/payments/{payment_id}/confirm' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: <your_publishable_key>' \
--data '{
    "payment_method": "card",
    "client_secret": "<client_secret_from_step_1>",
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

**Expected Response:**

```json
{
  "payment_id": "pay_At7O43TJJZyP7OmrcdQD",
  "status": "requires_capture",
  "amount": 6540,
  "amount_capturable": 6540,
  "currency": "USD",
  "capture_method": "manual"
}
```

---

### Step 3: Capture Funds

**cURL:**

```bash
curl --location 'https://sandbox.hyperswitch.io/payments/{payment_id}/capture' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: <your_api_key>' \
--data '{
    "amount_to_capture": 6540,
    "statement_descriptor_name": "Your Merchant Name",
    "statement_descriptor_suffix": "Room 302"
}'
```

**Node.js:**

```javascript
async function capturePayment(paymentId) {
  const capture = await hyperswitch.paymentIntents.capture(
    paymentId,
    {
      amount_to_capture: 6540,
      statement_descriptor_name: "Your Merchant Name",
      statement_descriptor_suffix: "Room 302"
    }
  );

  console.log('Payment captured:', capture.payment_id);
  console.log('Status:', capture.status);
  return capture;
}
```

**Python:**

```python
def capture_payment(payment_id):
    url = f"https://sandbox.hyperswitch.io/payments/{payment_id}/capture"
    
    headers = {
        "Content-Type": "application/json",
        "Accept": "application/json",
        "api-key": "<your_api_key>"
    }
    
    payload = {
        "amount_to_capture": 6540,
        "statement_descriptor_name": "Your Merchant Name",
        "statement_descriptor_suffix": "Room 302"
    }
    
    response = requests.post(url, headers=headers, json=payload)
    response.raise_for_status()
    capture = response.json()
    
    print(f"Payment captured: {capture['payment_id']}")
    print(f"Status: {capture['status']}")
    return capture
```

**Expected Response (Success):**

```json
{
  "payment_id": "pay_At7O43TJJZyP7OmrcdQD",
  "status": "succeeded",
  "amount": 6540,
  "amount_captured": 6540,
  "amount_capturable": 0,
  "currency": "USD",
  "capture_method": "manual"
}
```

---

## Capture Types

### Full Capture

Capture the entire authorized amount in a single request.

```json
{
  "amount_to_capture": 6540
}
```

**Status Transition:** `requires_capture` → `succeeded`

**Use Case:** Standard single-delivery orders

### Partial Capture

Capture less than the authorized amount. The remaining amount is automatically voided.

```json
{
  "amount_to_capture": 5000
}
```

**Status Transition:** `requires_capture` → `partially_captured`

**Use Case:** Order adjustments, partial shipments, early checkouts

### Multiple Partial Captures

Some connectors support capturing funds in multiple installments.

```json
// First capture
{ "amount_to_capture": 3000 }

// Second capture
{ "amount_to_capture": 2000 }
```

**Status Transition:** `requires_capture` → `partially_captured_and_capturable` → `succeeded`

**Use Case:** Subscription boxes with monthly deliveries, phased project payments

### Over Capture

Capture more than the originally authorized amount (requires enabling).

```bash
# Enable at payment creation
curl --location 'https://sandbox.hyperswitch.io/payments' \
--header 'api-key: <your_api_key>' \
--data '{
    "amount": 10000,
    "currency": "USD",
    "capture_method": "manual",
    "enable_overcapture": true
}'
```

Then capture:
```json
{
  "amount_to_capture": 11500
}
```

**Use Case:** Hotels (incidentals, room service), shipping adjustments, tips/gratuities

**Learn more:** [Over Capture Documentation](https://docs.hyperswitch.io/about-hyperswitch/payment-suite-1/payments-cards/manual-capture/overcapture)

---

## Webhook Events

Configure webhooks to receive real-time updates on payment status changes.

### Key Events for Manual Capture

| Event | Description | When Triggered |
|-------|-------------|----------------|
| `payment_authorized` | Authorization successful | Payment confirmed, funds held |
| `payment_captured` | Capture successful | Funds settled after capture |
| `payment_succeeded` | Payment complete | Full capture processed |
| `payment_cancelled` | Authorization voided | Payment cancelled before capture |

### Webhook Payload Example (payment_captured)

```json
{
  "event_id": "evt_1234567890abcdef",
  "event_type": "payment_captured",
  "created": "2024-03-08T10:30:00.000Z",
  "data": {
    "payment_id": "pay_At7O43TJJZyP7OmrcdQD",
    "status": "succeeded",
    "amount": 6540,
    "amount_captured": 6540,
    "currency": "USD",
    "customer_id": "cust_123456",
    "capture_method": "manual",
    "connector": "stripe",
    "metadata": {
      "order_id": "ORD-2024-001",
      "hotel_id": "HTL-SF-001"
    }
  }
}
```

### Webhook Handler Example (Node.js)

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const expectedSignature = crypto
    .createHmac('sha512', secret)
    .update(payload)
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

app.post('/webhooks/hyperswitch', express.raw({ type: 'application/json' }), (req, res) => {
  const signature = req.headers['x-webhook-signature-512'];
  const payload = req.body;
  
  if (!verifyWebhookSignature(payload, signature, process.env.WEBHOOK_SECRET)) {
    return res.status(400).send('Invalid signature');
  }
  
  const event = JSON.parse(payload);
  
  switch (event.event_type) {
    case 'payment_authorized':
      console.log('Payment authorized:', event.data.payment_id);
      // Update order status to "authorized"
      break;
      
    case 'payment_captured':
      console.log('Payment captured:', event.data.payment_id);
      // Fulfill order, send confirmation
      break;
      
    case 'payment_cancelled':
      console.log('Payment cancelled:', event.data.payment_id);
      // Release inventory, notify customer
      break;
  }
  
  res.status(200).send('Received');
});
```

---

## Error Handling

### Common Errors

| Error Code | Description | Resolution |
|------------|-------------|------------|
| `IR_01` | Invalid API key | Check API key, ensure using correct environment |
| `IR_02` | Payment not found | Verify payment_id is correct |
| `IR_03` | Payment already captured | Cannot capture same payment twice |
| `IR_04` | Capture amount exceeds capturable amount | Verify amount_to_capture <= amount_capturable |
| `IR_05` | Payment not in capturable state | Payment must be in 'requires_capture' status |
| `IR_06` | Authorization expired | Re-authorize the payment |
| `CE_01` | Connector error | Check connector settings and retry |

### HTTP Status Codes

| Status Code | Meaning | Action |
|-------------|---------|--------|
| `200` | Success | Process response |
| `400` | Bad Request | Check request payload |
| `401` | Unauthorized | Verify API key |
| `404` | Not Found | Check resource ID |
| `409` | Conflict | Payment in wrong state |
| `422` | Unprocessable Entity | Validation failed |
| `500` | Server Error | Retry with exponential backoff |

### Error Response Example

```json
{
  "error": {
    "type": "invalid_request",
    "code": "IR_04",
    "message": "Capture amount (7000) exceeds amount capturable (6540)",
    "param": "amount_to_capture"
  }
}
```

### Retry Strategy

```python
import time
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

def create_retry_session(
    retries=3,
    backoff_factor=0.3,
    status_forcelist=(500, 502, 503, 504)
):
    session = requests.Session()
    retry = Retry(
        total=retries,
        read=retries,
        connect=retries,
        backoff_factor=backoff_factor,
        status_forcelist=status_forcelist,
    )
    adapter = HTTPAdapter(max_retries=retry)
    session.mount('https://', adapter)
    return session

# Use retry session for API calls
session = create_retry_session()
response = session.post(url, headers=headers, json=payload)
```

---

## Testing

### Sandbox Environment

Use the Hyperswitch Sandbox to test Manual Capture without real charges:

**Base URL:** `https://sandbox.hyperswitch.io`

**Dashboard:** [https://app.hyperswitch.io](https://app.hyperswitch.io)

### Test Card Numbers

| Card Type | Number | CVC | Expiry | 3DS | Result |
|-----------|--------|-----|--------|-----|--------|
| **Visa** | `4242424242424242` | Any 3 digits | Any future | No | Success |
| **Visa (3DS)** | `4000002500003155` | Any 3 digits | Any future | Yes | Success |
| **Mastercard** | `5555555555554444` | Any 3 digits | Any future | No | Success |
| **Amex** | `378282246310005` | Any 4 digits | Any future | No | Success |
| **Declined** | `4000000000000002` | Any 3 digits | Any future | No | Decline |
| **Insufficient Funds** | `4000000000009995` | Any 3 digits | Any future | No | Decline |

### Test Scenarios

#### Scenario 1: Full Capture Flow

```bash
# 1. Create payment
curl -X POST https://sandbox.hyperswitch.io/payments \
  -H "api-key: $API_KEY" \
  -d '{"amount":10000,"currency":"USD","capture_method":"manual"}'

# 2. Confirm with test card
curl -X POST https://sandbox.hyperswitch.io/payments/{id}/confirm \
  -H "api-key: $PUBLISHABLE_KEY" \
  -d '{"payment_method":"card","client_secret":"...","payment_method_data":{"card":{"card_number":"4242424242424242","card_exp_month":"12","card_exp_year":"2027","card_cvc":"123"}}}'

# 3. Capture full amount
curl -X POST https://sandbox.hyperswitch.io/payments/{id}/capture \
  -H "api-key: $API_KEY" \
  -d '{"amount_to_capture":10000}'
```

#### Scenario 2: Partial Capture

```bash
# Capture partial amount
curl -X POST https://sandbox.hyperswitch.io/payments/{id}/capture \
  -H "api-key: $API_KEY" \
  -d '{"amount_to_capture":7500}'

# Status becomes 'partially_captured'
```

#### Scenario 3: Cancel Authorization

```bash
# Cancel before capture
curl -X POST https://sandbox.hyperswitch.io/payments/{id}/cancel \
  -H "api-key: $API_KEY"

# Status becomes 'cancelled', funds released
```

---

## Next Steps

### Related Documentation

* [API Reference - Create Payment](https://api-reference.hyperswitch.io/v1/payments/payments--create)
* [API Reference - Confirm Payment](https://api-reference.hyperswitch.io/v1/payments/payments--confirm)
* [API Reference - Capture Payment](https://api-reference.hyperswitch.io/v1/payments/payments--capture)
* [Over Capture Documentation](https://docs.hyperswitch.io/about-hyperswitch/payment-suite-1/payments-cards/manual-capture/overcapture)
* [Webhooks Guide](https://docs.hyperswitch.io/explore-hyperswitch/payment-orchestration/quickstart/webhooks)
* [Unified Checkout Integration](https://docs.hyperswitch.io/hyperswitch-cloud/integration-guide/web)

### Get Help

* **Technical Support:** [support@hyperswitch.io](mailto:support@hyperswitch.io)
* **Community Discord:** [Join our community](https://discord.gg/hyperswitch)
* **GitHub Issues:** [Report bugs](https://github.com/juspay/hyperswitch/issues)

### Ready to go live?

[Contact Sales →](https://hyperswitch.io/contact) to discuss your requirements and get production access.

---

## Glossary

| Term | Definition |
|------|------------|
| **Authorization (Auth)** | The process of verifying that a customer has sufficient funds and placing a hold on those funds |
| **Capture** | The process of settling the authorized funds to the merchant's account |
| **Capture Method** | The strategy for capturing funds: `automatic` (immediate) or `manual` (deferred) |
| **Client Secret** | A secure token used by the frontend to confirm payments |
| **Hold Duration** | The time period during which authorized funds remain blocked |
| **Over Capture** | Capturing an amount greater than the originally authorized amount |
| **Partial Capture** | Capturing less than the full authorized amount |
| **Requires Capture** | Payment status indicating successful authorization awaiting capture |
| **Void/Cancel** | Releasing the held funds without capturing them |
| **Webhook** | HTTP callback for receiving real-time event notifications |

---

## Validation Rules

### Create Payment

| Field | Required | Validation |
|-------|----------|------------|
| `amount` | ✅ | Integer > 0, in smallest currency unit (cents) |
| `currency` | ✅ | 3-letter ISO currency code (e.g., "USD") |
| `capture_method` | ✅ | Must be "manual" for Manual Capture |
| `confirm` | ❌ | Set to `false` for separate auth flow |
| `customer_id` | ❌ | Max 64 characters |
| `metadata` | ❌ | Max 15 keys, 255 chars per value |

### Capture Payment

| Field | Required | Validation |
|-------|----------|------------|
| `amount_to_capture` | ✅ | Integer > 0 and <= amount_capturable |
| `statement_descriptor_name` | ❌ | Max 22 characters |
| `statement_descriptor_suffix` | ❌ | Max 22 characters |

---

*Last updated: March 2024*
