# Intention API

The **Payment Intention** is the starting point for every Paymob payment. One API call covers normal checkout, Pixel, SDKs, subscriptions, Auth/Cap, and saved cards.

---

## Create Intention

```
POST https://accept.paymob.com/v1/intention/
Authorization: Token {secret_key}
Content-Type: application/json
```

### Request Body

```json
{
  "amount": 10000,
  "currency": "EGP",
  "payment_methods": [1234],
  "items": [
    {
      "name": "Product Name",
      "amount": 10000,
      "quantity": 1,
      "description": "Full description"
    }
  ],
  "billing_data": {
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com",
    "phone_number": "+201000000000",
    "country": "EGY"
  },
  "special_reference": "order_12345",
  "notification_url": "https://your-server.com/paymob/callback"
}
```

### Field Reference

| Field | Required | Type | Notes |
|---|---|---|---|
| `amount` | Yes | integer | In cents (10000 = 100.00 EGP) |
| `currency` | Yes | string | ISO 4217: EGP, SAR, AED, OMR |
| `payment_methods` | Yes | array[int] | Integration IDs for allowed payment methods |
| `items[].name` | **Yes** | string | Commonly missed — causes 400 |
| `items[].amount` | **Yes** | integer | Per-item amount in cents |
| `items[].quantity` | No | integer | Defaults to 1 |
| `billing_data` | Yes | object | first_name, last_name, email, phone_number, country |
| `special_reference` | No | string | Your internal order ID — returned in callbacks |
| `notification_url` | No | string | Webhook endpoint — receives processed callbacks + card tokens |
| `card_tokens` | No | array[string] | For saved card payments (CIT). Up to 3 tokens |
| `subscription_plan_id` | No | integer | For creating subscriptions |
| `subscription_start_date` | No | string | YYYY-MM-DD format |

### Response

```json
{
  "id": 123456,
  "order": { "id": 789012 },
  "client_secret": "egy_csk_live_xxxxxxxxxxxx",
  "payment_keys": [{ "key": "payment_token_xxx" }]
}
```

| Field | Use |
|---|---|
| `client_secret` | Pass to Unified Checkout URL or Pixel. One-time use. |
| `order.id` | Paymob order ID — correlate callbacks. |
| `id` | Intention ID — alternative for correlation. |
| `payment_keys[].key` | Payment token for MIT (Moto) flows. |

### Common Errors

| Status | Error | Root Cause | Fix |
|---|---|---|---|
| 404 | `Integration ID/Name does not exist` | Wrong integration ID or status mismatch | Must match: (1) same test/live status as secret key, (2) valid and belongs to your account, (3) passed as integer not string, (4) well-configured |
| 400 | `items: { name: ["This field is required."] }` | Missing item name | Add `items[].name` and `items[].amount` |
| 400 | `items: { amount: ["This field is required."] }` | Missing item amount | Same — both name and amount required |

---

## Update Intention

```
PUT https://accept.paymob.com/v1/intention/{client_secret}
Authorization: Token {secret_key}
```

Can update: `amount`, `billing_data`, `special_reference`, `notification_url`.

**Must include** `accept_order_id` in body (the order ID from the original intention response).

| Error | Cause | Fix |
|---|---|---|
| 400 `accept_order_id: ["This field is required."]` | Missing order ID | Pass the order.id from the original intention response |

---

## Split Amount (Marketplace Payouts)

Add to Create Intention body:

```json
{
  "split_amounts": [
    { "mid": "connected_MID_1", "amount_cents": 3000, "description": "Commission" },
    { "mid": "connected_MID_2", "amount_cents": 7000, "description": "Merchant share" }
  ]
}
```

**Requires account configuration** — contact support@paymob.com.

---

## Split Payment (Multiple Cards)

Add to Create Intention body:

```json
{
  "split_payment_methods": [1234]
}
```

Where `1234` is an Auth integration ID. Customer can use up to 3 cards.
**Requires account configuration.**

---

## Auth Token (for Subscriptions, QuickLinks, Transaction Inquiry)

```
POST https://accept.paymob.com/api/acceptance/oauth/token
Content-Type: application/json

{
  "api_key": "your_api_key"
}
```

Response: `{ "access_token": "...", "expires_in": 3600 }`

- Valid for **1 hour**. Generate a fresh one per session.
- Pass as: `Authorization: Bearer {access_token}`
- API key is same for test and live environments
- ⚠ The auth token endpoint path differs between Paymob environments. If `/api/acceptance/oauth/token` returns 404, try `/oauth/token` or check the Paymob Dashboard for your region's correct URL.

---

## Anti-Patterns

- **Do not hardcode integration IDs** — they differ per environment (test/live). Load from env config.
- **Do not cache client_secret** — it's one-time use. Every payment attempt needs a fresh intention.
- **Do not pass integration_id as string** — must be integer. `"1234"` will fail, `1234` works.
- **Do not reuse the same intention for retries** — create a new one with updated `special_reference`.
