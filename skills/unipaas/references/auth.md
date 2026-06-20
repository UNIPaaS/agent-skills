# Authentication & Authorization

Regardless of how you integrate, you perform at least one server-to-server call to connect with Unipaas.

## Base URLs

- Sandbox: `https://sandbox.unipaas.com/platform`
- Live: `https://api.unipaas.com/platform`

Sandbox and Live credentials are completely separate and share no data. Use the Private Key from your portal (developer/keys). Currently only the Private Key is needed.

## Mode 1 — Server to server (Private Key)

Used by: hosted onboarding link, checkout page, Pay-in API, Account API, Payout API.

Send the private key as a bearer token. Example checkout create:

```
curl --location --request POST 'https://sandbox.unipaas.com/platform/pay-ins/checkout' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {{PRIVATE_KEY}}' \
--data-raw '{
    "amount": 10,
    "currency": "GBP",
    "orderId": "1000456",
    "email": "test@test.com",
    "country": "GB"
}'
```

The private key is your secret password and must never be exposed on the client side.

## Mode 2 — The /authorize flow (client to server)

Used for client-side **platform embeds** (the `unipaas.components` drop-in UIs: onboarding, balance/account, pay portal, notification, invoice). For a client application to talk to Unipaas without exposing the private key, an OAuth 2.0 mechanism issues a temporary `accessToken`.

Two steps:
1. **Server to server** (using your private key) → receive an `accessToken`.
2. **Client side** → use the `accessToken` to run the JavaScript embed.

Request:

```
curl --request POST \
  --url https://sandbox.unipaas.com/platform/authorize \
  --header 'Accept: application/json' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <PLATFORM_SECRET_KEY>' \
  --data-raw '{
      "scopes": ["ewallet_read", "payout_write"],
      "vendorId": "{vendorId}"
   }'
```

Response:

```json
{
    "accessToken": "{{VENDOR_ACCESS_TOKEN}}",
    "expiresIn": 3600,
    "scopes": ["ewallet_read"],
    "vendorId": "60c9a73bf51a8b4bd2034c13",
    "merchantId": "60c9a73bce55c1202708e4cc"
}
```

Notes:
- If you omit `vendorId`, a **public access token** is issued (used for the "zero state" of embeds, before a vendor exists).
- For Embedded Acquisition you may pass a non-registered `vendor` object (with `reference` and basic data) instead of `vendorId`.

### Token lifecycle

- Each access token is valid for **1 hour** (`expiresIn` is in seconds; shown as `3600`, and `86400` in some examples).
- The token **auto-refreshes** every time an embed communicates with the Unipaas servers.
- To swap a token in already-mounted embeds, use `components.reset({ accessToken: "<newAccessToken>" })`.

**Important:** the `/authorize` access token is for platform embeds. Buyer checkout (secure fields / buyer web-embeds) uses the `sessionToken` from `POST /pay-ins/checkout`, not this access token.

## Scopes

All-Components convenience scopes:

| Scope | Description |
| --- | --- |
| `portal_read` | All GET permissions for using UI Web-Embeds |
| `portal_write` | All POST permissions for using UI Web-Embeds |

Granular scopes:

| Scope | Description |
| --- | --- |
| `onboarding_read` / `onboarding_write` | Onboarding component GET / POST |
| `ewallet_read` | Balance/account component GET |
| `payout_read` / `payout_write` | Payout features GET / POST |
| `link_read` / `link_write` | Balance component extra features |
| `notification_read` / `notification_write` | Notification component |
| `invoice_read` / `invoice_write` | Invoice component |
| `direct_debit_read` / `direct_debit_write` | Mandate Form / Direct Debit embed |

The `sessionToken` issued by `POST /pay-ins/checkout` carries buyer-side scopes such as `websdk_access` (and `direct_debit_read`/`direct_debit_write` for DD).

## Idempotency

The API supports idempotent requests so a request isn't executed multiple times due to retries/errors. Add header:

```
requestId: <id>
```

to `POST` requests. The same request applied multiple times does not change the result beyond the initial request (e.g. one transaction, not many). Request IDs are valid for 24 hours. Recommended for voids, captures, payments to prevent duplicates.

## Webhook verification

Webhooks created via the API include a base64-encoded `X-Hmac-SHA256` header computed with your `<PLATFORM_SECRET_KEY>`. Compute the HMAC digest of the body and compare to the header to confirm the webhook came from Unipaas. Respond `2XX` within 10 seconds; after 5 failures you get an email warning, after 15 the hook is invalidated.
