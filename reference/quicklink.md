# QuickLinks API

Programmatically create Paymob-hosted payment links. Share via URL — no checkout integration needed.

---

## Create QuickLink

```
POST https://accept.paymob.com/api/acceptance/quicklinks
Authorization: Bearer {auth_token}
Content-Type: application/json
```

```json
{
  "amount_cents": 100000,
  "currency": "EGP",
  "reference_id": "order_12345_unique",
  "integration_ids": [1234, 5678],
  "expires_at": "2026-06-30T23:59:59Z",
  "is_live": false,
  "description": "Payment for Order #12345"
}
```

### Required Fields

| Field | Type | Notes |
|---|---|---|
| `amount_cents` | integer | Amount in cents |
| `currency` | string | EGP, SAR, AED, OMR |
| `reference_id` | string | **Must be unique per merchant** |
| `integration_ids` | array[int] | Payment method integration IDs |
| `is_live` | boolean | `false` for test, `true` for live |

### Optional Fields

| Field | Type | Notes |
|---|---|---|
| `expires_at` | datetime | Must be in the future |
| `description` | string | Displayed on the payment page |

### Common Errors

| Error | Fix |
|---|---|
| `Reference ID already exists` (400) | Use a unique `reference_id` each time |
| `incorrect credentials` (401) | Auth token expired (1h TTL) |
| `expires_at can't be in the past` (400) | Set expiry date in the future |
| `Integration ID does not exist` (404) | Integration ID status must match `is_live` value |

### Response

```json
{
  "id": 123456,
  "url": "https://accept.paymob.com/quicklink/abc123",
  "reference_id": "order_12345_unique"
}
```

Share the `url` with your customer.

---

## Cancel QuickLink

```
DELETE https://accept.paymob.com/api/acceptance/quicklinks/{id}
Authorization: Bearer {auth_token}
```

Cancels the link — future payment attempts will fail.

---

## Supported Payment Methods

QuickLinks work with: Cards, Mobile Wallets, Apple Pay, Google Pay, BNPL, Bank Installments, Kiosk.

---

## Anti-Patterns

- **Do not rely on QuickLink redirect alone for payment confirmation** — use Transaction Processed Callback as source of truth.
- **Do not reuse `reference_id`** — each QuickLink needs a unique reference.
- **Do not leak your auth token** — it has access to create and cancel payment links.
- **Do not set `is_live` incorrectly** — test payments must use `is_live: false` with test integration IDs.
