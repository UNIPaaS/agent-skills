# Security and PCI

**Keep the private key server-side.** It is your secret password and must never reach the browser. Every
call that authenticates with it (create-payment, Pay-in, captures, voids, refunds, accounts, payouts,
transfers, vendor onboarding, webhook config) runs server-to-server.

For client-side platform embeds, exchange the private key for a short-lived, scoped /authorize access
token and put that token on the client, never the key. Request only the scopes the surface needs (usually
tied to a vendorId); the token is short-lived, so refresh it server-side rather than widening its scope or
lifetime. A public zero-state token covers the pre-onboarding case where no vendor exists yet. Buyer
payment surfaces use the per-checkout sessionToken instead, which scopes the client to one checkout. See
`references/client-embeds.md` for which surface uses which token. Make write calls idempotent with the
requestId header so a retry cannot double-charge or double-pay.

**Verify authorizations server-side.** A client-side success callback can be manipulated; confirm the
final state from your server before fulfilling.

PCI surface, lowest to highest exposure: hosted Checkout Page and Web SDK secure fields capture card data
in Unipaas-controlled iframed fields (reduced burden); the Pay-in API handles raw card data and requires
SAQ D. Tokenization lets you reuse a stored card so you never re-handle the raw PAN.

Webhooks include an HMAC signature header; recompute and compare it before acting on a webhook. Respond
200 within 10 seconds and do the work asynchronously; Unipaas retries on failure and invalidates the
endpoint after 15 failed attempts.

Fetch for specifics: `/docs/authentication/`, `/docs/api-only-server-to-server/`,
`/docs/store-card-tokenization/`, `/docs/unipaas-webhook-notifications/` (the HMAC signature header and
how to verify it).
