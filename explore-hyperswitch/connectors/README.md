---
description: >-
  Connectors are the bridge between Juspay hyperswitch and external payment services. They let you integrate multiple payment processors (PSPs, acquirers, APMs) through a single unified API—add credentials, configure methods, and start transacting without rewriting your integration code. This guide covers setup, authentication, webhooks, error handling, and troubleshooting.
icon: plug
---

# Connectors Integration

## TL;DR

Connectors are the bridge between Juspay hyperswitch and external payment services. They let you integrate multiple payment processors (PSPs, acquirers, APMs) through a single unified API—add credentials, configure methods, and start transacting without rewriting your integration code. This guide covers setup, authentication, webhooks, error handling, and troubleshooting.

---

## Overview

Connectors are integrations that allow Juspay hyperswitch to communicate with external payment services such as PSPs, acquirers, APMs, card vaults, 3DS authentications, fraud management systems, subscription services, and payout providers. They act as bridges between your Juspay hyperswitch setup and the third-party services that move or manage money for your business.

Every provider has its own APIs, authentication methods, and feature sets. Juspay hyperswitch standardises these differences through connectors, exposing a single unified Payments API. This means you can add, switch, or remove processors without rewriting your code—just plug in credentials and start transacting.

Connectors form the foundation of Juspay hyperswitch's payment orchestration layer, enabling you to manage payments, routing, 3DS authentication, fraud checks, and payouts through a single interface.

---

## Why Multiple Processors?

As your business grows, you will need to expand payment offerings with more payment processors. This need might arise due to multiple reasons:

| Reason | Description |
|--------|-------------|
| **Vendor Independence** | Reducing dependency on a single processor and avoiding vendor lock-in |
| **Performance Optimisation** | Introducing a challenger processor for better cost and authorisation rates |
| **Global Reach** | Launching a business in new geographies with a local payment processor |
| **Localised Experience** | Offering local or new payment methods for your customers |
| **Reliability** | Reducing technical downtimes and improving success rates with a fallback |

Integrating and maintaining multiple payment processors and their different versions is a time and resource intensive process. Juspay hyperswitch can add a new PSP in approximately 2 weeks, allowing you to focus on your core business activities.

---

## Adding a Connector

Most connector integrations follow a simple click-and-connect flow on Juspay hyperswitch using your connector credentials. However, some connectors may require additional setup details as required on the Control Centre.

### Standard Setup Steps

1. **PSP Registration**: You need to be registered with the PSP in order to proceed. If you aren't, you can quickly set up your account by signing up on their dashboard.

2. **Platform Access**: You should have registered on the Juspay hyperswitch Control Centre.

3. **Credential Mapping**: Add the PSP authentication credentials from their dashboard into the Juspay hyperswitch Control Centre.

4. **Method Configuration**: Choose the payment methods you want to utilise with the connector.

5. **Activation**: Enable the PSP once you're done.

---

## Authentication

Authentication credentials vary across different PSPs. Understanding these requirements is essential for successful connector configuration.

### Common Authentication Patterns

| Pattern | Description | Example Providers |
|---------|-------------|-------------------|
| **API Key** | Single secret key for authentication | Airwallex, Adyen (partial) |
| **API Key + Secret** | Key and secret pair for enhanced security | Fiserv, PayU |
| **Client Credentials** | OAuth-style client ID and secret | PayPal, Airwallex |
| **Multi-field** | Merchant ID, terminal IDs, and keys | Braintree, Cybersource, Fiserv |

### Authentication Credentials by Provider

| Provider | Required Credentials | Environment Variables |
|----------|---------------------|----------------------|
| Authorize.net | API Login ID and Transaction Key | `AUTHORIZE_NET_API_LOGIN_ID`, `AUTHORIZE_NET_TRANSACTION_KEY` |
| Adyen | API key and Account ID | `ADYEN_API_KEY`, `ADYEN_ACCOUNT_ID` |
| Braintree | Merchant ID, Public key, and Private key | `BRAINTREE_MERCHANT_ID`, `BRAINTREE_PUBLIC_KEY`, `BRAINTREE_PRIVATE_KEY` |
| Airwallex | API key and Client ID | `AIRWALLEX_API_KEY`, `AIRWALLEX_CLIENT_ID` |
| Fiserv | API Key, API Secret, Merchant ID, and Terminal ID | `FISERV_API_KEY`, `FISERV_API_SECRET`, `FISERV_MERCHANT_ID`, `FISERV_TERMINAL_ID` |
| PayPal | Client Secret and Client ID | `PAYPAL_CLIENT_ID`, `PAYPAL_CLIENT_SECRET` |
| Stripe | Secret key | `STRIPE_SECRET_KEY` |
| Checkout.com | Secret key and Public key | `CHECKOUT_SECRET_KEY`, `CHECKOUT_PUBLIC_KEY` |

