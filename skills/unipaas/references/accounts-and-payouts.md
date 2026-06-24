# Accounts and payouts

This is the money-movement back half: where collected funds sit, and how they leave.

**Accounts** (the balance objects, called ewallets in the API). Accepted payments land in an account. Every
account has an owner: the platform (no `vendorId`) or a vendor (`vendorId` set). Balances are unique per
owner per currency (GBP, USD, EUR), so a vendor has one balance per currency, never two. The available
portion is the `payable_balance` - only those funds can move. What you do with accounts:

- **Split** - divide a payment across vendor accounts at pay-in time (the `vendorId` on the transaction).
- **Transfer** - move funds from the platform account to a vendor account. One direction only
  (platform to vendor), only from the payable balance, and not reversible once committed.
- **Read balances** - real-time platform and vendor balances.

**Payouts** move a payable balance out to a bank account or other registered payout option, for the
platform or a vendor. Payouts can be on-demand or scheduled (per vendor; vendors can manage their own
schedule if you embed the vendor view). Before a payout you need the account id, the vendor id, and a
registered payout option.

Both transfers and payouts are **two-step: create, then commit or cancel**. Funds are only deducted on
commit. Treat the create call as a reservation and the commit as the irreversible step.

All of this is server-to-server with the private key. Implement the account and payout webhooks so you are
notified of vendor balance and payout activity rather than polling.

Fetch for specifics (objects, endpoints, payloads, scheduling):
- Accounts and transfers: `/docs/ewallets-overview/`, `/docs/transfer-between-ewallets/`.
- Payouts: `/docs/pay-out-funds-overview/`, `/docs/register-new-vendor-payment-option-1/` (create),
  `/docs/commit-cancel-payout/` (commit/cancel), `/docs/get-payouts/`,
  `/docs/edit-vendor-payment-option-1/` (payout options), `/docs/payouts-from-unipaas-portal/` (scheduling).
