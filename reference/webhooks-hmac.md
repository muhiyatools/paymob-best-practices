# Webhooks & HMAC

Paymob sends webhook callbacks after every payment event. **HMAC verification is mandatory** — without it, callbacks can be forged.

---

## Callback Types

| Type | Method | Where It Goes | Purpose |
|---|---|---|---|
| **Transaction Processed** | POST | Your `notification_url` or integration's Processed Callback URL | Backend notification — this is the source of truth |
| **Transaction Response** | GET (redirect) | Integration's Response Callback URL | Browser redirect — shows customer the result page |
| **Card Token** | POST | Your `notification_url` or integration's Processed Callback URL | Delivers saved card token |
| **Subscription** | POST | Plan's `webhook_url` or registered webhook URL | Subscription state changes |

---

## Transaction Processed Callback (The Source of Truth)

Sent after every successful/declined transaction and after post-payment actions (refund, void, capture).

### Sample Payload

```json
{
  "id": 1920364654,
  "success": true,
  "amount_cents": 100000,
  "currency": "EGP",
  "order": { "id": 2175037543 },
  "is_refunded": false,
  "refunded_amount_cents": 0,
  "is_voided": false,
  "is_captured": false,
  "captured_amount": 0,
  "pending": false,
  "integration_id": 123456,
  "is_3d_secure": true,
  "is_auth": false,
  "is_standalone_payment": true,
  "source_data": {
    "type": "card",
    "sub_type": "MasterCard",
    "pan": "xxxx-xxxx-xxxx-2346"
  },
  "error_occured": false,
  "has_parent_transaction": false,
  "owner": 246628,
  "created_at": "2024-06-13T11:33:44.592345"
}
```

### Critical Fields

| Field | What to Do |
|---|---|
| `id` | Transaction ID — use for refund/void/capture API calls |
| `success` | `true` = payment completed. But only trust after HMAC verification |
| `order.id` | Correlate to your internal order |
| `is_refunded` / `is_voided` / `is_captured` | Check these for post-payment actions |
| `pending` | If `true`, transaction is not yet final |
| `source_data.type` | `card`, `wallet`, etc. |

---

## HMAC Verification Protocol

Every callback includes an `hmac` query parameter (or body field for subscriptions). You must:

1. **Extract values in this exact key order**
2. **Concatenate into one string**
3. **Hash with SHA-512 using your HMAC secret**
4. **Compare to the received hmac value**

### Key Order for Transaction Callbacks (POST)

```
amount_cents
created_at
currency
error_occured
has_parent_transaction
obj.id          ← note: nested key (obj → id)
integration_id
is_3d_secure
is_auth
is_capture
is_refunded
is_standalone_payment
is_voided
order.id        ← note: nested key (order → id)
owner
pending
source_data.pan ← note: nested key (source_data → pan)
source_data.sub_type
source_data.type
success
```

### Key Order for Transaction Callbacks (GET / Response)

Same as POST but:
- `obj.id` → `id`
- `order.id` → `order_id`

### Critical: HMAC Key Names vs JSON Field Names

Some HMAC key names differ from the actual JSON callback field names. Your implementation must map them:

| HMAC Key | JSON Field | Reason |
|---|---|---|
| `obj.id` | `id` | Legacy naming — value is at top level, not nested |
| `is_capture` | `is_captured` | HMAC spec uses shorter form |
| `order.id` | `order.id` | Same — nested, direct access works |
| `source_data.pan` | `source_data.pan` | Same — nested, direct access works |

### Implementation (Node.js)

```javascript
const crypto = require('crypto');

function verifyTransactionHmac(data, hmacSecret, receivedHmac) {
  // Maps HMAC spec key names to actual JSON field paths
  const KEY_MAP = {
    'obj.id': 'id',
    'is_capture': 'is_captured',
  };

  const POST_KEYS = [
    'amount_cents', 'created_at', 'currency', 'error_occured',
    'has_parent_transaction', 'obj.id', 'integration_id', 'is_3d_secure',
    'is_auth', 'is_capture', 'is_refunded', 'is_standalone_payment',
    'is_voided', 'order.id', 'owner', 'pending', 'source_data.pan',
    'source_data.sub_type', 'source_data.type', 'success'
  ];

  const concatenated = POST_KEYS.map(key => {
    const jsonPath = KEY_MAP[key] || key;
    const parts = jsonPath.split('.');
    let value = data;
    for (const part of parts) {
      if (value == null || typeof value !== 'object') return '';
      value = value[part];
    }
    return value != null ? String(value) : '';
  }).join('');

  const hash = crypto.createHmac('sha512', hmacSecret)
    .update(concatenated)
    .digest('hex');

  return hash === receivedHmac;
}
```

For GET (Response) callbacks, use the same approach with key mapping: `id` → `id`, `order_id` → `order_id` (instead of `obj.id` and `order.id`).

---

## Card Token Callback HMAC

| Key Order |
|---|
| `card_subtype`, `created_at`, `email`, `id`, `masked_pan`, `merchant_id`, `order_id`, `token` |

Same process: sort → concatenate → SHA-512 → compare.

---

## Subscription Callbacks

HMAC is provided **in the request body** under the key `hmac` (not as a query parameter). Calculate HMAC using the subscription data fields. Refer to Paymob's subscription guide for the exact key ordering.

---

## Implementation Rules

1. **Verify HMAC first** — before any business logic. If HMAC doesn't match, return 401 and log.
2. **Use processed callback as source of truth** — the response (redirect) callback can be spoofed.
3. **Handle duplicates** — Paymob may send the same callback more than once. Make your handler idempotent.
4. **Respond 200 OK quickly** — non-200 responses trigger retries.
5. **Transaction Inquiry API as fallback** — if you miss a callback, use it to fetch transaction data.

---

## Webhook Testing Tool

Use https://webhook.paymob.com/ to:
- Generate a temporary endpoint URL
- Capture real callback payloads for inspection
- Validate your HMAC implementation before going live

---

## Anti-Patterns

- **Do not skip HMAC verification** — unverified callbacks are trivial to forge.
- **Do not confirm payment based on redirect callback alone** — the browser redirect can be tampered with.
- **Do not use SHA-256** — Paymob uses SHA-512 for HMAC.
- **Do not hardcode the HMAC secret** — store it in environment config, rotated per environment.
- **Do not log the full callback body in production** — it contains PII (pan, email, phone).