### Storing Credentials Securely

Never hardcode credentials in your application code. Use environment variables or a secure secrets manager:

```bash
# Example .env file
ADYEN_API_KEY=your_adyen_api_key_here
ADYEN_ACCOUNT_ID=your_adyen_account_id
STRIPE_SECRET_KEY=sk_live_your_stripe_secret_key
```

```python
# Python example using environment variables
import os
from dotenv import load_dotenv

load_dotenv()

connector_config = {
    "adyen": {
        "api_key": os.getenv("ADYEN_API_KEY"),
        "account_id": os.getenv("ADYEN_ACCOUNT_ID")
    },
    "stripe": {
        "secret_key": os.getenv("STRIPE_SECRET_KEY")
    }
}
```

---

## Code Examples

### Creating a Payment with a Connector

```bash
# Create a payment using the Juspay hyperswitch API
curl --location 'https://api.juspay.io/hyperswitch/payments' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'api-key: your_api_key_here' \
--data '{
    "amount": 6540,
    "currency": "USD",
    "confirm": true,
    "capture_method": "automatic",
    "capture_on": "2024-01-01T00:00:00Z",
    "customer_id": "cus_123456789",
    "email": "customer@example.com",
    "name": "John Doe",
    "phone": "9999999999",
    "phone_country_code": "+1",
    "description": "Test payment",
    "return_url": "https://example.com/return",
    "payment_method": "card",
    "payment_method_type": "credit",
    "payment_method_data": {
        "card": {
            "card_number": "4111111111111111",
            "card_exp_month": "12",
            "card_exp_year": "2025",
            "card_holder_name": "John Doe",
            "card_cvc": "123"
        }
    },
    "routing": {
        "type": "single",
        "data": "stripe"
    },
    "billing": {
        "address": {
            "line1": "123 Main St",
            "line2": "Apt 4B",
            "city": "New York",
            "state": "NY",
            "zip": "10001",
            "country": "US",
            "first_name": "John",
            "last_name": "Doe"
        }
    }
}'
```

```javascript
// JavaScript/Node.js example
const axios = require('axios');

async function createPayment() {
    const paymentData = {
        amount: 6540,
        currency: 'USD',
        confirm: true,
        capture_method: 'automatic',
        customer_id: 'cus_123456789',
        email: 'customer@example.com',
        payment_method: 'card',
        payment_method_data: {
            card: {
                card_number: '4111111111111111',
                card_exp_month: '12',
                card_exp_year: '2025',
                card_holder_name: 'John Doe',
                card_cvc: '123'
            }
        },
        routing: {
            type: 'single',
            data: 'stripe'
        },
        return_url: 'https://example.com/return'
    };

    try {
        const response = await axios.post(
            'https://api.juspay.io/hyperswitch/payments',
            paymentData,
            {
                headers: {
                    'Content-Type': 'application/json',
                    'Accept': 'application/json',
                    'api-key': process.env.HYPERSWITCH_API_KEY
                }
            }
        );
        console.log('Payment created:', response.data);
        return response.data;
    } catch (error) {
        console.error('Payment failed:', error.response?.data || error.message);
        throw error;
    }
}
```

```python
# Python example
import requests
import os

def create_payment():
    url = "https://api.juspay.io/hyperswitch/payments"
    
    payload = {
        "amount": 6540,
        "currency": "USD",
        "confirm": True,
        "capture_method": "automatic",
        "customer_id": "cus_123456789",
        "email": "customer@example.com",
        "payment_method": "card",
        "payment_method_data": {
            "card": {
                "card_number": "4111111111111111",
                "card_exp_month": "12",
                "card_exp_year": "2025",
                "card_holder_name": "John Doe",
                "card_cvc": "123"
            }
        },
        "routing": {
            "type": "single",
            "data": "stripe"
        },
        "return_url": "https://example.com/return"
    }
    
    headers = {
        "Content-Type": "application/json",
        "Accept": "application/json",
        "api-key": os.getenv("HYPERSWITCH_API_KEY")
    }
    
    try:
        response = requests.post(url, json=payload, headers=headers)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"Payment failed: {e}")
        raise
```

