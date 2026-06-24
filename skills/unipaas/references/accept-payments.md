# Accept payments

Take a payment from a buyer, optionally split across vendors. A transaction sent without a vendorId is
recorded against the platform account. To split, put each vendor's share in the `items` array with a
per-item `platformFee`, which is a percentage (0-100), not a cash amount.

Every tier starts with the same server-side create-payment call (requires the private key); it returns a
hosted checkout link and a `sessionToken` for client-side surfaces. Pick the tier by dev effort and PCI
posture:

- **Checkout Page** - redirect the buyer to a Unipaas-hosted page. Lowest effort, reduced PCI.
- **Web SDK** - your own page with Unipaas secure fields or buyer web-embeds. UI control, reduced PCI.
- **Pay-in API** - server-to-server with raw card data. Full control, requires PCI SAQ D.

Confirm every payment from your server before fulfilling: re-fetch the authorization and require the
captured status. Client callbacks are hints, the server check is truth.

Fetch for specifics (endpoints, payloads, statuses, fields):
- Create-payment and the tiers: `/docs/create-payment/`, `/docs/checkout-page/`, `/docs/api-only-server-to-server/`.
- Post-payment (void, refund, capture): `/docs/post-payment-actions/`, `/docs/using-authorisation-and-capture/`.
- Save cards, recurring, bank debit: `/docs/store-card-tokenization/`, `/docs/subscriptions/`, `/docs/direct-debit/`.
- Wallets and webhooks: `/docs/apple-pay-and-google-pay/`, `/docs/webhook-guide/`.
