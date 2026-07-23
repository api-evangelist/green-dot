---
name: Disburse funds to a recipient
description: >-
  Pay out funds to a recipient with the Green Dot Arc Disbursements API — create
  the payout link (ACH, bank card, or cash), execute the transfer, and confirm
  status. Use for payouts, earned-wage access, and refunds.
api: Green Dot Arc BaaS
docs: https://developer.greendot.com/embedded-finance/docs/disbursements-overview
operations:
  - disbursement_post_createachlink
  - disbursement_post_createbankcardlink
  - disbursement_post_createcashlink
  - disbursement_post_transfer
  - disbursement_get_gettransferstatus
---

# Disburse funds to a recipient

Send money out to a recipient using the Green Dot Arc Disbursements API.

## Preconditions
- Valid OAuth 2.0 Bearer token and `programCode`.
- The `disbursement`, `disbursementReversal`, and related adjustment types are
  configured for your program code (otherwise requests are rejected — contact
  your Green Dot liaison).

## Steps
1. **Create the payout link** for the chosen rail:
   - `disbursement_post_createachlink` — link a recipient bank account for ACH.
   - `disbursement_post_createbankcardlink` — link a recipient debit card.
   - `disbursement_post_createcashlink` — cash pickup / retail cash-out.
2. **Execute the transfer** — `disbursement_post_transfer` (single-phase transfer
   with webhook confirmation). Provide a fresh `X-GD-RequestId`; reuse it with an
   identical payload to safely retry.
3. **Confirm status** — `disbursement_get_gettransferstatus` to read the outcome,
   and/or `disbursement_put_updatetransfer` for maintenance.

## Conventions & error handling
- Balance adjustments backing disbursements (`disbursement`, `achOut`, and their
  reversals) must be configured per program code or the call is rejected.
- Retry only on `503` with exponential backoff (max 3); never on `400`/`500`.
- Prefer `ACH Transfer Events` / `Adjustment Final Status Events` webhooks over
  polling (`asyncapi/green-dot-webhooks.yml`).
