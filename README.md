# paymob-best-practices

An agent skill for Paymob payment integration. Saves you from the same mistakes I made the first time.

## What this is

If you're integrating Paymob into a website, mobile app, or backend, this skill walks you through the entire thing — intention APIs, checkout (redirect or embedded), webhooks with HMAC verification, saved cards, subscriptions, auth/cap, refunds, the works. Covers Egypt, KSA, UAE, and Oman.

It lives inside an AI agent (like OpenCode, Cline, Codex, etc.) and activates automatically when someone asks about Paymob.

## What's inside

- `SKILL.md` — the main file the agent reads. Has the decision tree, architecture flow, hard bans, and a table routing you to the right reference file.
- `reference/` — 11 files, each a deep dive into one area:

| File | What it covers |
|---|---|
| `intention.md` | Create/update intention, split payments, auth token |
| `checkout.md` | Unified Checkout (redirect) + Pixel (embedded JS) |
| `webhooks-hmac.md` | Callback types, HMAC verification with working code |
| `saved-cards.md` | CIT and MIT flows, card token creation |
| `subscriptions.md` | Plans, creation, management, callbacks |
| `auth-cap.md` | Authorize then capture flow |
| `managing-payments.md` | Refund, void, capture APIs + dashboard |
| `mobile-sdks.md` | iOS, Android, Flutter, React Native |
| `payment-methods.md` | Regional availability + constraints table |
| `testing.md` | Test cards, Apple Pay certs, go-live checklist |
| `quicklink.md` | Create and cancel payment links |

## Why you'd want this

Paymob's docs are decent but there's a lot of surface area. The things that'll bite you:

- `client_secret` is one-time use. Don't cache it.
- Test and live credentials are separate. Mixing them gives you a 404.
- The browser redirect callback can be faked. Only trust the server-side one.
- HMAC verification is not optional. Without it, anyone can forge a callback.
- Apple Pay doesn't work in WebViews.
- Subscriptions need a Moto integration ID, not Normal 3DS.
- Auth tokens expire after an hour.

All of these are baked into the skill's guardrails so the agent won't make those mistakes.

## Install

```bash
npx skills add muhiyatools/paymob-best-practices -g -y
```

Then start a new conversation with your agent and ask about Paymob integration.

## License

MIT
