---
description: Connectors form the foundation of Hyperswitch's payment orchestration layer, enabling seamless integration with various payment processors and service providers.
icon: network-wired
---

# Connectors

Connectors form the foundation of Hyperswitch's payment orchestration layer, enabling businesses to integrate with multiple payment processors (PSPs), fraud and risk management providers, tax processors and identity verification services.

As your business grows faster, there would be a need to expand payment offerings with more payment processors. This need might arise due to multiple reasons:

* **Vendor Independence:** Reducing dependency on a single processor and reducing vendor lock-in.
* **Performance Optimization:** Introducing a challenger processor for better cost and auth rates.
* **Global Reach:** Launching a business in new geography with a local payment processor.
* **Localized Experience:** Offering local or new payment methods for your customers.
* **Reliability:** Reducing technical downtimes and improving success rates with a fallback.

Integrating and maintaining multiple payment processors and their different versions is a time and resource intensive process. Hyperswitch can add a new PSP in [2 weeks](https://hyperswitch.io/blog/part-1-5-payment-challenges-for-vertical-saas-businesses), allowing you to focus on your core business activities.

---

## Prerequisites

Before adding a connector, ensure you have completed all [prerequisites](./prerequisites.md) including Hyperswitch account setup, PSP account registration, and credential access.

{% hint style="warning" %}
**Important:** Always test connectors in Sandbox mode before activating in Production.
{% endhint %}

---

## Adding a Connector

Most connector integrations follow a simple click-and-connect flow on Hyperswitch using your connector credentials. However, some connectors may require additional setup details as required on the control center.

### Standard Setup Steps

#### Step 1: PSP Registration

You need to be registered with the PSP in order to proceed. In case you aren't, you can quickly set up your account by signing up on their dashboard.

**What you need:**
- PSP account in good standing
- Access to PSP dashboard/developer portal
- Permissions to generate API credentials

#### Step 2: Platform Access

Ensure you have registered on [Hyperswitch Control Center](https://app.hyperswitch.io/) and have admin access to configure connectors.

**Navigate to:**
```
Dashboard → Connectors → Add Connector
```

#### Step 3: Credential Mapping

Add the PSP authentication credentials from their dashboard into the Hyperswitch Control Center.

**Security Best Practices:**
- Store credentials securely (Hyperswitch encrypts all stored credentials)
- Use environment-specific credentials (sandbox vs production)
- Rotate credentials periodically per PSP guidelines
- Never share API keys in code repositories

**Authentication Examples**

Authentication credentials vary across different PSPs. Common combinations include:

You will need to locate these credentials within your PSP's developer portal or dashboard, then copy them into Hyperswitch Control Center.

| Provider | Required Credentials | Where to Find in PSP Dashboard |
|----------|---------------------|---------------|
| **Authorize.net** | API Login ID and Transaction Key | Account → API Login ID & Transaction Key |
| **Adyen** | API key and Account ID | Developer → API credentials |
| **Braintree** | Merchant ID, Public key and Private key | Account → API Keys |
| **Airwallex** | API key and Client ID | Developer Center → API Keys |
| **Fiserv** | API Key, API Secret, Merchant ID and Terminal ID | Developer Portal → Credentials |
| **PayPal** | Client Secret and Client ID | Developer Dashboard → App Credentials |

{% hint style="info" %}
**Tip:** Not sure where to find your credentials? Check our [Integrations Directory](https://juspay.io/integrations) for the full list of supported providers.
{% endhint %}

#### Step 4: Method Configuration

Choose the payment methods you want to utilize with the connector by navigating to the next screen on Hyperswitch.

**Configuration Options:**
- Select supported payment methods (Cards, Wallets, BNPL, etc.)
- Enable/disable 3D Secure per method
- Set fallback priority (if using multiple connectors)
- Configure webhook endpoints (optional but recommended)

#### Step 5: Activation & Testing

Enable the PSP once you're done with configuration.

**Testing Checklist:**
- [ ] Credentials verified (green checkmarks in Control Center)
- [ ] Payment methods selected and enabled
- [ ] Sandbox mode enabled
- [ ] Complete sandbox test transactions
- [ ] Verify test payment processed successfully
- [ ] Test refund flow (if applicable)

{% hint style="info" %}
**Note:** Testing is performed after activating the connector in Sandbox mode. The connector must be active to process test transactions.
{% endhint %}

---

## Testing Your Connector

### Sandbox Testing Checklist

Once your connector is active in Sandbox mode:

- [ ] Create test payment (use test card numbers)
- [ ] Verify payment status updates
- [ ] Test refund flow
- [ ] Test 3D Secure flow (if enabled)
- [ ] Verify webhook delivery
- [ ] Check transaction in PSP dashboard

### Common Test Cards

| Card Number | Brand | Scenario |
|-------------|-------|----------|
| `4242424242424242` | Visa | Successful payment |
| `4000000000000002` | Visa | Declined payment |
| `4000000000000127` | Visa | Incorrect CVC |

### Switching to Production

1. Update connector credentials to production keys
2. Disable "Test Mode" in connector settings
3. Update webhook URL to production endpoint
4. Process a small test transaction
5. Monitor closely for first 24 hours

---

## Connector Health & Monitoring

### Check Connector Status

In Control Center → Connectors, look for:
- 🟢 **Active:** Connector healthy and processing
- 🟡 **Warning:** Issues detected (check alerts)
- 🔴 **Error:** Connector not responding

### Common Issues & Resolution

| Issue | Cause | Resolution |
|-------|-------|------------|
| Authentication Failed | Invalid API key | Re-enter credentials from PSP dashboard |
| Connection Timeout | Network issue | Check PSP status page, retry |
| Method Not Supported | PSP doesn't support method | Disable method in connector config |
| Rate Limited | Too many requests | Implement exponential backoff |

---

## Connector Types

Hyperswitch supports a wide variety of connectors to manage your entire financial stack:

### Core Payments
- **Payment Processors:** Stripe, Adyen, Checkout.com, etc.
- **Acquirers:** Traditional bank acquirers
- **APMs:** Alternative Payment Methods (PayPal, Klarna, etc.)

### Platforms
- **Payment Platforms:** Marketplace solutions (Stripe Connect, Adyen for Platforms)
- **Payout Processors:** For bulk payouts and disbursements

### Recurring Billing
- **[Subscription Providers](https://docs.hyperswitch.io/explore-hyperswitch/payment-orchestration/subscriptions):** Chargebee, Recurly, etc.

### Security & Risk
- **[Card Vaults](https://docs.hyperswitch.io/explore-hyperswitch/workflows/vault):** Secure card storage
- **[3DS Authentications](https://docs.hyperswitch.io/explore-hyperswitch/workflows/3ds-decision-manager):** 3D Secure handling
- **[Fraud Management](https://docs.hyperswitch.io/explore-hyperswitch/workflows/fraud-and-risk-management):** Risk scoring providers

---

## Security Best Practices

1. **Credential Rotation:** Update API keys every 90 days
2. **Environment Separation:** Never mix sandbox and production credentials
3. **Webhook Validation:** Always verify webhook signatures
4. **IP Whitelisting:** Restrict PSP dashboard access by IP
5. **Audit Logging:** Monitor connector configuration changes

---

## FAQ

**Q: Can I use the same connector for multiple currencies?**
A: Yes, most connectors support multi-currency. Check your PSP's documentation.

**Q: What happens if a connector fails?**
A: If you have multiple connectors, Hyperswitch can route to backup connectors (requires [Intelligent Routing](../workflows/intelligent-routing/)).

**Q: How do I update connector credentials?**
A: Go to Control Center → Connectors → [Your Connector] → Edit → Update credentials.

**Q: Can I disable a connector temporarily?**
A: Yes, toggle the "Disabled" switch in connector settings. No transactions will route to it.

**Q: How do I know which payment methods a connector supports?**
A: Check our [Integrations Directory](https://juspay.io/integrations) for supported methods per connector.

**Q: Can I test payments before activating a connector?**
A: No, the connector must be active (in Sandbox mode) to process test transactions. Activate first, then test.

---

## Quick Links

| Resource | Description |
|----------|-------------|
| [**Activate Connector**](https://docs.hyperswitch.io/explore-hyperswitch/connectors/activate-connector-on-hyperswitch) | Detailed guide on connector configuration |
| [**Integrations Directory**](https://juspay.io/integrations) | All available connectors and payment methods |
| [**Request Integration**](https://hyperswitch-io.slack.com/ssb/redirect) | Don't see your processor? Request on Slack |
| [**API Reference**](https://docs.hyperswitch.io/api-reference) | Complete API documentation |
| [**Webhooks Guide**](https://docs.hyperswitch.io/webhooks) | Webhook setup and verification |
