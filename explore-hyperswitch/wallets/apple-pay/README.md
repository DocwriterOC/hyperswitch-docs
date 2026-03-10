---
description: Guide to using Apple Pay payment method on Hyperswitch
icon: apple-pay
---

# Apple Pay

Apple Pay allows customers to securely pay using saved cards from their Apple Pay account on macOS (Safari) or iOS using Touch ID and Face ID, eliminating the need to manually enter card and shipping details. Apple Pay is supported by participating banks and card issuers in 75+ countries.

**Minimum Requirements:**
- macOS 10.12.1+ or iOS 10.1+
- Valid SSL certificate (HTTPS)
- Apple Developer Account

---

## TL;DR

- **Merchant Identity Certificate**: Used to prove your identity to Apple during payment sessions. Valid for 25 months.
- **Payment Processing Certificate**: Used to decrypt payment tokens. Valid for 25 months.
- **Domain verification file**: Host at `/.well-known/apple-developer-merchantid-domain-association` (no .txt extension).
- **iOS domain field**: Enter `web` — this is Apple's identifier for web-based merchant validation.
- **Base64 encode**: Certificates and keys must be base64-encoded before pasting into Hyperswitch.
- **Security**: Private keys are sensitive — store securely and never commit to version control.

---

## Certificate Types Explained

Apple Pay uses two distinct certificates. Understanding the difference is critical for correct setup:

| Certificate | Purpose | Who Generates CSR | Validity |
|-------------|---------|-------------------|----------|
| **Merchant Identity Certificate** | Proves your identity to Apple during payment sessions | You (merchant) | 25 months |
| **Payment Processing Certificate** | Decrypts payment tokens for transaction processing | Processor (e.g., Stripe) OR You (if decrypting at Hyperswitch) | 25 months |

### Merchant Identity Certificate

This certificate authenticates your server to Apple when initiating payment sessions. Think of it as your "ID card" that proves to Apple you are a legitimate merchant.

**When you need it:**
- Web domain verification
- iOS app payment sessions

**How it works:**
1. You generate a CSR (Certificate Signing Request) using OpenSSL
2. Upload the CSR to Apple Developer portal
3. Apple issues a `.cer` file
4. Convert to `.pem` format for use in Hyperswitch

### Payment Processing Certificate

This certificate allows you (or your processor) to decrypt the encrypted payment tokens that Apple sends during transactions.

**How it works (Option 1 — Processor handles decryption):**
1. Get CSR from your processor's dashboard
2. Upload to Apple Developer portal
3. Download `.cer` file
4. Upload `.cer` back to processor's dashboard

**How it works (Option 2 — Hyperswitch handles decryption):**
1. You generate CSR using OpenSSL
2. Upload to Apple Developer portal
3. Download `.cer` file
4. Base64 encode and paste into Hyperswitch

---

## Web Domain Setup

### Prerequisites

Before integrating Apple Pay Web Domain:

