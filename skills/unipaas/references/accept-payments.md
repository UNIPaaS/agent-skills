# Accept Payments

With Unipaas Accept Payments you can take online payments worldwide, choose payment methods per country, store cards for later, accept bank transfers / corporate cards / recurring payments, attach invoices, add due dates and reminders, and get fraud protection. A transaction sent **without a `vendorId`** is recorded as a platform transaction.

## Payment methods

- **Credit cards** — Visa, Mastercard, AMEX, with SCA (3DS2).
- **Pay by Bank / instant bank transfer** — open banking, pay directly from a bank account.
- **Direct Debit** — UK bank-account collections, recurring or one-off (requires mandate).
- **Alternative payment methods** — Apple Pay, Google Pay, Klarna, PayPal, and local methods.

## The three integration tiers

Choose by development effort and PCI posture (different options have different PCI requirements).

| Integration | Effort | Customization | Who it's for |
| --- | --- | --- | --- |
| **Payment Link** | No code (created in portal) | Page hosted by Unipaas; logo/colors customizable | Small/manual platforms |
| **Checkout Page** | Low — auto-create links, redirect buyers to a hosted page | Hosted by Unipaas; logo/colors customizable | Small platforms, limited dev capacity |
| **Web SDK** | Low — your own checkout with embedded secure fields / web-embeds | Style your own page; reduced PCI | Platforms wanting UI control without SAQ D |
| **API Only (Server to Server)** | Mid — build your own form, send raw card data | Full control | **PCI-certified platforms only (SAQ D)** |

Before any integration, complete the **Create Payment** step — it's required for all integration types.

## Create Payment (the checkout call)

This call must run from your backend; it requires your private key. `POST /pay-ins/checkout`.

SaaS platform payload (vendorId sent directly, no items array):

```json
{
  "amount": 100,
  "currency": "GBP",
  "vendorId": "66532df5d55926b2b12a874a",
  "reference": "1000456",
  "description": "Service June",
  "email": "test@test.com",
  "country": "GB",
  "invoiceUrl": "http://yourcompany.com/invoice.pdf",
  "dueDate": "2020-12-13",
  "vatAmount": 19,
  "successfulPaymentRedirect": "http://yourcompany.com/redirect",
  "paymentMethods": ["bankTransfer"],
  "consumer": { "name": "Raul Runte", "reference": "CON-REF-123" }
}
```

Marketplace payload (vendorId inside `items` for split payments):

```json
{
  "amount": 100,
  "currency": "GBP",
  "orderId": "1000456",
  "email": "test@test.com",
  "country": "GB",
  "items": [
    { "name": "Iphone case", "amount": 100, "vendorId": "5ee8e655a65f08fcd71fe4d9", "platformFee": 0, "quantity": 1 }
  ]
}
```

Response includes `shortLink` (hosted checkout page URL), `signedLinkId`, and a **`sessionToken`** (used by the Web SDK / buyer web-embeds — see references/web-sdk.md). The `item` amounts must sum to the total `amount`; `platformFee` sets your fee per vendor.

## The Checkout Page (hosted, low effort)

Reduces PCI burden; fully supports SCA (3DS2). Key object properties: `amount`, `currency`, `orderId`, `email`, `country` (all required), `item[]` (for splitting), `successfulPaymentRedirect` (optional).

Flow: create the checkout, redirect the buyer to `shortLink`. On success the buyer returns to `successfulPaymentRedirect` with Authorization ID, Order ID, Reference ID in the URL.

**Verify server-side** with `GET /pay-ins/{authorizationId}`:

```
curl --location --request GET 'https://sandbox.unipaas.com/platform/pay-ins/{authorizationId}' \
--header 'Authorization: Bearer --PRIVATE_KEY--'
```

Status must be `CAPTURED` for success. Verification ensures no client-side manipulation occurred.

Deactivate a checkout: `POST /pay-ins/checkout/{signedLinkId}/expire` (irreversible; not available for offline methods like Direct Debit).

