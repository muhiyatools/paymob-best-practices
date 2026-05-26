# Payment Methods & Regional Availability

Complete reference for which payment methods work in which region and their constraints.

---

## Cards

| Aspect | Detail |
|---|---|
| **Regions** | EGY, KSA, UAE, OMN |
| **Networks** | Visa, Mastercard, Amex (+ MADA in KSA, + Omannet in OMN) |
| **Types** | Normal 3DS, Moto, Card On File, Auth/Cap, Verification |
| **Actions** | Void, Refund (full/partial), Capture (full/partial for Auth) |
| **Integration** | APIs, Mobile SDKs, Plugins, Payment Links |
| **Test mode** | Yes |

### Card Type Comparison

| Type | OTP | CVV | Use |
|---|---|---|---|
| Normal 3DS | Yes | Yes | Standard customer checkout |
| Moto | Yes | Yes | Back-to-back merchant charges |
| Card On File | Yes | No | Saved card CIT flow |
| Auth/Cap | Yes | Yes | Reserve then capture |
| Verification | No | No | Validate card without charging |

---

## Mobile Wallets

| Aspect | Detail |
|---|---|
| **Regions** | EGY, KSA only |
| **EGY providers** | Vodafone Cash, Orange Cash, e& money, We Pay, Bank Wallets |
| **KSA providers** | stc pay |
| **Actions** | Refund (full/partial) |
| **Integration** | APIs, Mobile SDKs, Plugins, Payment Links |
| **Test mode** | Yes |

---

## Apple Pay

| Aspect | Detail |
|---|---|
| **Regions** | EGY, KSA, UAE, OMN |
| **Actions** | Refund (full/partial) |
| **Integration** | APIs, Mobile SDKs, Plugins, Payment Links |
| **Test mode** | **No** — live integration IDs only |
| **Requirements** | Apple Pay certificates + domain verification for web; certificates only for SDK |

---

## Google Pay

| Aspect | Detail |
|---|---|
| **Regions** | KSA, UAE, OMN (not EGY) |
| **Actions** | Refund (full/partial) |
| **Integration** | APIs, Mobile SDKs, Plugins, Payment Links |
| **Test mode** | **No** — live integration IDs only |

---

## BNPL (Buy Now, Pay Later)

| Provider | Region | Refunds | Integration Channels |
|---|---|---|---|
| Valu | EGY | Full/Partial | APIs, SDKs, Plugins, Links |
| Souhoola | EGY | Full/Partial | APIs, SDKs, Plugins, Links |
| Aman Installments | EGY | Full/Partial | APIs, Plugins, Links |
| Forsa | EGY | Full/Partial | APIs, SDKs, Plugins, Links |
| Sympl | EGY | — | APIs, SDKs, Plugins, Links |
| Contact | EGY | Full/Partial | APIs, SDKs, Plugins, Links |
| Premium | EGY | — | — |
| Tabby | KSA, UAE | Full/Partial | APIs, SDKs, Plugins, Links |
| Tamara | KSA, UAE | **Full only** (no partial) | APIs, SDKs, Plugins, Links |

**How BNPL works**: Customer selects BNPL at checkout → you get paid upfront → customer repays in installments to the BNPL provider.

---

## Bank Installments

| Aspect | Detail |
|---|---|
| **Regions** | EGY only |
| **Actions** | Refund (full/partial) |
| **Integration** | APIs, Plugins, Payment Links |
| **Test mode** | **No** — live integration IDs only |

---

## Kiosk

| Aspect | Detail |
|---|---|
| **Regions** | EGY only |
| **How it works** | Customer generates reference → pays cash at kiosk → payment confirmed async |
| **Actions** | **No refunds** |
| **Integration** | APIs, Plugins, Payment Links |
| **Test mode** | Yes |

---

## Live-Only Methods

These methods have **no test integration IDs**. You can only test them in production mode:

```
✗ Apple Pay        — live only
✗ Google Pay       — live only
✗ Bank Installments — live only
```

---

## Constraint Summary

| Constraint | Methods |
|---|---|
| Cannot be refunded | Kiosk |
| Cannot be partially refunded | Tamara (full refund only) |
| Void supported | Cards only |
| Capture supported | Cards (Auth type only) |
| EGY only | Bank Installments, Kiosk |
| KSA, UAE, OMN only | Google Pay |
| EGY, KSA only | Mobile Wallets |
