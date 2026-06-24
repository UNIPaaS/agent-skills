# Client-side embeds and the token boundary

Everything Unipaas renders in the browser. Get the token right first: which token a surface uses is the
single most common integration mistake.

Three tokens, three jobs (never interchange them):

- **Private key** - server only. Authenticates every server-to-server call (create-payment, Pay-in,
  captures, refunds, accounts, payouts, onboarding). Never reaches the browser.
- **`/authorize` access token** - short-lived and scoped. Your server POSTs to `/authorize` with scopes
  (usually a `vendorId`); the browser uses the returned token. This is what **platform embeds** use.
- **`sessionToken`** - per-checkout, returned by the server-side create-payment call. This is what the
  **buyer payment surfaces** use. It scopes the client to one checkout.

## Buyer payment surfaces (sessionToken)

Build your own checkout while card data stays off your page. Both paths take the `sessionToken`, never the
private key and never the `/authorize` token.

- **Secure fields** - load `cdn.unipaas.com/unipaas.sdk.js` (the `Unipaas` global) and call
  `new Unipaas().initTokenize(sessionToken, ...)` to render isolated, iframed card inputs inside your own
  form. Card data never touches your DOM, which reduces PCI burden. Secure fields are a feature of that
  bundle, not a separate npm package (there are no `@unipaas/*` packages).
- **Buyer web-embeds** - load `cdn.unipaas.com/embedded-ui.js` and call
  `unipaas.buyerComponents(sessionToken, config)`, then
  `.create("checkout"|"card"|"digitalWallet").mount(...)` to drop in ready-made, white-label components in
  containers you provide.

## Platform embeds (/authorize access token)

`unipaas.components` renders the platform-side widgets - balance, invoice, onboarding, payPortal,
notification - and authenticates with the `/authorize` access token, not the sessionToken. Mint the token
server-side with the scopes the widget needs.

## Always verify server-side

A client-side success callback is a hint, not proof. After it fires, re-fetch the authorization from your
server and require the captured status before fulfilling.

Fetch for specifics (script URLs, init calls, field config, events, scopes, mount widths):
`/docs/web-sdk/`, `/docs/buyer-ui-embedded-checkout-implementation-guide/`,
`/docs/ui-web-embeds-integration-guide/`, `/docs/handling-dom-events/`, `/docs/authentication/`.