Apple Pay / Google Pay: contact `support@unipaas.com` to activate. The hosted page auto-displays the right wallet by device/browser; Unipaas processes Google Pay end-to-end (no Google API/console setup needed unless using a native app).

## API Only (server to server, mid effort)

`POST /pay-ins` with raw card data — **requires PCI SAQ D**. Specify `amount`, `currency`, `orderId`, `paymentOption`, `consumer.email`, `consumer.country`; optional `transactionType` (`Auth` or `Sale`, default `Sale`).

```
curl --location --request POST 'https://sandbox.unipaas.com/platform/pay-ins' \
--header 'Authorization: Bearer --YOUR_PRIVATE_KEY--' \
--data-raw '{ "amount": 50, "currency": "EUR", ... }'
```

## Authorization object & statuses

Authorizations are created automatically in the checkout/Web SDK flows; you create them yourself only with the API-only integration. Key statuses: `INITIALIZED`, `AUTHORIZED`, `CAPTURED`, `VOIDED`, `REFUNDED`, `PARTIALLY_*`, `DECLINED`, `PENDING`, `PENDING_3D`, `CANCELLED`, `ERROR`. A successful payment is `CAPTURED`.

## Authorisation & capture

By default payments capture immediately. To separate, send `"transactionType": "Auth"` on `POST /pay-ins/checkout`, then later capture with `POST /pay-ins/{authorizationId}/capture` (body `{"amount":100}`). Authorizations reserve funds, usually for up to seven days.

## Post-payment actions

- **Void** (cancel before capture): `POST /pay-ins/{authorizationId}/void`. Only for `transactionType: "Auth"`, only full amount; use `requestId` to prevent duplicate voids.
- **Refund** (cards only): `POST /pay-ins/{authorizationId}/refund` with `{"amount":100}`. Full or partial, multiple allowed up to original total, requires available balance.
- **Capture**: `POST /pay-ins/{authorizationId}/capture`.
- **Status**: `GET /pay-ins/{authorizationId}`.

## Store cards (tokenization) for future MITs/CITs

On `POST /pay-ins/checkout` set `storePaymentMethod: true` (set `amount: 0` to store without charging) and a unique `reference`. After completion, the `authorization/update` webhook returns `paymentOption.id` (the card token — store for MITs) and `consumerId` (store for CITs).

- **MIT** (charge later, no customer present): `POST /pay-ins` with `paymentOptionId`.
- **CIT** (returning customer chooses saved card): `POST /pay-ins/checkout` with `consumerId`.

Prefer the **Web SDK Store Card flow** over raw API for zero-auth/SCA handling.

## Subscriptions

`POST /pay-ins/plans` to create a plan (name, `period`/`periodUOM`, `autoRenewal`, `pricingModel` `fixed`|`ramp`, `vendorId` or platform ID). Then `POST /pay-ins/checkout` with `plans: [planId]` and `consumer.reference` to create the subscription checkout (returns `shortLink` and `sessionToken`). Handle the `authorization/update` webhook; check `authorizationStatus: "Captured"`, store `subscriptionId`. Update via `PATCH /pay-ins/plans/{planId}` and `PATCH /pay-ins/plans/{planId}/subscriptions/{subscriptionId}`.

## Direct Debit (UK)

Requires a mandate. Recommended: UI Web-Embeds / Mandate Form. Or API: `POST /vendors/{vendorId}/mandates` to request a mandate; collect via `POST /pay-ins/checkout` with `offlinePayment: true` and `paymentMethods: ["directDebit"]`. Mandate statuses: `sent`, `pending_approval`, `active`, `cancelled`, `rejected`, `declined`. Cancel collection: `POST /pay-ins/{authorizationId}/void`. Subscribe to `mandate/update`, `authorization/update`, `ewalletTransaction/create`, `notification/create` webhooks.

## Webhooks

Subscribe via `POST /webhooks`. Key webhooks: `authorization/update` (every authorization/status change — verify by re-fetching `GET /pay-ins/{authorizationId}`), `ewalletTransaction/create`, `onboarding/update`, `payout/update`, `ewallet/create`, `notification/create`, plan/subscription events. Respond `2XX` within 10 seconds.
