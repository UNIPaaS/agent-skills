# Vendor onboarding

Register your sub-merchants (vendors) so they can accept payments and be paid out. Unipaas runs the
KYC/KYB verification; you supply what you already know about the vendor to raise conversion. Two vendor
types: companies (privately held) and individuals / sole traders. Send the `type` on creation so Unipaas
asks for the right fields.

The flow is three steps:

1. **Create the vendor** - a server-to-server call under your platform account (private key), one per user
   who receives money on your platform. Pre-fill known fields (name, email, address) to shorten the form.
2. **Run the onboarding** - pick one of three tiers:
   - **Hosted onboarding link** - no code. Configure branding and defaults in the portal, then send each
     vendor a link by email, SMS, or WhatsApp.
   - **Embedded UI** - launch the onboarding web-embed in your own page and get the outcome via callback.
     This is a platform embed, so it authenticates with the `/authorize` access token (see
     `references/client-embeds.md`), not the private key.
   - **Onboarding API** - build your own front end. Fetch the field set, collect, and submit server-side.
     This tier requires a Unipaas integration review before go-live.
3. **React to the result** - the `onboarding/update` webhook reports the application outcome on submission
   and every status change after.

Onboarding carries three independent statuses: the **onboarding status** (progress, and what the vendor
still needs to provide), the **pay-in status** (whether the vendor may accept payments), and the **pay-out
status** (whether the vendor may withdraw a payable balance). A vendor can be cleared to take payments
before it is cleared to pay out; gate your UI on the specific status, not on "onboarded" in general. The
vendor object carries `acceptPayments` and `receivePayout` booleans for exactly this; gate on those. The
onboarding status itself is not on the vendor object, so track it from the `onboarding/update` webhook.

Fetch for specifics (fields, payloads, the three tiers, statuses, testing):
`/docs/vendor-onboarding-overview/`, `/docs/create-vendor/`, `/docs/onboarding-link/`,
`/docs/onboarding-ui/`, `/docs/onboarding-api/`, `/docs/onboarding-statuses/`,
`/docs/onboardingupdate-webhook/`, `/docs/test-onboarding/`.
