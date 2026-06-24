---
name: unipaas
description: Building, wiring, or reviewing a Unipaas embedded-payments integration for a platform or marketplace - the /authorize access-token flow, accepting payments (Checkout Page, Web SDK secure fields, Pay-in API), client-side buyer checkout, vendor onboarding, accounts, payouts, subscriptions, Direct Debit, and webhooks. Use when wiring up Unipaas, building a checkout or payment form, embedding secure card fields or platform embeds, onboarding vendors, splitting or paying out funds, or handling authorization and onboarding webhooks.
---

# Build on Unipaas

Use this when building, modifying, or reviewing a Unipaas integration. This skill maps the surface and
the rules; fetch the live docs for exact endpoints, parameters, and payloads. Unipaas is embedded
payments for platforms and marketplaces: you onboard vendors (sub-merchants), accept payments, hold
balances in accounts, and pay funds out, with the money split you choose.

Sandbox and Live credentials are separate and share no data. Start in sandbox; get a key from the portal
and read `/docs/getting-started/` and `/docs/authentication/` before writing code.

## The integration map

Four products, each with a no-code, an embedded-UI, and an API tier. Pick the tier that matches your dev
capacity and PCI posture.

- **Accept payments** - take a payment from a buyer, optionally split across vendors. Hosted Checkout
  Page, Web SDK secure fields, or the server-to-server Pay-in API. See `/docs/accept-payments-overview/`.
- **Vendor onboarding** - register sub-merchants so they can transact and be paid. Hosted link, embedded
  onboarding UI, or the Onboarding API. See `/docs/vendor-onboarding-overview/`.
- **Accounts** - the balances that hold collected funds per vendor and per platform. See
  `/docs/ewallets-overview/`.
- **Payout funds** - move settled balances out to vendors. See `/docs/pay-out-funds-overview/`.

Two client-side embed families sit on top: buyer payment embeds (secure fields via `unipaas.sdk.js`,
buyer web-embeds via `unipaas.buyerComponents`), both fed by a checkout `sessionToken`; and platform
embeds via `unipaas.components`, fed by an `/authorize` access token. See `/docs/ui-web-embeds/`.

## Routing table

| Building... | Surface | Fetch |
| --- | --- | --- |
| Authorize a client-side platform embed | /authorize access token | /docs/authentication/ |
| Create a payment / checkout session | Pay-in (server-side) | /docs/create-payment/ |
| Hosted, low-effort checkout | Checkout Page (redirect) | /docs/checkout-page/ |
| Your own card form, reduced PCI | Web SDK secure fields | /docs/web-sdk/ |
| Full control, raw card data (SAQ D) | Pay-in API | /docs/api-only-server-to-server/ |
| Buyer-side embedded checkout UI | Buyer web-embeds | /docs/buyer-ui-embedded-checkout-implementation-guide/ |
| Platform embeds (balance, onboarding, payPortal) | unipaas.components | /docs/ui-web-embeds-integration-guide/ |
| Save a card for later MIT/CIT | Tokenization | /docs/store-card-tokenization/ |
| Recurring billing | Subscriptions | /docs/subscriptions/ |
| Direct Debit collections | Direct Debit | /docs/direct-debit/ |
| Auth-then-capture | Authorisation and capture | /docs/using-authorisation-and-capture/ |
| Onboard a vendor | Vendor onboarding | /docs/create-vendor/ |
| Pay a vendor out | Payouts | /docs/pay-out-funds-overview/ |
| React to events server-side | Webhooks | /docs/webhook-guide/ |
| Sandbox test cards and scenarios | Testing | /docs/test-cards/ |

## Critical rules (always apply)

- The secret/private key stays server-side. Never put it in the browser.
- Authorization is two steps: POST /authorize to mint a short-lived scoped token, then call with it.
- Web SDK secure fields are iframed; card data never touches your page.
- Amounts are in the major currency unit, not the minor one: `amount: 150` means 150.00 (pounds/euros), never 150 pence/cents.

These hold regardless of tier. The buyer-payment embeds authenticate with the per-checkout
`sessionToken` from the server-side create-payment call, not the private key and not the `/authorize`
token. Always confirm a payment from your server before fulfilling: re-fetch the authorization and treat
a client-side success callback as a hint, the server status as truth.

## References

- `references/accept-payments.md` - the three tiers, create-payment, tokenization, subscriptions, Direct Debit, post-payment.
- `references/client-embeds.md` - the three tokens and which surface uses which; secure fields, buyer embeds, platform embeds.
- `references/vendor-onboarding.md` - registering vendors, the three onboarding tiers, KYC/KYB, and the onboarding statuses.
- `references/accounts-and-payouts.md` - where funds sit (balances), transfers, and paying funds out (two-step create then commit).
- `references/security.md` - key handling, the /authorize token, the PCI surface, webhook HMAC.

## Trust the live docs

For exact endpoints, parameters, limits, and current code, fetch the per-page markdown at
`/docs/<slug>.md` (or the whole corpus at `/llms-full.txt`). When this skill and the docs disagree,
trust the docs. This skill is a map and a small set of stable rules; the live docs are the source of
truth and version with the API.
