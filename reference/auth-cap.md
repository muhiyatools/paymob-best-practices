# Authorization & Capture (Auth/Cap)

Two-step payment flow: **authorize** (reserve funds) → **capture** (settle later).

---

## When to Use

- Final amount unknown at checkout (e.g., shipping costs added later)
- Services confirmed after payment (e.g., booking with variable extras)
- Partial fulfillment (ship items as they become available)
- Want control over settlement timing

---

## Flow

```
1. Create Intention with Auth integration ID
   → Funds held on customer's card (not captured)
   → Callback includes is_auth: true

2. (Later) Call Capture API
   → Full or partial amount
   → Multiple partial captures allowed (up to auth amount)

3. Remaining authorized amount auto-voids after expiry
```

---

## Step 1: Authorization

Create Intention with an **Auth** card integration ID:

```json
{
  "amount": 50000,
  "currency": "EGP",
  "payment_methods": [1234],
  "items": [{"name": "Deposit", "amount": 50000, "quantity": 1}],
  "billing_data": { /* ... */ },
  "notification_url": "https://your-server.com/paymob/callback"
}
```

### Callback

After successful authorization, you receive a Transaction Processed Callback with:

```json
{
  "success": true,
  "is_auth": true,
  "is_captured": false,
  "captured_amount": 0,
  "amount_cents": 50000
}
```

Funds are **reserved** but not transferred to your account.

---

## Step 2: Capture

```
POST https://accept.paymob.com/api/acceptance/capture
Authorization: Token {secret_key}
Content-Type: application/json
```

```json
{
  "transaction_id": 1920364654,
  "amount_cents": 35000
}
```

| Field | Required | Notes |
|---|---|---|
| `transaction_id` | Yes | The Auth transaction ID from Step 1 callback |
| `amount_cents` | Yes | Must be ≤ original authorized amount |

### Full vs Partial Capture

- **Full capture**: pass the original `amount_cents`
- **Partial capture**: pass any amount ≤ authorized amount
- Multiple partial captures are allowed (total must not exceed authorized amount)

### Common Errors

| Error | Cause | Fix |
|---|---|---|
| `Capture amount cannot exceed auth amount` | `amount_cents` > authorized amount | Pass ≤ original auth amount |
| `Invalid transaction id` | Transaction is not Auth type, or is declined, or already fully captured | Use a successful Auth transaction ID |

### Callback

After capture, you receive a callback for the parent transaction with:

```json
{
  "is_captured": true,
  "captured_amount": 35000
}
```

---

## Step 3: Auto-Void

Any remaining authorized amount that is not captured **before the authorization expiry** is automatically voided. No action needed — the funds are released back to the customer.

---

## Constraints

| Aspect | Rule |
|---|---|
| Supported payment methods | **Cards only** (with Auth integration type) |
| Capture window | Before authorization expires (typically a few days — check with Paymob) |
| Partial captures | Multiple allowed, cannot exceed total authorized amount |
| Full capture | Capture the exact authorized amount |
| Void of authorized amount | If you need to cancel before capturing, call Void API |

---

## Anti-Patterns

- **Do not use Auth/Cap with non-card payment methods** — only card transactions support Auth/Cap.
- **Do not attempt to capture after authorization expires** — the hold is released. Create a new auth.
- **Do not assume full amount will be captured** — design your system to handle partial captures.
- **Do not forget to check `is_auth: true` in callback** — non-Auth transactions will fail on capture.
