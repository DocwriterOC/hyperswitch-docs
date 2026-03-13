---
description: Prerequisites for adding and configuring connectors in Hyperswitch
icon: checklist
---

# Connector Prerequisites

Before adding a connector in Hyperswitch, ensure you have completed all the requirements below.

## Checklist

### Hyperswitch Account
- [ ] Registered on [Hyperswitch Control Center](https://app.hyperswitch.io/)
- [ ] Admin access to configure connectors
- [ ] Organization set up with appropriate settings

### API Keys
- [ ] Access to your Hyperswitch API keys
- [ ] Navigate to: Settings → API Keys
- [ ] Generated and securely stored API key for your environment

### PSP Account
- [ ] Active account with your chosen payment processor
- [ ] Account in good standing
- [ ] Access to PSP dashboard/developer portal
- [ ] Permissions to generate API credentials

### PSP Credentials
- [ ] Required authentication details from your PSP dashboard
- [ ] API keys, secrets, or certificates as required by the PSP
- [ ] Merchant ID and any additional identifiers

### Environment Decision
- [ ] Decision on whether to configure in Sandbox or Production
- [ ] Sandbox credentials for testing (recommended first)
- [ ] Production credentials for live transactions

{% hint style="warning" %}
**Important:** Always test connectors in Sandbox mode before activating in Production.
{% endhint %}

---

## Quick Start

Once you've completed the prerequisites above, proceed to [Adding a Connector](./README.md#adding-a-connector) for step-by-step configuration instructions.
