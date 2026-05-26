# Managing Payments: Refund, Void, Capture

Post-payment actions on existing transactions.

---

## Quick Reference

| Action | What It Does | Supported Methods | Same Day Only | Callback Flag |
|---|---|---|---|---|
| **Refund** | Return funds to customer | All except Kiosk | No | `is_refunded: true` |
| **Void** | Cancel before settlement | Cards only | Yes | `is_voided: true` |
| **Capture** | Collect authorized funds | Cards (Auth type) | No | `is_captured: true` |

---

## Refund (API)

Return full or partial payment to the customer.

```
POST https://accept.paymob.com/api/acceptance/refund
Authorization: Token {secret_key}
Content-Type: application/json
```

```json
{
  "transaction_id": 1920364654,
  "amount_cents": 50000
}
```

### Callback

You receive a callback for the **parent** (original) transaction:

```json
{
  "is_refunded": true,
  "refunded_amount_cents": 50000
}
```

The refund transaction itself has its own `id` and includes a `parent_transaction` key pointing to the original transaction.

### Rules

- Refund amount must be ≤ original transaction amount minus already refunded
- Multiple partial refunds allowed (up to full original amount)
- Processing time varies by payment method and issuing bank
- Refund goes back to the **same payment method**

### Common Error

| Error | Fix |
|---|---|
| `Requested Refund Amount is greater than the maximum refund amount` | Pass amount ≤ (original amount - already refunded) |

---

## Void (API)

Cancel a transaction before settlement. Same business day only.

```
POST https://accept.paymob.com/api/acceptance/void
Authorization: Token {secret_key}
Content-Type: application/json
```

```json
{
  "transaction_id": 1920364654
}
```

### Rules

- **Cards only** — other methods don't support void
- **Same business day** — after settlement, use refund instead
- Customer does **not** receive a separate refund — transaction is simply cancelled

### Callback

```json
{
  "is_voided": true
}
```

---

## Capture (API)

Collect authorized funds (see `reference/auth-cap.md` for full flow).

```
POST https://accept.paymob.com/api/acceptance/capture
Authorization: Token {secret_key}
Content-Type: application/json
```

```json
{
  "transaction_id": 1920364654,
  "amount_cents": 50000
}
```

### Rules

- **Cards with Auth type only** — transaction must have `is_auth: true`
- Full or partial capture allowed
- Multiple partial captures allowed (up to auth amount)
- Must capture before authorization expires

### Common Errors

| Error | Fix |
|---|---|
| `Capture amount cannot exceed auth amount` | Pass amount ≤ authorized amount |
| `Invalid transaction id` | Transaction is not Auth or is declined |

### Callback

```json
{
  "is_captured": true,
  "captured_amount": 50000
}
```

---

## Dashboard Operations

All three actions can also be performed from Paymob Dashboard:
1. Go to **Transactions**
2. Find the target transaction
3. Click **Refund**, **Void**, or **Capture**
4. For partial refund/capture: check the partial option and enter amount

---

## Anti-Patterns

- **Do not attempt refund on Kiosk payments** — Kiosk doesn't support refunds.
- **Do not attempt void after settlement** — after settlement, use refund instead.
- **Do not attempt capture on a non-Auth transaction** — only transactions with `is_auth: true` can be captured.
- **Do not assume capture is instant** — settlement timing varies by acquirer and region.