### Listing Available Connectors

```bash
# Get list of configured connectors
curl --location 'https://api.juspay.io/hyperswitch/account/connectors' \
--header 'Accept: application/json' \
--header 'api-key: your_api_key_here'
```

```javascript
// JavaScript example
async function listConnectors() {
    try {
        const response = await axios.get(
            'https://api.juspay.io/hyperswitch/account/connectors',
            {
                headers: {
                    'Accept': 'application/json',
                    'api-key': process.env.HYPERSWITCH_API_KEY
                }
            }
        );
        return response.data;
    } catch (error) {
        console.error('Failed to list connectors:', error);
        throw error;
    }
}
```

---

## Webhook Configuration

Webhooks allow Juspay hyperswitch to notify your application about payment events in real time.

### Setting Up Webhooks

1. **Configure Webhook URL** in the Control Centre:
   - Navigate to **Settings** → **Webhooks**
   - Enter your endpoint URL (must be HTTPS)
   - Select the events you want to receive

2. **Verify Webhook Signatures** to ensure security:

```python
# Python webhook verification example
import hmac
import hashlib

def verify_webhook_signature(payload, signature, secret):
    """
    Verify the webhook signature from Juspay hyperswitch
    
    Args:
        payload: The raw request body
        signature: The X-Webhook-Signature-512 header value
        secret: Your webhook secret from the Control Centre
    
    Returns:
        bool: True if signature is valid
    """
    expected_signature = hmac.new(
        secret.encode('utf-8'),
        payload.encode('utf-8'),
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(expected_signature, signature)

# Flask example
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/webhooks/hyperswitch', methods=['POST'])
def handle_webhook():
    payload = request.get_data(as_text=True)
    signature = request.headers.get('X-Webhook-Signature-512')
    secret = os.getenv('HYPERSWITCH_WEBHOOK_SECRET')
    
    if not verify_webhook_signature(payload, signature, secret):
        return jsonify({"error": "Invalid signature"}), 401
    
    event = request.json
    
    # Handle different event types
    event_type = event.get('event_type')
    
    if event_type == 'payment_succeeded':
        handle_payment_success(event['data'])
    elif event_type == 'payment_failed':
        handle_payment_failure(event['data'])
    elif event_type == 'refund_succeeded':
        handle_refund_success(event['data'])
    
    return jsonify({"status": "received"}), 200
```

```javascript
// Node.js webhook verification example
const crypto = require('crypto');
const express = require('express');

const app = express();
app.use(express.raw({ type: 'application/json' }));

function verifyWebhookSignature(payload, signature, secret) {
    const expectedSignature = crypto
        .createHmac('sha256', secret)
        .update(payload)
        .digest('hex');
    
    return crypto.timingSafeEqual(
        Buffer.from(signature),
        Buffer.from(expectedSignature)
    );
}

app.post('/webhooks/hyperswitch', (req, res) => {
    const signature = req.headers['x-webhook-signature-512'];
    const secret = process.env.HYPERSWITCH_WEBHOOK_SECRET;
    
    if (!verifyWebhookSignature(req.body, signature, secret)) {
        return res.status(401).json({ error: 'Invalid signature' });
    }
    
    const event = JSON.parse(req.body);
    
    switch (event.event_type) {
        case 'payment_succeeded':
            handlePaymentSuccess(event.data);
            break;
        case 'payment_failed':
            handlePaymentFailure(event.data);
            break;
        case 'refund_succeeded':
            handleRefundSuccess(event.data);
            break;
    }
    
    res.json({ status: 'received' });
});
```

### Common Webhook Events

| Event Type | Description |
|------------|-------------|
| `payment_succeeded` | Payment completed successfully |
| `payment_failed` | Payment failed |
| `payment_processing` | Payment is being processed |
| `payment_cancelled` | Payment was cancelled |
| `refund_succeeded` | Refund completed successfully |
| `refund_failed` | Refund failed |
| `dispute_created` | A dispute was raised |

---

## Error Handling

### Common Error Codes

| Error Code | Description | Resolution |
|------------|-------------|------------|
| `CE_00` | External Connector Error (generic) | Check connector logs and verify PSP status |
| `CE_01` | Payment authorization failed | Verify payment details and retry; check PSP account status |
| `CE_02` | Payment authentication failed | 3DS or authentication step failed; prompt customer to retry |
| `CE_03` | Payment capture failed | Authorization succeeded but capture failed; contact support |
| `CE_04` | Invalid card data | Customer should verify card number, expiry, and CVV |
| `CE_05` | Card expired | Customer needs to update card details or use different card |
| `CE_06` | Refund failed | Refund could not be processed; verify refund eligibility |
| `CE_07` | Verification failed | Address or card verification failed; check billing details |
| `CE_08` | Dispute failed | Dispute/chargeback operation failed; contact support |

