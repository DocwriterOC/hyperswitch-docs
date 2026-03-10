---
description: PayPal integration guide for Juspay hyperswitch - covers setup, configuration, webhooks, payment flow, and error handling
icon: paypal
---

# PayPal Integration Guide

## TL;DR

Integrate PayPal with Juspay hyperswitch to let customers pay using their PayPal balance, linked bank accounts, or saved cards—without entering card details on your site. This guide covers: PayPal Developer setup, Juspay hyperswitch connector configuration, webhook setup with signature verification, payment flow implementation, server-side verification, and handling PayPal-specific error codes.

**Quick Start:**
1. Create a PayPal app in the Developer Dashboard → get Client ID and Secret
2. Configure the PayPal connector in Juspay hyperswitch Control Centre
3. Set up webhooks in PayPal (for real-time updates) and in Juspay hyperswitch (to receive them)
4. Implement the redirect flow and **always verify payment status server-side**
5. Handle webhook events: `CHECKOUT.ORDER.APPROVED`, `PAYMENT.CAPTURE.COMPLETED`

---

## Prerequisites

Before you begin, ensure you have:

| Requirement | Details |
|-------------|---------|
| **PayPal Business Account** | Active business account (not personal) |
| **PayPal Developer Access** | Access to [PayPal Developer Dashboard](https://developer.paypal.com/) |
| **Juspay hyperswitch Account** | Configured account with API access |
| **HTTPS Endpoints** | Valid `return_url` and `cancel_url` (required for PayPal security) |

---

## Configuration

### Step 1: Create a PayPal App

1. Log in to the [PayPal Developer Dashboard](https://developer.paypal.com/)
2. Navigate to **My Apps & Credentials**
3. Click **Create App**
4. Enter an app name (e.g., "Juspay hyperswitch Integration")
5. Select the **Merchant** app type
6. Click **Create App**
7. Copy the **Client ID** and **Secret** (you'll need the Secret only once—store it securely)
8. Note the environment: **Sandbox** for testing, **Live** for production

**Important:** Treat the Secret like a password. Never expose it in client-side code.

### Step 2: Configure PayPal in Juspay hyperswitch

1. Log in to [Juspay hyperswitch Control Centre](https://app.hyperswitch.io/)
2. Go to **Connectors** → **Add Connector**
3. Select **PayPal**
4. Enter the credentials:
   - **Client ID**: From your PayPal app
   - **Client Secret**: From your PayPal app
   - **Environment**: Sandbox or Production
5. Enable desired payment methods (PayPal Wallet)
6. Click **Activate**

---

## Webhook Setup

Webhooks ensure your system receives real-time updates about payment status changes. You must configure webhooks in **both** PayPal and Juspay hyperswitch.

### Configure Webhooks in PayPal

1. In the [PayPal Developer Dashboard](https://developer.paypal.com/), go to **My Apps & Credentials**
2. Select your app
3. Scroll to **Webhooks** and click **Add Webhook**
4. Enter your Juspay hyperswitch webhook URL (provided in Juspay hyperswitch Control Centre under **Developers** → **Webhooks**)
   - Format: `https://webhooks.hyperswitch.io/webhooks/{your-merchant-id}/paypal`
5. Select the following event types:
   - `CHECKOUT.ORDER.APPROVED` — Customer approved the payment in PayPal
   - `PAYMENT.CAPTURE.COMPLETED` — Payment successfully captured
   - `PAYMENT.CAPTURE.DENIED` — Payment capture failed
   - `CHECKOUT.ORDER.COMPLETED` — Order completed (alternative confirmation)
6. Click **Save**

### Configure Webhooks in Juspay hyperswitch

1. In Juspay hyperswitch Control Centre, go to **Developers** → **Webhooks**
2. Click **Add Endpoint**
3. Enter your server's webhook URL (where Juspay hyperswitch will send events)
4. Select events to subscribe to:
   - `payment_intent.succeeded`
   - `payment_intent.failed`
   - `refund.succeeded`
5. Copy the **Webhook Signing Secret** — you'll need it for signature verification
6. Click **Add Endpoint**

### Webhook Signature Verification

**Always verify webhook signatures** to ensure events come from Juspay hyperswitch and not malicious actors.

**Example: Verify Webhook Signature (Node.js)**

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature, 'hex'),
    Buffer.from(expectedSignature, 'hex')
  );
}

// In your webhook handler
app.post('/webhooks/hyperswitch', (req, res) => {
  const signature = req.headers['x-webhook-signature'];
  const payload = JSON.stringify(req.body);
  
  if (!verifyWebhookSignature(payload, signature, WEBHOOK_SECRET)) {
    return res.status(401).send('Invalid signature');
  }
  
  // Process the webhook event
  const event = req.body;
  handleWebhookEvent(event);
  res.status(200).send('OK');
});
```

**Example: Verify Webhook Signature (Python)**

```python
import hmac
import hashlib

def verify_webhook_signature(payload: str, signature: str, secret: str) -> bool:
    expected_signature = hmac.new(
        secret.encode('utf-8'),
        payload.encode('utf-8'),
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(signature, expected_signature)

# In your webhook handler
@app.route('/webhooks/hyperswitch', methods=['POST'])
def handle_webhook():
    signature = request.headers.get('X-Webhook-Signature')
    payload = request.get_data(as_text=True)
    
    if not verify_webhook_signature(payload, signature, WEBHOOK_SECRET):
        return 'Invalid signature', 401
    
    event = request.json
    handle_webhook_event(event)
    return 'OK', 200
```

---

## Payment Flow

### Customer Experience

1. Customer selects **PayPal** at checkout
2. Your backend creates a payment intent with Juspay hyperswitch
3. Customer is redirected to PayPal to log in and confirm payment
4. PayPal redirects customer back to your `return_url`
5. **Your server verifies the payment status** (never trust the redirect URL alone)
6. Webhook confirms the final payment state

### Create Payment Intent

```bash
curl -X POST https://api.hyperswitch.io/v1/payments \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{
    "amount": 10000,
    "currency": "USD",
    "customer_id": "cust_12345",
    "payment_method": "wallet",
    "payment_method_type": "paypal",
    "return_url": "https://your-site.com/payment/success",
    "cancel_url": "https://your-site.com/payment/cancel"
  }'
```

**Response:**

```json
{
  "payment_id": "pay_abcdefghijklmnopqrstuvwxyz",
  "status": "requires_customer_action",
  "next_action": {
    "type": "redirect_to_url",
    "url": "https://www.paypal.com/checkoutnow?token=EC_TOKEN"
  }
}
```

Redirect the customer to the `url` provided in `next_action`.

---

## Handling Redirect

After PayPal authentication, the customer is redirected back to your specified URLs with query parameters.

### Redirect URL Format

**Success URL:**
```
https://your-site.com/payment/success?payment_id=pay_abc&status=succeeded
```

**Cancel URL:**
```
https://your-site.com/payment/cancel?payment_id=pay_abc&status=cancelled
```

### All Possible Redirect Status Values

| Status Value | Meaning | Next Step |
|--------------|---------|-----------|
| `succeeded` | Payment completed successfully | Verify server-side, fulfill order |
| `requires_capture` | Authorised but not captured | Capture the payment if not auto-captured |
| `processing` | Payment is being processed | Poll API or wait for webhook |
| `cancelled` | Customer cancelled in PayPal | Offer retry or alternative payment |
| `failed` | Payment attempt failed | Check error code, offer retry |
| `expired` | Payment session expired | Create new payment intent |

**⚠️ CRITICAL: Never trust the client-side status.** The redirect URL status can be spoofed or manipulated. Always verify the payment status server-side.

### Server-Side Verification (Required)

**Always verify payment status using the Juspay hyperswitch API** after the customer returns from PayPal.

**Example: Server-Side Verification (cURL)**

```bash
curl -X GET https://api.hyperswitch.io/v1/payments/pay_abcdefghijklmnopqrstuvwxyz \
  -H "api-key: YOUR_API_KEY"
```

**Example: Server-Side Verification (Node.js)**

```javascript
const axios = require('axios');

async function verifyPayment(paymentId) {
  try {
    const response = await axios.get(
      `https://api.hyperswitch.io/v1/payments/${paymentId}`,
      {
        headers: { 'api-key': HYPERSWITCH_API_KEY }
      }
    );
    
    const payment = response.data;
    
    // Only fulfill the order if status is "succeeded"
    if (payment.status === 'succeeded') {
      await fulfillOrder(payment);
      return { success: true, payment };
    } else {
      return { success: false, status: payment.status };
    }
  } catch (error) {
    console.error('Payment verification failed:', error);
    return { success: false, error: error.message };
  }
}

// In your return_url handler
app.get('/payment/success', async (req, res) => {
  const { payment_id } = req.query;
  
  // ALWAYS verify server-side — never trust query params alone
  const result = await verifyPayment(payment_id);
  
  if (result.success) {
    res.render('payment-success', { payment: result.payment });
  } else {
    res.render('payment-pending', { status: result.status });
  }
});
```

**Example: Server-Side Verification (Python)**

```python
import requests

def verify_payment(payment_id: str) -> dict:
    response = requests.get(
        f'https://api.hyperswitch.io/v1/payments/{payment_id}',
        headers={'api-key': HYPERSWITCH_API_KEY}
    )
    response.raise_for_status()
    
    payment = response.json()
    
    if payment['status'] == 'succeeded':
        fulfill_order(payment)
        return {'success': True, 'payment': payment}
    else:
        return {'success': False, 'status': payment['status']}

# In your return_url handler
@app.route('/payment/success')
def payment_success():
    payment_id = request.args.get('payment_id')
    
    # ALWAYS verify server-side — never trust query params alone
    result = verify_payment(payment_id)
    
    if result['success']:
        return render_template('payment-success.html', payment=result['payment'])
    else:
        return render_template('payment-pending.html', status=result['status'])
```

---

## Payment Status Lifecycle

Understanding the payment status flow helps you handle edge cases and provide better customer experiences.

```
requires_customer_action
         │
         ▼ (customer approves in PayPal)
    processing
         │
    ┌────┴────┐
    ▼         ▼
succeeded   failed
    │
    ▼ (optional)
refunded / partially_refunded
```

### Status Definitions

| Status | Description | Customer Action |
|--------|-------------|-----------------|
| `requires_customer_action` | Awaiting customer approval in PayPal | Customer must complete PayPal flow |
| `processing` | Payment is being processed by PayPal | Wait for webhook or poll API |
| `requires_capture` | Authorised but not yet captured | Capture via API if manual capture enabled |
| `succeeded` | Payment completed successfully | None — fulfill order |
| `failed` | Payment attempt failed | Offer retry or alternative |
| `cancelled` | Customer cancelled the payment | Offer retry |
| `expired` | Payment session timed out | Create new payment intent |
| `refunded` | Full refund processed | None |
| `partially_refunded` | Partial refund processed | None |
| `disputed` | Customer opened a dispute | Handle via PayPal Resolution Center |

### Webhook Event Mapping

| PayPal Webhook Event | Juspay hyperswitch Status | When It Fires |
|----------------------|-------------------|---------------|
| `CHECKOUT.ORDER.APPROVED` | `requires_capture` or `processing` | Customer approved in PayPal |
| `PAYMENT.CAPTURE.COMPLETED` | `succeeded` | Funds captured successfully |
| `PAYMENT.CAPTURE.DENIED` | `failed` | Capture was declined |

---

## Webhook Events Reference

Subscribe to these PayPal webhook events in your PayPal app:

| Event Name | Description | Juspay hyperswitch Status |
|------------|-------------|-------------------|
| `CHECKOUT.ORDER.APPROVED` | Customer approved the payment in PayPal | `requires_capture` / `processing` |
| `PAYMENT.CAPTURE.COMPLETED` | Payment was successfully captured | `succeeded` |
| `PAYMENT.CAPTURE.DENIED` | Payment capture was declined | `failed` |
| `CHECKOUT.ORDER.COMPLETED` | Order reached final completed state | `succeeded` |
| `CHECKOUT.ORDER.CANCELLED` | Customer cancelled the checkout | `cancelled` |

---

## Refunds

Process PayPal refunds via Juspay hyperswitch:

```bash
curl -X POST https://api.hyperswitch.io/v1/refunds \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{
    "payment_id": "pay_abcdefghijklmnopqrstuvwxyz",
    "amount": 10000,
    "reason": "Customer request"
  }'
```

**Refund Response:**

```json
{
  "refund_id": "ref_abcdefghijklmnopqrstuvwxyz",
  "payment_id": "pay_abcdefghijklmnopqrstuvwxyz",
  "status": "pending",
  "amount": 10000,
  "currency": "USD"
}
```

PayPal refunds typically process within 1-5 business days. The customer will receive the refund to their original payment method.

---

## Currency Precision Requirements

PayPal has specific currency precision requirements. Amounts must be specified in the smallest currency unit (cents for USD, etc.).

| Currency | Decimal Places | Example (100.00 USD) |
|----------|---------------|---------------------|
| USD, EUR, GBP | 2 | `10000` (100.00 × 100) |
| JPY, KRW | 0 | `10000` (no decimals) |
| BHD, JOD, KWD, OMR | 3 | `100000` (100.000 × 1000) |

**Important:** Sending the wrong precision will result in errors or incorrect amounts.

### Zero-Decimal Currencies

For zero-decimal currencies, do not multiply:

```json
{
  "amount": 10000,
  "currency": "JPY"
}
// = ¥10,000 (not ¥100.00)
```

### Three-Decimal Currencies

For three-decimal currencies, multiply by 1000:

```json
{
  "amount": 100500,
  "currency": "BHD"
}
// = BD 100.500
```

---

## PayPal-Specific Error Codes

| Error Code | Description | Resolution |
|------------|-------------|------------|
| `PP_ACCOUNT_RESTRICTED` | PayPal account has restrictions | Contact PayPal support |
| `PP_AMOUNT_EXCEEDED` | Transaction amount exceeds limits | Reduce amount or contact PayPal |
| `PP_CURRENCY_NOT_SUPPORTED` | Currency not supported for this account | Use a supported currency |
| `PP_INVALID_CLIENT_CREDENTIALS` | Client ID or Secret is incorrect | Verify credentials in Juspay hyperswitch |
| `PP_ORDER_ALREADY_CAPTURED` | Payment already captured | Check payment status; do not double-capture |
| `PP_ORDER_EXPIRED` | PayPal order expired | Create a new payment intent |
| `PP_PAYEE_ACCOUNT_INVALID` | Merchant account issue | Verify PayPal account setup |
| `PP_PAYMENT_NOT_APPROVED` | Customer did not approve payment | Prompt customer to complete PayPal flow |
| `PP_TRANSACTION_LIMIT_EXCEEDED` | Daily/transaction limit reached | Wait or contact PayPal to increase limits |
| `PP_WEBHOOK_VERIFICATION_FAILED` | Webhook signature invalid | Check webhook secret configuration |

### Common HTTP Status Codes

| Status | Meaning | Action |
|--------|---------|--------|
| `400` | Bad Request | Check request payload format |
| `401` | Unauthorized | Verify API key |
| `402` | Payment Required | Payment failed — check error code |
| `404` | Not Found | Payment ID doesn't exist |
| `409` | Conflict | Payment in conflicting state |
| `422` | Unprocessable | Validation error — check error message |
| `429` | Rate Limited | Slow down requests |
| `500` | Server Error | Retry with exponential backoff |

---

## Testing

### PayPal Sandbox Testing

1. Create Sandbox accounts in [PayPal Developer Dashboard](https://developer.paypal.com/dashboard/accounts)
   - Create at least one **Business** account (as merchant)
   - Create at least one **Personal** account (as buyer)
2. Use Sandbox credentials in Juspay hyperswitch connector configuration
3. Test the complete flow using [PayPal Sandbox buyer accounts](https://developer.paypal.com/dashboard/accounts)

### Test Scenarios

| Scenario | Expected Result |
|----------|-----------------|
| Successful payment | Status: `succeeded`, webhook received |
| Customer cancels | Status: `cancelled`, redirect to cancel_url |
| Insufficient funds | Status: `failed`, appropriate error code |
| Session timeout | Status: `expired`, create new intent |
| Webhook retry | Juspay hyperswitch retries failed webhooks automatically |

### Sandbox vs Live

| Environment | API Endpoint | PayPal URL |
|-------------|--------------|------------|
| Sandbox | `https://api.hyperswitch.io/v1` | `https://www.sandbox.paypal.com` |
| Live | `https://api.hyperswitch.io/v1` | `https://www.paypal.com` |

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Redirect fails | `return_url` not HTTPS | Use valid HTTPS URLs with SSL certificate |
| Payment stuck in `processing` | Webhook not configured | Set up webhooks in PayPal and Juspay hyperswitch |
| Authentication errors | Invalid Client ID/Secret | Verify credentials match the environment (Sandbox vs Live) |
| Webhook not received | URL incorrect or blocked | Verify webhook URL is publicly accessible |
| Signature verification fails | Wrong webhook secret | Copy the correct signing secret from Juspay hyperswitch |
| Currency error | Wrong precision | Use correct decimal places for the currency |
| Duplicate payments | Double-click on PayPal button | Implement idempotency with `payment_id` |

### Debug Checklist

- [ ] PayPal app is created with correct type (Merchant)
- [ ] Credentials match the environment (Sandbox vs Live)
- [ ] `return_url` and `cancel_url` use HTTPS
- [ ] Webhooks configured in both PayPal and Juspay hyperswitch
- [ ] Webhook URL is publicly accessible (not localhost)
- [ ] Webhook signature verification implemented
- [ ] Server-side payment verification implemented
- [ ] Error handling covers all PayPal-specific codes

---

## Next Steps

- [Configure additional webhooks](../workflows/webhooks.md)
- [Set up other wallet payment methods](./README.md)
- [Implement 3D Secure for card payments](../payment-experience/payment/web/error-codes.md)
- [Set up sandbox environment for testing](../../../setup-hyperswitch-locally/)
- [Review payment retry strategies](../payment-flows-and-management/)

---

## Quick Reference

### Essential cURL Commands

**Create Payment:**
```bash
curl -X POST https://api.hyperswitch.io/v1/payments \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{"amount":10000,"currency":"USD","payment_method":"wallet","payment_method_type":"paypal","return_url":"https://yoursite.com/success","cancel_url":"https://yoursite.com/cancel"}'
```

**Verify Payment:**
```bash
curl -X GET https://api.hyperswitch.io/v1/payments/{payment_id} \
  -H "api-key: YOUR_API_KEY"
```

**Process Refund:**
```bash
curl -X POST https://api.hyperswitch.io/v1/refunds \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{"payment_id":"{payment_id}","amount":10000}'
```

### Key PayPal Webhook Events

| Event | When It Fires |
|-------|---------------|
| `CHECKOUT.ORDER.APPROVED` | Customer approved payment |
| `PAYMENT.CAPTURE.COMPLETED` | Payment captured |
| `PAYMENT.CAPTURE.DENIED` | Capture failed |

### Remember

1. **Always verify payment status server-side** — never trust redirect URL parameters
2. **Verify webhook signatures** to prevent spoofing
3. **Use correct currency precision** — 2 decimals for USD/EUR, 0 for JPY, 3 for BHD
4. **Test in Sandbox** before going live
5. **Handle all error codes** gracefully
