---
icon: file-invoice-dollar
---

# Accept Card Payments

> **TL;DR**: Learn how to process card payments using Juspay Hyperswitch, from one-time payments to recurring billing with saved payment methods.

Juspay Hyperswitch provides flexible payment processing with multiple flow patterns to accommodate different business needs. The system supports one-time payments, saved payment methods, and recurring billing through a comprehensive API design.

## Integration Path

### Client-Side SDK Payments (Tokenise Post Payment)

Refer to the Payments (Cards) section if your flow requires the SDK to initiate payments directly. In this model, the SDK handles the payment trigger and communicates downstream to the Hyperswitch server and your chosen Payment Service Providers (PSPs). This path is ideal for supporting dynamic, frontend-driven payment experiences.

![Integration Path Overview](https://1943537505-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2Fkf7BGdsPkCw9nalhAIlE%2Fuploads%2FHk97nO7DYWl6Z6lnrndD%2Fimage.png?alt=media&token=5ef5acc5-63c5-4b07-9fb9-75163cb0a278)

## One-Time Payment Patterns

### 1. Instant Payment (Automatic Capture)

**Use Case:** Simple, immediate payment processing

**Endpoint:** `POST /payments`

![Instant Payment Flow](https://1943537505-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2Fkf7BGdsPkCw9nalhAIlE%2Fuploads%2FQbdrFcNa7kcEVzo8SCeZ%2Fimage.png?alt=media&token=7d7e0e13-b06f-4c19-80cb-e76dccc309ce)

**Required Fields:**

| Field | Value | Description |
|-------|-------|-------------|
| `confirm` | `true` | Immediately confirm the payment |
| `capture_method` | `"automatic"` | Capture funds automatically |
| `payment_method` | - | The payment method to use |

**Final Status:** `succeeded`

### 2. Two-Step Manual Capture

**Use Case:** Deferred capture (e.g., ship before charging)

![Two-Step Manual Capture Flow](https://1943537505-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2Fkf7BGdsPkCw9nalhAIlE%2Fuploads%2FRgI9s4yiESb48I4Ayhmf%2Fimage.png?alt=media&token=02bea455-8a22-4d54-a8cf-a835afc00d52)

**Flow:**

1. **Authorise:** `POST /payments` with `capture_method: "manual"`
2. **Status:** `requires_capture`
3. **Capture:** `POST /payments/{payment_id}/capture`
4. **Final Status:** `succeeded`

Read more about [Manual Capture](https://docs.hyperswitch.io/~/revisions/2M8ySHqN3pH3rctBK2zj/about-hyperswitch/payment-suite-1/payments-cards/manual-capture).

### 3. Fully Decoupled Flow

**Use Case:** Complex checkout journeys with multiple modification steps. Useful in headless checkout or B2B portals where data is filled progressively.

![Fully Decoupled Flow](https://1943537505-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2Fkf7BGdsPkCw9nalhAIlE%2Fuploads%2FFuGafLUkayRfQezNBnrc%2Fimage.png?alt=media&token=06a35889-3d9e-441e-b7c8-c3e1863c3d12)

**Endpoints:**

| Step | Endpoint | Purpose |
|------|----------|---------|
| Create | `POST /payments` | Initialise the payment |
| Update | `POST /payments/{payment_id}` | Modify payment details |
| Confirm | `POST /payments/{payment_id}/confirm` | Confirm the payment |
| Capture | `POST /payments/{payment_id}/capture` | Capture funds (if manual) |

### 4. 3D Secure Authentication Flow

**Use Case:** Enhanced security with customer authentication

![3D Secure Authentication Flow](https://1943537505-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2Fkf7BGdsPkCw9nalhAIlE%2Fuploads%2Ff9yXTJllYj8KadAon2h5%2Fimage.png?alt=media&token=a0003439-d782-496c-8180-e9cc42c06754)

**Additional Fields:**

| Field | Value | Description |
|-------|-------|-------------|
| `authentication_type` | `"three_ds"` | Enable 3D Secure authentication |

**Status Progression:** `processing` → `requires_customer_action` → `succeeded`

Read more about the [3DS Decision Manager](https://docs.hyperswitch.io/~/revisions/9QlGypixZFcbkq8oGjaF/explore-hyperswitch/workflows/3ds-decision-manager).

## Recurring Payments and Payment Storage

### Saving Payment Methods

**During Payment Creation:**

- Add `setup_future_usage: "off_session"` or `"on_session"`
- Include `customer_id`
- **Result:** `payment_method_id` returned on success

**Understanding `setup_future_usage`:**

| Value | Use When | Example Scenarios |
|-------|----------|-------------------|
| `on_session` | The customer is actively present during the transaction | Saving card details for faster checkouts where the customer will still be present to initiate the payment (e.g., card vaulting for e-commerce sites) |
| `off_session` | You intend to charge the customer later without their active involvement at the time of charge | Subscriptions, recurring billing, or merchant-initiated transactions (MITs) where the customer has pre-authorised future charges |

### Using Saved Payment Methods

![Using Saved Payment Methods](https://1943537505-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2Fkf7BGdsPkCw9nalhAIlE%2Fuploads%2FOb8tolYsFRPWM4tyr2BE%2Fimage.png?alt=media&token=f466306f-f5a3-4ecf-bd25-c6c15e216da3)

**Steps:**

1. **Initiate:** Create payment with `customer_id`
2. **List:** Get saved cards via `GET /customers/payment_methods`
3. **Confirm:** Use selected `payment_token` in confirm call

### PCI Compliance and `payment_method_id`

Storing `payment_method_id` (which is a token representing the actual payment instrument, which could be a payment token, network token, or payment processor token) significantly reduces your PCI DSS scope. Hyperswitch securely stores the sensitive card details and provides you with this token. While you still need to ensure your systems handle `payment_method_id` and related customer data securely, you avoid the complexities of storing raw card numbers. Always consult with a PCI QSA to understand your specific compliance obligations.

## Recurring Payment Flows

### Customer-Initiated Transaction (CIT) Setup

![Customer-Initiated Transaction Setup](https://1943537505-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2Fkf7BGdsPkCw9nalhAIlE%2Fuploads%2F6O5caNNsaZoSmwKBDOh3%2Fimage.png?alt=media&token=a603a8b9-346f-4479-8f8a-3c128ded16c2)

Read more about [Recurring Payments](https://docs.hyperswitch.io/~/revisions/j00Urtz9MpwPggJzRCsi/about-hyperswitch/payment-suite-1/payments-cards/recurring-payments).

### Merchant-Initiated Transaction (MIT) Execution

![Merchant-Initiated Transaction Execution](https://1943537505-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2Fkf7BGdsPkCw9nalhAIlE%2Fuploads%2FvwIQJKBVjm1iwp5fcKdI%2Fimage.png?alt=media&token=43c3f647-b156-4291-856c-923f85016dcb)

Read more about [Recurring Payments](https://docs.hyperswitch.io/~/revisions/j00Urtz9MpwPggJzRCsi/about-hyperswitch/payment-suite-1/payments-cards/recurring-payments).

## Status Flow Summary

![Payment Status Flow Summary](https://1943537505-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2Fkf7BGdsPkCw9nalhAIlE%2Fuploads%2FNvUEtwyOhOeyirtGk9CX%2Fimage.png?alt=media&token=ecd05894-e85f-4452-b8c2-e4b3da2212c9)

## Notes

- **Terminal States:** `succeeded`, `failed`, `cancelled`, `partially_captured` are terminal states requiring no further action
- **Capture Methods:** System supports `automatic` (funds captured immediately), `manual` (funds captured in a separate step), `manual_multiple` (funds captured in multiple partial amounts via separate steps), and `scheduled` (funds captured automatically at a future predefined time) capture methods
- **Authentication:** 3DS authentication automatically resumes payment processing after customer completion
- **MIT Compliance:** Off-session recurring payments follow industry standards for merchant-initiated transactions
