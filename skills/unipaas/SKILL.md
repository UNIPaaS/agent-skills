---
name: unipaas
description: >-
  Integrate Unipaas embedded payments for platforms and marketplaces — sandbox
  setup, the /authorize access-token flow, accepting payments (Checkout Page,
  Web SDK, Pay-in API), client-side buyer checkout via sessionToken, onboarding,
  accounts, payouts, subscriptions, Direct Debit, and webhooks. Use when wiring
  up Unipaas, building a checkout or payment form, tokenizing/saving cards,
  embedding secure card fields (unipaas.sdk.js) or buyer web-embeds
  (embedded-ui.js / unipaas.buyerComponents), platform embeds
  (unipaas.components), onboarding vendors, splitting payments between vendors,
  paying out funds, or handling authorization/onboarding webhooks.
---

# Unipaas

## Foundation

Unipaas provides end-to-end **embedded payments for digital platforms and marketplaces** (SaaS, gig economy, B2B/B2C marketplaces). You onboard vendors (sub-merchants), accept payments, manage accounts (balances), and pay funds out — while controlling money flows and staying PSD2/PCI compliant.

**Get a sandbox key.** Create a free test account at `https://portal.unipaas.com/signup`. The test account is unlimited and always available. Find your **Private Key** in the portal under the developer/keys section. Sandbox and Live credentials are completely separate and share no data.

**Base URLs.**
- Sandbox: `https://sandbox.unipaas.com/platform`
- Production (Live): `https://api.unipaas.com/platform`

**Two auth modes (details in references/auth.md).**
1. **Server-to-server**: send your **Private Key** as `Authorization: Bearer {{PRIVATE_KEY}}` directly on API calls (checkout creation, pay-ins, accounts, payouts, onboarding). The private key must stay server-side.
2. **Two-step `/authorize` flow** (for client-side platform embeds): your server `POST`s the private key to `/authorize` with `scopes` and (usually) a `vendorId`, and receives a temporary `accessToken`. The client uses that access token — never the private key.

**Idempotency.** To avoid duplicate execution on retries, add a `requestId: <id>` header to `POST` requests. The same `requestId` applied multiple times yields a single result. Request IDs are valid for 24 hours.

## Decision map

Four products, three integration tiers. Pick the tier that matches your dev capacity and PCI posture.

| Product | No-code | Low effort | Mid effort |
| --- | --- | --- | --- |
| **Onboarding** | Hosted onboarding link | Embedded UI (`unipaas.components`) | Onboarding API |
| **Accept payments** | Payment Link (portal) | Checkout Page / Web SDK | Pay-in API (server-to-server) |
| **Account management** | Portal view | Account/Balance component | Account API |
| **Pay funds out** | Manual/Scheduled payout via portal | Payout form / Account component | Payouts API |

Guidance:
- **Want zero code?** Use Payment Link, hosted onboarding link, portal payouts.
- **Want your own UI with low effort + reduced PCI?** Use the Checkout Page (redirect), Web SDK secure fields, or buyer web-embeds; use platform embeds for onboarding/balance/portal.
- **Want full control / white-label / native app?** Use the APIs. Pay-in API (raw card data) requires PCI SAQ D.

Note: a transaction sent **without a `vendorId`** is recorded against the platform account.

## Client-side checkout paths

See **references/web-sdk.md** for full steps. Two buyer-payment paths, both authenticated with the **`sessionToken`** returned by a server-side `POST /pay-ins/checkout` — **not** the `/authorize` access token.

1. **Secure-fields SDK** — load `https://cdn.unipaas.com/unipaas.sdk.js` (exposes the `Unipaas` global). Build your own card form, then `new Unipaas().initTokenize(sessionToken, fields, options)`. Secure fields are a feature of this bundle, not a separate package.
2. **Buyer web-embeds** — load `https://cdn.unipaas.com/embedded-ui.js`, then `unipaas.buyerComponents(sessionToken, config)` and `.create("checkout"|"card"|"digitalWallet").mount("#id")`.

**Auth boundary (important):**
- Buyer payment (secure fields and buyer web-embeds) → `sessionToken` from `POST /pay-ins/checkout`.
- **Platform embeds** (`unipaas.components`: balance, invoice, onboarding, payPortal, notification) → the **`/authorize` access token** with scopes + vendorId. Buyer checkout does **not** use the `/authorize` access token.

After any client-side authorization, **verify server-side** with `GET /pay-ins/{authorizationId}`; status must be `CAPTURED` for success.

## References

- **references/auth.md** — the `/authorize` flow, scopes, token lifecycle, idempotency.
- **references/accept-payments.md** — Accept Payments product, the three tiers, the Checkout Page, create-payment, tokenization, webhooks.
- **references/web-sdk.md** — the two client-side buyer-payment paths fed by `sessionToken`.
- **references/security.md** — key handling, what stays server-side, the PCI surface.