### Error Handling Examples

```python
# Python error handling example
import requests
from requests.exceptions import HTTPError, Timeout, ConnectionError

def create_payment_with_error_handling():
    try:
        response = requests.post(
            'https://api.juspay.io/hyperswitch/payments',
            json=payment_data,
            headers=headers,
            timeout=30
        )
        response.raise_for_status()
        return response.json()
        
    except HTTPError as e:
        error_data = e.response.json()
        error_code = error_data.get('error_code')
        error_message = error_data.get('error_message')
        
        if error_code == 'CE_01':
            # Payment authorization failed - check credentials and PSP status
            print(f"Authorization failed: {error_message}")
            # Alert ops team, check connector config
        elif error_code == 'CE_03':
            # Payment capture failed - check PSP status
            print(f"Capture failed: {error_message}")
            # Try alternate connector
        else:
            print(f"Payment error {error_code}: {error_message}")
        
        raise
        
    except Timeout:
        print("Request timed out. Consider implementing retry logic.")
        raise
        
    except ConnectionError:
        print("Connection error. Check network connectivity.")
        raise
```

```javascript
// JavaScript error handling example
async function createPaymentWithErrorHandling() {
    try {
        const response = await axios.post(
            'https://api.juspay.io/hyperswitch/payments',
            paymentData,
            { 
                headers,
                timeout: 30000 
            }
        );
        return response.data;
        
    } catch (error) {
        if (error.response) {
            const { error_code, error_message } = error.response.data;
            
            switch (error_code) {
                case 'CE_01':
                    console.error('Authorization failed:', error_message);
                    // Alert ops team
                    break;
                case 'CE_03':
                    console.error('Capture failed:', error_message);
                    // Check PSP status
                    break;
                default:
                    console.error(`Payment error ${error_code}:`, error_message);
            }
        } else if (error.code === 'ECONNABORTED') {
            console.error('Request timeout');
        } else if (error.code === 'ECONNREFUSED') {
            console.error('Connection refused');
        }
        
        throw error;
    }
}
```

---

## Troubleshooting

### Common Issues and Solutions

#### Connector Authentication Failures

**Symptom**: Payments fail with `CE_03` or authentication errors

**Solutions**:
1. Verify credentials are correctly entered in the Control Centre
2. Check for extra spaces in API keys or secrets
3. Ensure you're using the correct environment credentials (sandbox vs production)
4. Verify the connector is enabled and active
5. Check if the PSP account is active and not suspended

#### Payment Method Not Available

**Symptom**: Specific payment methods don't appear or fail

**Solutions**:
1. Confirm the payment method is enabled for the connector in the Control Centre
2. Verify the payment method is supported in the transaction currency
3. Check if the payment method requires additional configuration (e.g., 3DS)
4. Ensure the payment method data is formatted correctly in the API request

#### Webhook Not Receiving Events

**Symptom**: Webhooks aren't being delivered to your endpoint

**Solutions**:
1. Verify the webhook URL is accessible from the internet (use ngrok for local testing)
2. Ensure the URL uses HTTPS (required for production)
3. Check that your endpoint returns a 2xx status code quickly (< 5 seconds)
4. Verify the webhook is configured for the correct events
5. Check webhook logs in the Control Centre for delivery status

#### High Error Rates

**Symptom**: Unusually high failure rates for a connector

**Solutions**:
1. Check connector health in the Control Centre dashboard
2. Review recent error logs for patterns
3. Verify PSP status page for known issues
4. Consider enabling fallback to another connector
5. Check if recent changes to integration may have caused issues

#### Timeout Issues

**Symptom**: Payments timing out with `CE_08`

**Solutions**:
1. Check PSP status for known latency issues
2. Implement retry logic with exponential backoff
3. Consider using asynchronous payment confirmation
4. Enable fallback connectors in your routing configuration
5. Contact PSP support if issues persist

### Debugging Tips

1. **Enable Detailed Logging**: Log full request/response bodies during development
2. **Use Sandbox First**: Always test new connectors in sandbox environment
3. **Check Connector Health Dashboard**: Monitor real-time connector performance
4. **Review Webhook Logs**: Verify event delivery and response times
5. **Test with Different Payment Methods**: Ensure all configured methods work

