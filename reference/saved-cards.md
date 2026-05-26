# Pay With Saved Cards

Allow returning customers to pay without re-entering card details. Two transaction types: **CIT** (customer present) and **MIT** (merchant-initiated, no customer interaction).

---

## Flow Overview

```
First Payment:  Create Intention → Customer saves card → You receive card_token
Future CIT:     Create Intention with card_tokens → Customer authenticates (OTP)
Future MIT:     Create Intention (Moto) → Get payment_key → Pay Request with card_token
```

---

## Step 1: Create Card Token

This happens during the customer's first payment when they opt to save their card.

### Create Intention

Use **Verification**, **Normal 3DS**, or **Auth** card integration ID:

```json
{
  "amount": 10000,
  "currency": "EGP",
  "payment_methods": [1234],
  "items": [{"name": "Product", "amount": 10000, "quantity": 1}],
  "billing_data": { /* ... */ },
  "notification_url": "https://your-server.com/paymob/callback"
}
```

### Render Checkout

Customer sees "Save card for future purchases" option (controlled by `showSaveCard` in Pixel or Dashboard setting for Unified Checkout).

### Receive Card Token

Paymob sends a **Card Token callback** to your `notification_url`:

```json
{
  "type": "TOKEN",
  "obj": {
    "id": 8555026,
    "token": "e98aceb96f5a370ddf46460db9d555f88bf12448f80e1839b39f78ab",
    "masked_pan": "xxxx-xxxx-xxxx-2346",
    "merchant_id": 246628,
    "card_subtype": "MasterCard",
    "email": "test@test.com",
    "order_id": 264064419,
    "created_at": "2024-11-13T12:32:23.859982"
  }
}
```

**Store** the `token` value (associated with this customer) — you'll use it for future payments.

**HMAC verification** is required. Key order: `card_subtype`, `created_at`, `email`, `id`, `masked_pan`, `merchant_id`, `order_id`, `token` (see `reference/webhooks-hmac.md`).

---

## Step 2: CIT (Customer-Initiated Transaction)

Customer chooses a saved card at checkout and completes OTP authentication.

### Create Intention

```json
{
  "amount": 10000,
  "currency": "EGP",
  "payment_methods": [1234],
  "card_tokens": ["e98aceb96f5a370ddf46460db9d555f88bf12448f80e1839b39f78ab"],
  "items": [{"name": "Product", "amount": 10000, "quantity": 1}],
  "billing_data": { /* ... */ }
}
```

**Card integration type**: Use **Normal 3DS**, **Auth**, or **Card On File**.
- Card On File → customer only enters OTP (no CVV)
- Normal 3DS → customer enters CVV + OTP

**card_tokens array**: accepts up to **3 tokens** (for split across multiple saved cards).

### Render Checkout

Customer sees the saved card pre-selected. They authenticate with OTP. Payment completes as normal.

---

## Step 3: MIT (Merchant-Initiated Transaction)

Merchant charges a saved card without customer interaction (recurring, top-ups, etc.).

### Create Intention

Use **Moto** card integration ID:

```json
{
  "amount": 10000,
  "currency": "EGP",
  "payment_methods": [1234],
  "items": [{"name": "Product", "amount": 10000, "quantity": 1}],
  "billing_data": { /* ... */ }
}
```

### Extract Payment Key

From the response:

```json
{
  "payment_keys": [{ "key": "moto_payment_token_xxx" }]
}
```

### Call Pay Request

The Pay Request endpoint charges the saved card. The exact URL is documented in Paymob's MIT guide under Developers Reference. A typical request:

```javascript
POST https://accept.paymob.com/api/acceptance/pay
Authorization: Token {secret_key}

{
  "source": {
    "identifier": "e98aceb96f5a370ddf46460db9d555f88bf12448f80e1839b39f78ab",
    "subtype": "CARD"
  },
  "payment_token": "moto_payment_token_xxx"
}
```

Payment processes without customer interaction. Callback delivered to `notification_url`.

**Note**: The MIT Pay Request endpoint URL may vary by region. Check the official Paymob MIT documentation for your region's correct URL.

---

## Constraints

- `card_tokens` array: max **3 tokens** per intention
- Card On File type → OTP only (no CVV required)
- Moto type (MIT flow) → no customer interaction. OTP/CVV are handled via the card token, not re-entered.
- Card tokens are tied to the merchant account that created them
- Token doesn't expire but may become invalid if the card is replaced

---

## Anti-Patterns

- **Do not store raw PAN** — only store the Paymob card token. Storing PAN puts you in PCI scope.
- **Do not use Normal 3DS for MIT** — use Moto integration ID for merchant-initiated charges.
- **Do not reuse card tokens across different merchant accounts** — tokens are account-specific.
- **Do not skip HMAC verification on card token callbacks** — tokens can be intercepted.
