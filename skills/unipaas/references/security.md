# Security & PCI

## Key handling — keep the private key server-side

Your **Private Key** (`PRIVATE_KEY` / `<PLATFORM_SECRET_KEY>`) is your secret password. It must be used **only** from your backend and **must never be exposed on the client side**. Any call that authenticates with `Authorization: Bearer {{PRIVATE_KEY}}` — checkout creation, pay-ins, accounts, payouts, onboarding, webhooks management — runs server-to-server.

For client-side platform embeds, exchange the private key for a temporary **access token** via the `/authorize` flow (see references/auth.md). **Keep the access token, never the private key, on the client.** The access token is short-lived (1 hour, auto-refreshing) and scoped, which limits exposure if leaked.

Sandbox and Live credentials are completely separate and share no data. Test with the sandbox key; switch the key and base URL (`api.unipaas.com`) for production.

## What must stay server-side

- The **Private Key** and all calls using it.
- `POST /pay-ins/checkout` (create payment / checkout) — explicitly "must be performed from your backend server"; it requires the private key.
- `POST /pay-ins` (API-only payments), captures, voids, refunds, payouts, transfers, account creation, vendor creation/onboarding submissions, webhook configuration.
- The HMAC **webhook secret** (your platform secret key) used to verify the `X-Hmac-SHA256` header.

## What may run client-side

- The buyer-payment bundles authenticated with the **`sessionToken`** from `POST /pay-ins/checkout`: secure fields (`unipaas.sdk.js`) and buyer web-embeds (`embedded-ui.js` → `unipaas.buyerComponents`). The `sessionToken` is created per-checkout on your server and scopes the client to that one checkout — it is not your private key.
- Platform embeds (`unipaas.components`) authenticated with the **`/authorize` access token**.

## Verify authorizations server-side

Client-side success callbacks can be manipulated. Always confirm the final state from your server with `GET /pay-ins/{authorizationId}`; a successful payment is `CAPTURED`. Treat client events as hints, the server check as truth.

## The PCI surface

- **Checkout Page (hosted)** and **Web SDK secure fields / buyer web-embeds**: card data is captured in Unipaas-controlled secure fields / hosted page, reducing your PCI compliance burden. These fully support SCA (3DS2). Apple Pay and Google Pay also reduce risk (biometric SCA, lower chargebacks).
- **API Only (server to server)**: you **collect and pass raw card data**, which requires assessing your PCI compliance to **SAQ D** — the most extensive self-certification. This method is **not available for non-PCI-certified platforms**.
- **Tokenization**: store cards with `storePaymentMethod: true` and reuse the returned `paymentOption.id` for future MITs/CITs, so you never re-handle raw PAN. Card schemes may require a zero-authorization (`amount: 0`) plus SCA before storing; prefer the Web SDK Store Card flow over implementing 3DS yourself.

## Webhook integrity

Webhooks created via the API include a base64-encoded `X-Hmac-SHA256` header computed from your platform secret key and the body. Recompute the HMAC and compare to verify the request originated from Unipaas before acting on it. Respond `2XX` within 10 seconds; repeated failures (5 → email warning, 15 → invalidation) disable the hook.

## Sensitive-data migration

To migrate card data from another provider, use the Unipaas **PGP public key** to protect the data, coordinate with `support@unipaas.com`, and map old processor IDs to the Unipaas `paymentOptionId` via the `paymentOption/update` webhook.