1. **Apple Developer Account**: Sign up at [Apple Developer](https://developer.apple.com)
2. **SSL Certificate**: Your domain must use HTTPS
3. **macOS 10.12.1+ or iOS 10.1+** on customer devices

{% hint style="warning" %}
Since the Apple Pay Web Domain flow involves decryption at Hyperswitch, you may need to contact your payment processor to enable this feature for your account. Stripe is one processor that requires this.
{% endhint %}

<details>
<summary>You can use the following text in the email</summary>

* Attach our PCI DSS AoC certificate and copy our Support team (hyperswitch@juspay.in).
* Stripe Account id: <`Enter your account id:` you can find it [here](https://dashboard.stripe.com/settings/user)>
* A detailed business description: <`One sentence about your business`>. The business operates across `xx` countries and has customers across the world.
* Feature Request: We are using Hyperswitch, a Level 1 PCI DSS 3.2.1 compliant Payments Orchestrator, to manage payments on our website. In addition to Stripe, since we are using other processors as well to process payments across multiple geographies, we wanted to use Hyperswitch's Payment Processing certificate to decrypt Apple pay tokens and send the decrypted Apple pay tokens to Stripe. So, please enable processing decrypted Apple pay token feature on our Stripe account. We've attached Hyperswitch's PCI DSS AoC for reference.

</details>

### Step 1: Create Apple Merchant ID

1. Log in to your [Apple Developer account](https://developer.apple.com)
2. Navigate to **Certificates, Identifiers & Profiles** → **Identifiers**
3. Click the **+** button to add a new identifier
4. Select **Merchant IDs** and click **Continue**
5. Enter a description (e.g., "Merchant ID for test environment")
6. Enter a unique identifier using reverse domain notation (e.g., `merchant.com.yourdomain.sandbox`)
7. Click **Continue**, then **Register**

### Step 2: Validate Merchant Domain

1. In Apple Developer portal, go to **Identifiers** and select your Merchant ID
2. Under **Merchant Domains**, click **Add Domain**
3. Enter your merchant domain (e.g., `yourdomain.com`) and click **Save**
4. Click **Download** to get the domain verification file
5. **Host the file at:**
   ```
   https://yourdomain.com/.well-known/apple-developer-merchantid-domain-association
   ```
   {% hint style="warning" %}
   **Important:** The file must NOT have a `.txt` extension when hosted.
   {% endhint %}
6. Once hosted, return to Apple Developer portal and click **Verify**
7. Ensure the status shows as **Verified**

### Step 3: Configure in Hyperswitch

1. Log in to the Hyperswitch control centre
2. Navigate to the **Connectors** tab and select your processor
3. While selecting Payment Methods, click **Apple Pay** in the Wallet section
4. Select the **Web Domain** option
5. Enter your verified domain name
6. Click **Verify and Enable** to complete the setup

---

## iOS Application Setup

### Step 1: Create Merchant Identity Certificate

{% hint style="warning" %}
**Security Warning:** Private keys are sensitive credentials. Store them securely using a secrets manager (e.g., AWS Secrets Manager, Azure Key Vault, HashiCorp Vault) and never commit them to version control.
{% endhint %}

1. Open a terminal and create the CSR and private key:
   ```bash
   openssl req -out uploadMe.csr -new -newkey rsa:2048 -nodes -keyout certificate_sandbox.key
   ```

2. Enter your details when prompted (Country, Organisation, etc.)
3. This generates two files:
   - `uploadMe.csr` — upload to Apple
   - `certificate_sandbox.key` — keep secure, needed for Hyperswitch

4. In Apple Developer portal, select your Merchant ID
5. Under **Apple Pay Merchant Identity Certificate**, click **Create Certificate**
6. Upload `uploadMe.csr` and click **Continue**
7. Download the `.cer` file
8. Convert to `.pem` format:
   ```bash
   openssl x509 -inform der -in merchant_id.cer -out certificate_sandbox.pem
   ```

### Step 2: Create Payment Processing Certificate (If Decrypting at Hyperswitch)

{% tabs %}
{% tab title="Payment Processing Details At Connector" %}

* You will need to get a **.csr** file from your processor's dashboard _(like Adyen, Cybersource)_
* Log in to your [Apple Developer account](https://developer.apple.com/account/resources/certificates/list), go to Identifiers and select the Merchant ID you created previously
* Under the **Apple Pay Payment Processing Certificate** section, click on Create Certificate
* After answering whether the Merchant ID will be processed exclusively in China mainland, click on Continue
* Upload the **.csr** you received from your processor and click Continue
* Click on the prompted Download button and you will get a .**cer** file, Upload this **.cer** file you received while creating Apple MerchantID Certificate on the processor's dashboard.

{% hint style="warning" %}
This final step is specific to the processor being used and is not necessary in Sandbox Test environment for some processors, such as Authorize.Net.
{% endhint %}

{% endtab %}

{% tab title="Payment Processing Details At Hyperswitch" %}

1. Generate the private key:
   ```bash
   openssl ecparam -name prime256v1 -genkey -noout -out ppc_private.key
   ```

2. Generate the CSR:
   ```bash
   openssl req -out ppc_uploadMe.csr -new -key ppc_private.key
   ```

3. In Apple Developer portal, select your Merchant ID
4. Under **Apple Pay Payment Processing Certificate**, click **Create Certificate**
5. Upload `ppc_uploadMe.csr` and click **Continue**
6. Download the `.cer` file

{% hint style="warning" %}
Please note since this flow involves decryption at Hyperswitch, you may need to write to your payment processor to get this feature enabled for your account. Stripe is one among them.
{% endhint %}

<details>
<summary>You can use the following text in the email</summary>

* Attach our PCI DSS AoC certificate and copy our Support team (hyperswitch@juspay.in).
* Stripe Account id: <`Enter your account id:` you can find it [here](https://dashboard.stripe.com/settings/user)>
* A detailed business description: <`One sentence about your business`>. The business operates across `xx` countries and has customers across the world.
* Feature Request: We are using Hyperswitch, a Level 1 PCI DSS 3.2.1 compliant Payments Orchestrator, to manage payments on our website. In addition to Stripe, since we are using other processors as well to process payments across multiple geographies, we wanted to use Hyperswitch's Payment Processing certificate to decrypt Apple pay tokens and send the decrypted Apple pay tokens to Stripe. So, please enable processing decrypted Apple pay token feature on our Stripe account. We've attached Hyperswitch's PCI DSS AoC for reference.

</details>

{% endtab %}
{% endtabs %}

### Step 3: Base64 Encode Certificates and Keys

Before entering data into Hyperswitch, you must base64 encode the files:

**Encode certificate (.pem or .cer):**
```bash
base64 -i certificate_sandbox.pem
```

**Encode private key (.key):**
```bash
base64 -i certificate_sandbox.key
```

Copy the entire base64-encoded string and paste into the Hyperswitch dashboard.

### Step 4: Configure in Hyperswitch

1. Log in to the Hyperswitch dashboard
2. Navigate to **Connectors** and select your processor
3. Click **Apple Pay** in the Wallet section
4. Select **iOS Certificate** option
5. Fill in the form:

   | Field | Value |
   |-------|-------|
   | **Apple Merchant Identifier** | Your Merchant ID (e.g., `merchant.com.yourdomain.sandbox`) |
   | **Merchant Certificate** | Base64-encoded content of `.pem` file |
   | **Merchant Private Key** | Base64-encoded content of `.key` file |
   | **Display Name** | Your merchant name (shown to customers) |
   | **Domain** | Enter `web` (see explanation below) |
   | **Domain Name** | Your verified domain from Apple Developer |

{% hint style="info" %}
**Why Enter "web" in the Domain Field?**

The Domain field in the iOS configuration expects the value `web` when you're using web-based merchant validation. This is an Apple-defined identifier that tells the system to use web domain validation rather than app-specific validation. It does not refer to your actual domain name — that goes in the **Domain Name** field.
{% endhint %}

<figure><img src="../../../.gitbook/assets/Screenshot 2024-08-06 at 6.56.28 PM.png" alt="" width="563"><figcaption></figcaption></figure>

### Step 5: Integrate with Xcode

1. Open your iOS project in Xcode
2. Go to **Project Settings** → **Signing & Capabilities**
3. Click **+ Capability**
4. Add **Apple Pay**
5. Select the Merchant ID you created earlier

<figure><img src="../../../.gitbook/assets/applepay.png" alt=""><figcaption><p>Enable the Apple Pay capability in Xcode</p></figcaption></figure>

---

## Certificate Expiry and Renewal

Apple Pay certificates are valid for **25 months** from the date of creation.

### Monitoring Expiry

- Apple does not send automatic expiry notifications
- Mark renewal dates in your calendar when creating certificates
- Set reminders 30 days before expiry

### Renewal Process

**Merchant Identity Certificate:**
1. Generate a new CSR and private key
2. Create a new certificate in Apple Developer portal
3. Convert the new `.cer` to `.pem`
4. Update the certificate in Hyperswitch
5. Test transactions before the old certificate expires

**Payment Processing Certificate:**
- **If processor handles decryption:** Contact your processor for renewal instructions
- **If Hyperswitch handles decryption:**
  1. Generate new CSR and key pair
  2. Create new certificate in Apple Developer portal
  3. Base64 encode and update in Hyperswitch

{% hint style="danger" %}
There is no grace period — expired certificates immediately stop working. Plan renewals in advance.
{% endhint %}

---

## Testing

### Test Transaction Guidance

**Before going live:**

1. **Use Apple Pay Sandbox:**
   - Add test cards to your Apple Pay wallet using Apple's [test card numbers](https://developer.apple.com/apple-pay/sandbox-testing/)
   - Test cards simulate successful and failed transactions

2. **Test Scenarios:**
   - Successful payment with Touch ID / Face ID
   - Payment cancellation
   - Network failure during transaction
   - Invalid certificate handling
   - Expired certificate simulation

3. **Minimum Test Checklist:**
   - [ ] Domain verification shows "verified" in Apple Developer
   - [ ] Payment sheet displays correctly on macOS Safari
   - [ ] Payment sheet displays correctly on iOS
   - [ ] Transaction completes successfully
   - [ ] Payment token decrypts correctly (if handling at Hyperswitch)
   - [ ] Error messages display appropriately

4. **Test Environment Indicators:**
   - Sandbox transactions show "[Test Mode]" or similar indicators
   - No real money is charged
   - Test cards have specific BIN ranges

---

## Troubleshooting

### Domain Verification Issues

**Problem:** Domain verification fails with "Unable to verify domain"

**Solutions:**
- Ensure the file is hosted at exactly `/.well-known/apple-developer-merchantid-domain-association` (no `.txt` extension)
- Verify the file is accessible via HTTPS (not HTTP)
- Check that the file content matches exactly what Apple provided
- Ensure no redirects are in place for the `.well-known` path
- Confirm the domain in Apple Developer matches exactly (including subdomains)

### Certificate Issues

**Problem:** "Invalid certificate" error

**Solutions:**
- Verify the certificate is in `.pem` format (not `.cer`)
- Ensure the entire certificate content is base64-encoded
- Check that the certificate has not expired
- Confirm you're using the correct certificate type (Identity vs Processing)

**Problem:** "Private key mismatch"

**Solutions:**
- Ensure the private key matches the CSR used to generate the certificate
- Verify the key is base64-encoded correctly
- Check for extra whitespace or missing characters in the encoded string

### Payment Processing Issues

**Problem:** Payment succeeds but transaction fails

**Solutions:**
- If decrypting at Hyperswitch, verify the Payment Processing Certificate is correctly configured
- Check processor logs for decryption errors
- Ensure the Payment Processing Certificate has not expired
- Contact your processor to confirm Apple Pay is enabled on your account

### iOS Integration Issues

**Problem:** Apple Pay sheet does not appear

**Solutions:**
- Verify the Merchant ID in Xcode matches the one in Apple Developer
- Ensure the "Domain" field in Hyperswitch is set to `web`
- Check that the domain name is verified in Apple Developer
- Confirm the device supports Apple Pay (iOS 10.1+, macOS 10.12.1+)

### General Debugging Steps

1. Check Apple Developer portal for any warnings or errors
2. Verify all certificates are current and not expired
3. Test domain file accessibility with:
   ```bash
   curl -I https://yourdomain.com/.well-known/apple-developer-merchantid-domain-association
   ```
4. Review Hyperswitch dashboard for connector-specific error messages
5. Check browser console for JavaScript errors during payment initiation

---

## Security Best Practices

### Private Key Handling

{% hint style="danger" %}
**Critical:** Private keys grant access to payment processing. Protect them accordingly:
{% endhint %}

- **Never** commit private keys to version control (Git, SVN, etc.)
- **Never** share private keys via email or messaging
- **Never** store private keys in plaintext configuration files
- **Always** use a secrets manager (AWS Secrets Manager, Azure Key Vault, HashiCorp Vault)
- **Always** restrict access to private keys using IAM/role-based access
- **Rotate** keys periodically as part of security hygiene

### Certificate Storage

- Store certificates in encrypted format when not in use
- Limit access to certificates to essential personnel only
- Maintain an audit log of certificate access and updates

---

## Quick Reference Commands

```bash
# Generate Merchant Identity CSR and key
openssl req -out uploadMe.csr -new -newkey rsa:2048 -nodes -keyout certificate_sandbox.key

# Convert .cer to .pem
openssl x509 -inform der -in merchant_id.cer -out certificate_sandbox.pem

# Generate Payment Processing key (ECC)
openssl ecparam -name prime256v1 -genkey -noout -out ppc_private.key

# Generate Payment Processing CSR
openssl req -out ppc_uploadMe.csr -new -key ppc_private.key

# Base64 encode a file (Linux/macOS)
base64 -i input.pem

# Base64 encode to stdout (copy-paste friendly)
base64 input.pem | pbcopy  # macOS
base64 input.pem | xclip -selection clipboard  # Linux with xclip
```

---

## Support

For additional help:
- [Apple Pay Documentation](https://developer.apple.com/apple-pay/)
- [Hyperswitch Documentation](https://docs.hyperswitch.io)
- Contact your payment processor for processor-specific guidance

_Please feel free to reach out to Hyperswitch support if you are stuck at any stage when integrating and testing Apple Pay._
