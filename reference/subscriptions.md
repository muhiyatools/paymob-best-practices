# Subscriptions

Automated recurring billing. Paymob creates a subscription, saves the customer's card via 3DS, then handles recurring charges automatically.

---

## Prerequisites

- **Moto integration ID** — subscriptions must use Moto card integration, not Normal 3DS
- **Auth Token** — subscription management APIs use Bearer auth token, not secret key

---

## Architecture

```
Create Plan (API) → Create Subscription via Intention → Customer 3DS → Paymob handles recurring billing
                              ↓
                    You receive callbacks at plan's webhook_url
```

---

## Step 1: Create Subscription Plan

```
POST https://accept.paymob.com/api/acceptance/subscriptions/v1/plans
Authorization: Bearer {auth_token}
Content-Type: application/json
```

```json
{
  "name": "Monthly Premium",
  "amount_cents": 29900,
  "currency": "EGP",
  "frequency": 30,
  "periods": 12,
  "moto_integration_id": 1234,
  "webhook_url": "https://your-server.com/paymob/subscription-callback"
}
```

| Field | Required | Notes |
|---|---|---|
| `name` | Yes | Plan name |
| `amount_cents` | Yes | Billing amount in cents |
| `currency` | Yes | EGP, SAR, AED, OMR |
| `frequency` | Yes | **Valid values only**: 7, 15, 30, 60, 90, 180, 360 |
| `periods` | Yes | Number of billing cycles (null = unlimited) |
| `moto_integration_id` | Yes | Must be a Moto card integration ID |
| `webhook_url` | Yes | Where subscription callbacks are sent |

### Common Errors

| Error | Fix |
|---|---|
| `"5" is not a valid choice` for frequency | Use: 7, 15, 30, 60, 90, 180, or 360 |
| `incorrect credentials` (401) | Auth token expired (1h TTL) — generate fresh one |

---

## Step 2: Create Subscription (via Intention)

Add subscription fields to the Create Intention request:

```json
{
  "amount": 29900,
  "currency": "EGP",
  "payment_methods": [1234],
  "items": [{"name": "Monthly Premium", "amount": 29900, "quantity": 1}],
  "billing_data": { /* ... */ },
  "subscription_plan_id": 1186,
  "subscription_start_date": "2026-06-01"
}
```

- Customer completes **one 3DS transaction** to save their card
- Subscription is created and first billing occurs according to schedule
- You receive Transaction Processed Callback for the initial payment
- Subsequent billings happen automatically — you receive callbacks at plan's `webhook_url`

---

## Step 3: Manage Subscriptions (API)

All use `Authorization: Bearer {auth_token}`.

### Update Plan

```
PUT https://accept.paymob.com/api/acceptance/subscriptions/v1/plans/{plan_id}
```
Can update: amount, periods, moto_integration_id.

### List Plans

```
GET https://accept.paymob.com/api/acceptance/subscriptions/v1/plans
```

### Suspend Plan

```
POST https://accept.paymob.com/api/acceptance/subscriptions/v1/plans/{plan_id}/suspend
```

### Resume Plan

```
POST https://accept.paymob.com/api/acceptance/subscriptions/v1/plans/{plan_id}/resume
```

### Update Subscription

```
PUT https://accept.paymob.com/api/acceptance/subscriptions/v1/subscriptions/{subscription_id}
```
Can update: amount, end_date.

### List Subscriptions

```
GET https://accept.paymob.com/api/acceptance/subscriptions/v1/subscriptions
```
Query params: `plan_id`, `state` (active, suspended, cancelled).

### Suspend Subscription

```
POST https://accept.paymob.com/api/acceptance/subscriptions/v1/subscriptions/{id}/suspend
```

### Resume Subscription

```
POST https://accept.paymob.com/api/acceptance/subscriptions/v1/subscriptions/{id}/resume
```

| Error | Fix |
|---|---|
| `The subscription isn't suspended` | Can't resume a non-suspended subscription |
| `Subscription not found` | Invalid subscription ID |

### Cancel Subscription

```
POST https://accept.paymob.com/api/acceptance/subscriptions/v1/subscriptions/{id}/cancel
```

Permanent termination. Cannot be undone.

### List Subscription Cards

```
GET https://accept.paymob.com/api/acceptance/subscriptions/v1/subscriptions/{id}/cards
```

### Add Secondary Card

Create an Intention with `subscriptionv2_id` field set to the subscription ID:

```json
{
  "subscriptionv2_id": 7923,
  // ... other intention fields
}
```

Customer completes checkout and saves their new card. Callback arrives with `trigger_type: "add_secondry_card"`.

### Change Primary Card

```
POST https://accept.paymob.com/api/acceptance/subscriptions/v1/subscriptions/{id}/change_card
{"card_id": 8555026}
```

### Delete Secondary Card

```
DELETE https://accept.paymob.com/api/acceptance/subscriptions/v1/subscriptions/{id}/cards/{card_id}
```

### List Subscription Transactions

```
GET https://accept.paymob.com/api/acceptance/subscriptions/v1/subscriptions/{id}/transactions
```

### Register Webhook

```
POST https://accept.paymob.com/api/acceptance/subscriptions/v1/subscriptions/{id}/webhook
{"webhook_url": "https://your-server.com/paymob/subscription-callback"}
```

---

## Subscription Callbacks

Sent to the plan's `webhook_url` (or subscription-specific registered webhook).

**HMAC is in the request body** (not query param), key name `hmac`.

| `trigger_type` | Meaning |
|---|---|
| `suspended` | Subscription was suspended |
| `resumed` | Subscription was resumed |
| `cancelled` | Subscription was cancelled |
| `billing` | Recurring billing charge processed |
| `add_secondry_card` | Secondary card added |
| `change_primary_card` | Primary card changed |

Key subscription callback fields: `subscription_data.id`, `subscription_data.state`, `subscription_data.next_billing`, `trigger_type`.

---

## Anti-Patterns

- **Do not use Normal 3DS integration ID for subscriptions** — must be Moto type.
- **Do not reuse auth tokens across sessions without checking expiry** — token expires in 1 hour.
- **Do not forget to set `webhook_url` on the plan** — without it, you won't receive subscription state changes.
- **Do not use secret key for subscription management APIs** — they require Bearer auth token.
- **Do not confuse plan ID with subscription ID** — plan defines terms, subscription is an enrollment.
