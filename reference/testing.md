# Testing & Go-Live

---

## Test Credentials

### Card Test Numbers

| Network | Number | Expiry | CVV | Cardholder Name |
|---|---|---|---|---|
| Mastercard | `5123456789012346` | 01/39 | 123 | Test Account |
| Mastercard | `5123450000000008` | 01/39 | 123 | Test Account |
| Visa | `4111111111111111` | 01/39 | 123 | Test Account |

### Mobile Wallet Test Credentials

| Field | Value |
|---|---|
| Wallet Number | `01010101010` |
| MPin Code | `123456` |
| OTP | `123456` |

### 3DS OTP

Any 6 digits (e.g., `123456`) works during testing.

---

## Test Mode Setup

1. In Paymob Dashboard → **Settings**, toggle to **Test Mode**
2. Copy the **test secret key** and **test public key**
3. Go to **Developers → Payment Integrations** to get **test integration IDs** (ensure they're created in test mode)
4. Use these in your integration
5. Run test transactions: create intention → checkout → payment → verify callback
6. Test refund, void, and capture flows

---

## Go-Live Checklist

- [ ] Account created and documents validated
- [ ] Integration complete and tested in test environment
- [ ] All payment flows tested (card success, card decline, wallet, refund, void if applicable)
- [ ] HMAC verification implemented and validated using Webhook Testing Tool
- [ ] Transaction Processed Callback endpoint is active and responds 200
- [ ] Response Callback URL configured per integration ID in Paymob Dashboard
- [ ] Idempotency handled (same callback may arrive multiple times)
- [ ] Error states handled gracefully (declined, expired, pending)
- [ ] Test credentials swapped for live credentials
- [ ] Technical approval obtained from Paymob
- [ ] Live credentials issued
- [ ] Production launch

---

## Apple Pay Setup

### Domain Verification (for Pixel/Web)

1. **Apple Developer Portal** → Certificates, Identifiers & Profiles
2. Create or select a **Merchant ID**
3. Add your domain: `myshop.com` (no `https://`)
4. Download verification file: `apple-developer-merchantid-domain-association`
5. Upload to: `https://yourdomain/.well-known/apple-developer-merchantid-domain-association`

### Merchant Identity Certificate

```bash
openssl genpkey -algorithm RSA -out merchant_identity.key
openssl req -new -key merchant_identity.key -out merchant_identity.csr
```

Upload `merchant_identity.csr` to Apple Developer Portal → download `.cer`:

```bash
openssl x509 -inform DER -in merchant_id.cer -out merchant_certificate.pem
```

### Payment Processing Certificate

```bash
openssl ecparam -genkey -name prime256v1 -out payment_processing.key
openssl req -new -key payment_processing.key -out payment_processing.csr
```

Upload to Apple → download → convert to PEM. Upload both PEM certificates to Paymob Dashboard.

---

## Webhook Testing Tool

Use https://webhook.paymob.com/ to:
1. Generate a temporary endpoint URL
2. Set it as your `notification_url` in test intentions
3. Run test transactions
4. Inspect callback payloads
5. Validate your HMAC implementation against real data

---

## Live vs Test Differences

| Item | Test | Live |
|---|---|---|
| Secret Key | `egy_sk_test_...` | `egy_sk_live_...` |
| Public Key | `egy_pk_test_...` | `egy_pk_live_...` |
| Integration IDs | Created in test mode | Created in live mode |
| Real money | No | Yes |
| Apple Pay | Not available | Available |
| Google Pay | Not available | Available |
| Bank Installments | Not available | Available |
