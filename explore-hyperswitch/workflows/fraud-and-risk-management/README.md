---
description: Effectively enhance fraud detection with your preferred FRM engine
icon: shield-check
---

# Fraud & Risk Management

The Hyperswitch Fraud and Risk Management (FRM) workflow offers a comprehensive Unified API designed to cater to your specific payment validation needs, effectively enhancing fraud protection within your payment ecosystem.

## Key Benefits of FRM workflows

* **Processor-Agnostic Integration**: Single API connect lets you seamlessly connect with an FRM solution of choice.
* **Customized Fraud Strategies**: Enables users to adjust fraud prevention measures by allowing them to select between Pre-Auth and Post-Auth checks based on specific payment methods and connectors.
* **Unified Dashboard**: A consolidated interface that displays all flagged transactions, making decision-making on potential frauds a breeze.
* **Real-Time Alerts**: Immediate notifications are sent when potential fraudulent activity is detected, ensuring quick action and minimal losses.
* **Insightful Analytics**: Detailed reports on fraud patterns help inform decisions and strategy adjustments.

## Use cases for implementing FRM workflows

* **Online Marketplaces**: Secure transactions on e-commerce platforms, regardless of payment methods or processors. Benefit from chargeback guarantees, dispute resolution, account security, real-time fraud alerts, and streamlined fraud investigation tools.
* **High-Value Transactions**: Protect high value orders in luxury retail, or high-end custom services. Ensure payment validity, prevent chargebacks, optimize PSD2 compliance, reduce transaction friction, gain valuable fraud trend insights, and utilize predictive fraud analysis.

## Supported FRM workflows

| Pre-authorization flow | Post-authorization flow |
| ---------------------- | ------------------------ |
| The Pre-Auth flow is executed before payment authorization. | The Post-Auth flow occurs after payment authorization by the processor |
| This flow is supported for any payment method | This flow is supported for only cards payment method |
| Actions supported are: | Actions supported are: |
| - Continue on `Accept` | - Continue to `Accept` |
| - Halt on `Decline` | - Halt on `Decline` |
| | - Approve/Decline on `Review` |

## Pre-Auth Flow

Check fraud risk before sending to processor:

```bash
curl -X POST https://api.hyperswitch.io/v1/payments \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{
    "amount": 10000,
    "currency": "USD",
    "customer_id": "cust_12345",
    "frm_config": {
      "enabled": true,
      "flow": "pre_auth",
      "provider": "riskified"
    }
  }'
```

## Post-Auth Flow

Verify transaction legitimacy after processor authorization:

```bash
curl -X POST https://api.hyperswitch.io/v1/payments \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{
    "amount": 10000,
    "currency": "USD",
    "customer_id": "cust_12345",
    "frm_config": {
      "enabled": true,
      "flow": "post_auth",
      "provider": "signifyd"
    }
  }'
```

## FRM Providers

Hyperswitch integrates with leading fraud prevention providers:

| Provider | Pre-Auth | Post-Auth | Specialization |
|----------|----------|-----------|----------------|
| Riskified | ✓ | ✓ | E-commerce fraud |
| Signifyd | ✓ | ✓ | Chargeback guarantee |
| Forter | ✓ | ✓ | Enterprise fraud |
| Sift | ✓ | | Account abuse |

## Dashboard and Alerts

View and manage flagged transactions:

- **Risk Score**: Transaction-level risk assessment
- **Rule Triggers**: Which rules were triggered
- **Manual Review**: Queue for analyst review
- **Block/Allow Lists**: Manage exceptions

## Configuration

Set up FRM in Hyperswitch Control Centre:

1. Navigate to **Fraud & Risk**
2. Select your FRM provider
3. Enter API credentials
4. Configure rules and thresholds
5. Set up alert channels

## Analytics

Track fraud metrics:

- Fraud detection rate
- False positive rate
- Review queue size
- Blocked transaction value

## Next Steps

- [Configure FRM provider](./activating-frm-in-hyperswitch.md)
- [Review fraud blocklist](./fraud-blocklist.md)
