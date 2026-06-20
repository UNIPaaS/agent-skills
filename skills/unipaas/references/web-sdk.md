# Web SDK — client-side buyer payment

The Web SDK lets you build your own checkout while reducing PCI burden (full SCA / 3DS2 support). There are two buyer-payment paths. **Both are authenticated with the `sessionToken` returned by a server-side `POST /pay-ins/checkout`** — never the `/authorize` access token (that token is for platform embeds: balance, invoice, onboarding, payPortal).

## Step 0 — Create the checkout (server side) and get the sessionToken

This call requires your private key and must run on your backend.

```
curl --location --request POST 'https://sandbox.unipaas.com/platform/pay-ins/checkout' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {{PRIVATE_KEY}}' \
--data-raw '{
  "amount": 60,
  "currency": "EUR",
  "orderId": "dfgdfh4366",
  "description": "iphone accessories",
  "email": "test@unipaas.com",
  "country": "GB",
  "items": [
    { "name": "iphone case", "amount": 50, "vendorId": "5ee8e655a65f08fcd71fe4d9", "platformFee": 15 }
  ]
}'
```

The response contains a **`sessionToken`** field — keep it for the client-side steps below.

---

## Path 1 — Secure-fields SDK (`unipaas.sdk.js`)

Card fields live in your own form; the bundle's "secure fields" render isolated, PCI-reducing inputs. Secure fields are a feature of this bundle, not a separate npm package.

**Embed the script** (in `<head>`):

```html
<script src="https://cdn.unipaas.com/unipaas.sdk.js"></script>
<link rel="stylesheet" href="https://cdn.unipaas.com/style.css">
```

This exposes the global `Unipaas`.

**Add containers** for the mandatory fields (single unified card field recommended, or separate fields), plus a cardholder field and a submit button:

```html
<div class="payment-field">
  <label>Card details <div id="card"></div></label>
</div>
<div class="payment-field">
  <label>Cardholder name <div id="holdername"></div></label>
</div>
<button id="submit-form">Pay</button>
```

**Initialize** (end of `<body>`):

```html
<script>
  var unipaas = new Unipaas();
  var SESSION_TOKEN = 'TODO: sessionToken from POST /pay-ins/checkout';

  unipaas.initTokenize(
    SESSION_TOKEN,
    {
      // unified field:
      cardDetails: { selector: '#card', placeholder: { cardNumber: "1234 5678 9012 3456", cvv: "CVV", expiry: "MM/YY" } },
      // OR separate fields: cardNumber / cvv / expiry with their selectors
      cardHolder: { selector: "#holdername", placeholder: "A. Einstein" }
    },
    {
      additionalFields: { submitButton: { selector: "#submit-form" } },
      mode: 'test' // 'test' (sandbox) or 'live' (production)
    }
  );

  unipaas.on('onSuccess', function (data) { /* forward data to your server */ });
  unipaas.on('onError', function (err) { console.log('Error:', err); });
</script>
```

Events: `onSubmission`, `OnTokenSuccess` (keep `paymentOptionId`), `onSuccess` (authorization object with `authorizationId`, `consumerId`, `paymentOption`), `onError`.

**Apple Pay / Google Pay** in this bundle: add a single container `<div id="applepay_googlepay"></div>` and pass `paymentComponents` in the init options (Apple Pay prioritized due to OS restrictions):

```js
paymentComponents: [
  { type: "applePay",  params: { selector: "#applepay_googlepay" }, priority: true, renderImmediately: true },
  { type: "googlePay", params: { selector: "#applepay_googlepay" }, renderImmediately: true }
]
```

**Store card / returning consumer:** to charge a saved card, create a new checkout with `consumerId`, get a new `sessionToken`, then:

```js
unipaas.payWithToken(SESSION_TOKEN, { mode: 'test' });
// then anywhere on the page:
unipaas.makePayment('60a66fd0b3d40d819beb17ea'); // the paymentOptionId
```

**Always verify server-side**: `GET /pay-ins/{authorizationId}` — status must be `CAPTURED`.

---

## Path 2 — Buyer web-embeds (`embedded-ui.js` / `unipaas.buyerComponents`)

White-labeled, ready-made checkout components. Keep this path available alongside Path 1.

**Embed the script** (in `<head>`):

```html
<script type="application/javascript" src="https://cdn.unipaas.com/embedded-ui.js"></script>
```

**General configuration** (below `</body>`), authenticated with the `sessionToken`:

```html
<script type="text/javascript">
  const config_general = {
    theme: {
      type: "light",
      variables: {
        backgroundColor: "#FFFFFF",
        primaryTextColor: "#000000",
        primaryColor: "#2F80ED",
        digitalWalletButtonMode: "black"
      }
    }
  };
  const components = unipaas.buyerComponents("<sessionToken>", config_general);
</script>
```

**Create and mount components** into containers you provide. Mount each component only once per selector.

```html
<div id="checkout"></div>
<div id="card"></div>
<div id="digital_wallet"></div>

<script>
  // Full embedded checkout page (renders configured payment methods)
  const checkout = components.create("checkout");
  checkout.mount("#checkout");

  // Card embed (card fields + cardholder + save-card + pay button)
  const card = components.create("card");
  card.mount("#card");

  // Digital wallet (Apple Pay OR Google Pay, never both — OS rules)
  const digitalWallet = components.create("digitalWallet");
  digitalWallet.mount("#digital_wallet");
</script>
```

Optional: disable Google Pay — `components.create("checkout", { disableGooglePay: true })`.

Minimal widths: checkout 360px, card 340px, digital wallet 150px.

Do **not** load Unipaas checkout inside an iframe — flows like Open Banking need top-level navigation.

**DOM events** (below `</body>`):

```js
components.on("paymentSuccess", (e) => { /* show post-payment view */ });
components.on("paymentError", (e) => {});
components.on("paymentSubmission", (e) => {});
components.on("paymentCancel", (e) => {});
```

Checkout component also supports callbacks, e.g. `onPaymentSuccess` passed in the create config.

**Subscription buyer checkout** uses the same bundle: create the checkout with a `plans` array to get the `sessionToken`, then `components.create("recurring").mount("#recurring")`; listen for `subscriptionCreated` / `subscriptionError`.

---

## Auth summary

| Surface | Bundle / API | Auth token |
| --- | --- | --- |
| Secure card fields | `unipaas.sdk.js` (`new Unipaas().initTokenize`) | `sessionToken` from `POST /pay-ins/checkout` |
| Buyer web-embeds | `embedded-ui.js` (`unipaas.buyerComponents`) | `sessionToken` from `POST /pay-ins/checkout` |
| Platform embeds (balance, invoice, onboarding, payPortal, notification) | `embedded-ui.js` (`unipaas.components`) | `/authorize` access token (scopes + vendorId) |

Buyer checkout never uses the `/authorize` access token.
