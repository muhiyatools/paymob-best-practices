---
name: paymob-best-practices
description: Production-grade Paymob payment gateway integration — intention APIs, unified/embedded checkout, HMAC-secured webhooks, saved cards (CIT/MIT), subscriptions, auth/cap, refund/void/capture, split features, and mobile SDKs. Covers Egypt, KSA, UAE, Oman.
author: muhiyatools
---

# Paymob Best Practices

Production-grade Paymob payment integration. Every line prevents a production incident.

---

## When to Use

The user wants to integrate Paymob payments into a website, mobile app, backend service, or e-commerce platform. Covers API integration, checkout, webhooks, saved cards, subscriptions, auth/cap, refunds, and mobile SDKs.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Your Backend creates an Intention                           │
│     POST https://accept.paymob.com/v1/intention/                │
│     Auth: Token {secret_key}                                    │
│     Returns: { client_secret, order.id }                        │
├─────────────────────────────────────────────────────────────────┤
│  2. Present Checkout Experience                                 │
│     Unified Checkout (redirect): /v1/unifiedcheckout/?csk=...   │
│     Pixel (embedded JS): new Pixel({ publicKey, clientSecret }) │
├─────────────────────────────────────────────────────────────────┤
│  3. Customer completes payment on Paymob                        │
│     Paymob handles UI, 3DS auth, OTP verification               │
│     Browser redirects to your Response Callback URL             │
├─────────────────────────────────────────────────────────────────┤
│  4. Paymob sends Transaction Processed Callback to your server  │
│     POST → your notification_url                                │
│     Includes: id, success, order.id, HMAC signature             │
│     ⚠ YOU MUST VERIFY HMAC BEFORE TRUSTING                     │
├─────────────────────────────────────────────────────────────────┤
│  5. Your server confirms & fulfills the order                   │
│     Use order.id from callback to correlate                     │
│     Transaction Inquiry API as fallback if callback missed      │
└─────────────────────────────────────────────────────────────────┘
```

---

## Decision Tree

```
What's being built?
├── Custom website
│   ├── Want hosted UI?     → Intention API + Unified Checkout (`reference/checkout.md`)
│   ├── Want embedded UI?   → Intention API + Pixel (`reference/checkout.md`)
│   └── Just a payment link?→ QuickLinks API (`reference/quicklink.md`)
├── Mobile app
│   ├── Native iOS           → iOS SDK + backend Intention API (`reference/mobile-sdks.md`)
│   ├── Native Android       → Android SDK + backend Intention API (`reference/mobile-sdks.md`)
│   ├── Flutter              → Flutter bridge + backend Intention API (`reference/mobile-sdks.md`)
│   └── React Native         → React Native SDK + backend Intention API (`reference/mobile-sdks.md`)
└── Recurring billing        → Subscriptions (`reference/subscriptions.md`)

E-commerce? Paymob has native plugins for WooCommerce, Shopify, Magento, Odoo, OpenCart, PrestaShop, WHMCS, CS-Cart, ZenCart, Joomla, Laravel-Bagisto, OsCommerce, Drupal, and Staah — same credential flow as the API.

Need any of these?
├── Saved cards / faster checkout   → reference/saved-cards.md
├── Recurring / subscription billing → reference/subscriptions.md
├── Auth/Cap (hold then capture)     → reference/auth-cap.md
├── Refund / Void / Capture actions  → reference/managing-payments.md
├── Split payments / marketplace     → reference/intention.md (split_amounts section)
├── HMAC / webhook security          → reference/webhooks-hmac.md
└── Test credentials / go-live       → reference/testing.md
```

---

## Anti-Patterns / Hard Bans

These cause production failures. Do not do them.

1. **Do not reuse `client_secret`** — it's one-time use. Create a fresh intention per payment attempt.
2. **Do not mix test/live credentials** — test secret keys only work with test integration IDs, live with live. Mixing returns 404.
3. **Do not trust the browser redirect callback** — Transaction Response Callback can be spoofed. Only the server-side Transaction Processed Callback (POST) is the source of truth.
4. **Do not use WebView for Apple Pay** — Apple Pay is not supported in WebViews. Use native SDKs for Apple Pay in mobile apps.
5. **Do not skip HMAC verification** — every callback must be HMAC-verified with SHA-512. Unverified callbacks can be forged.
6. **Do not use Normal 3DS integration ID for subscriptions** — subscriptions require Moto integration ID.
7. **Do not forget auth token expiry** — auth tokens (for Subscriptions, QuickLinks, Transaction Inquiry) expire in 1 hour. Generate fresh ones.

---

## Command Routing

Match the user's task against these commands:

| If they want to... | Use this file |
|---|---|
| Create/update payment intention | `reference/intention.md` |
| Add checkout to their site | `reference/checkout.md` |
| Handle webhooks / verify HMAC | `reference/webhooks-hmac.md` |
| Implement saved cards | `reference/saved-cards.md` |
| Set up subscriptions | `reference/subscriptions.md` |
| Use Auth/Cap flow | `reference/auth-cap.md` |
| Refund/void/capture | `reference/managing-payments.md` |
| Integrate with mobile app | `reference/mobile-sdks.md` |
| Check regional availability | `reference/payment-methods.md` |
| Test / go live | `reference/testing.md` |
| Create payment links | `reference/quicklink.md` |

---

## Regional Quick Reference

| Method | EGY | KSA | UAE | OMN |
|---|---|---|---|---|
| Cards | Visa, MC, Amex | Visa, MC, Amex, MADA | Visa, MC, Amex | Visa, MC, Amex, Omannet |
| Mobile Wallets | Vodafone Cash, Orange Cash, e& money, We Pay, Bank Wallets | stc pay | — | — |
| Apple Pay | Yes | Yes | Yes | Yes |
| Google Pay | — | Yes | Yes | Yes |
| BNPL | Valu, Souhoola, Tabby, Tamara, Forsa, Sympl, etc. | Tabby, Tamara | Tabby, Tamara | — |
| Bank Installments | Yes | — | — | — |
| Kiosk | Yes | — | — | — |

---

## Integration Credentials

All from Paymob Dashboard → Settings:
- **Secret Key** — server-side API auth (different per test/live)
- **Public Key** — client-side checkout rendering (different per test/live)
- **API Key** — for generating auth tokens (same for both environments)
- **HMAC Secret** — for webhook signature verification
- **Integration IDs** — from Developers → Payment Integrations tab (create separate test/live)

Toggle test/live mode before copying keys — they're different per environment.

---

## Sources

- Official: https://developers.paymob.com/paymob-docs/getting-started/overview
- LLM feed: https://developers.paymob.com/paymob-docs/getting-started/overview/llms.txt