### Getting Support

If issues persist:

1. Check the [Status Page](https://status.juspay.io) for known outages
2. Review [Documentation](https://docs.juspay.io/hyperswitch) for updates
3. Contact support via the Control Centre with:
   - Payment ID
   - Error code
   - Timestamp
   - Connector name

---

## Connector Types

Juspay hyperswitch supports a wide variety of connectors:

| Category | Description | Examples |
|----------|-------------|----------|
| **Core Payments** | Payment processors, acquirers, and alternative payment methods | Stripe, Adyen, Checkout.com |
| **Platforms** | Payment platforms and payout processors | PayPal, Airwallex |
| **Recurring Billing** | Subscription providers | Chargebee, Recurly |
| **Security & Risk** | Card vaults, 3DS authentications, fraud management | CardinalCommerce, Sift |

---

## Supported Connectors

Juspay hyperswitch supports 100+ processors including:

**Global:**
- Stripe
- Adyen
- Checkout.com
- PayPal
- Worldpay

**Regional:**
- Razorpay (India)
- PayU (Europe/LatAm)
- MercadoPago (LatAm)
- Klarna (Europe)

**Enterprise:**
- Cybersource
- Fiserv
- Chase Paymentech
- First Data

For the complete list, visit the [Integrations Directory](https://docs.juspay.io/hyperswitch/integrations).

---

## Environment Differentiation

Juspay hyperswitch provides separate environments for testing and production.

### Environment Overview

| Environment | URL | Purpose |
|-------------|-----|---------|
| **Sandbox** | `https://sandbox.juspay.io/hyperswitch` | Development and testing |
| **Production** | `https://api.juspay.io/hyperswitch` | Live transactions |

### Best Practices

1. **Never use production credentials in sandbox** or vice versa
2. **Use test card numbers** in sandbox environment
3. **Configure separate webhooks** for each environment
4. **Use environment variables** to switch between environments

```bash
# .env.sandbox
HYPERSWITCH_API_URL=https://sandbox.juspay.io/hyperswitch
HYPERSWITCH_API_KEY=your_sandbox_key
HYPERSWITCH_WEBHOOK_SECRET=your_sandbox_webhook_secret

# .env.production
HYPERSWITCH_API_URL=https://api.juspay.io/hyperswitch
HYPERSWITCH_API_KEY=your_production_key
HYPERSWITCH_WEBHOOK_SECRET=your_production_webhook_secret
```

### Test Card Numbers (Sandbox)

| Card Number | Type | Result |
|-------------|------|--------|
| `4111111111111111` | Visa | Success |
| `4000000000000002` | Visa | Declined |
| `4000000000000127` | Visa | Insufficient funds |
| `5555555555554444` | Mastercard | Success |
| `378282246310005` | American Express | Success |

---

## Connector Health

Monitor connector status in the Control Centre:

- **Transaction success rates**: Percentage of successful payments
- **Error rate trends**: Historical view of failure rates
- **Latency metrics**: Average response time from connector
- **Uptime status**: Real-time availability of the connector

Use these metrics to:
- Identify underperforming connectors
- Make data-driven routing decisions
- Set up alerts for degraded performance

---

## Quick Links

| Resource | Description | Link |
|----------|-------------|------|
| Activate Connector | Detailed guide on configuring connectors and payment methods | [Documentation](https://docs.juspay.io/hyperswitch/connectors/activate-connector) |
| Integrations Directory | Complete list of available connectors and payment methods | [Directory](https://docs.juspay.io/hyperswitch/integrations) |
| API Reference | Complete API documentation | [API Docs](https://docs.juspay.io/hyperswitch/api) |
| Request Integration | Raise an integration request | [Slack Community](https://join.slack.com/t/hyperswitch-io) |
| Status Page | Check system status and incidents | [Status](https://status.juspay.io) |

---

## Next Steps

- [Activate your first connector](https://docs.juspay.io/hyperswitch/connectors/activate-connector)
- [Configure intelligent routing between connectors](https://docs.juspay.io/hyperswitch/workflows/intelligent-routing)
- [Set up fallback connectors for reliability](https://docs.juspay.io/hyperswitch/workflows/smart-retries)
- [Implement webhook handling](https://docs.juspay.io/hyperswitch/webhooks)
- [Review payment methods configuration](https://docs.juspay.io/hyperswitch/payment-methods)
