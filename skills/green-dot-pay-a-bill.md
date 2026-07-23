---
name: Pay a bill for an account holder
description: >-
  Create a payee and submit a bill payment from a Green Dot Arc embedded account,
  then track the payment. Use when adding BillPay to a partner app.
api: Green Dot Arc BaaS
docs: https://developer.greendot.com/embedded-finance/docs/billpay-1
operations:
  - account_get_getaccount
  - billpay_post_createpayee
  - billpay_post_createpayment
---

# Pay a bill for an account holder

Add a payee and pay a bill from an existing embedded account using the Green Dot
Arc BillPay API.

## Preconditions
- Valid OAuth 2.0 Bearer token and `programCode`.
- A funded `accountIdentifier` (GUID).

## Steps
1. **Confirm the account** — `account_get_getaccount` to verify status and
   available balance before creating a payment.
2. **Create the payee** — `billpay_post_createpayee`. `payeeType` must be
   `Merchant` or `Person`. For a `Merchant` payee provide `merchantId`; for a
   `Person` payee provide full address/contact info (they are not in Green Dot's
   vendor directory). The `accountIdentifier` must be a valid GUID and the
   `X-GD-RequestId` header is required for idempotency and tracking. The response
   returns the new `payeeIdentifier` and status.
3. **Submit the payment** — `billpay_post_createpayment` referencing the
   `payeeIdentifier`, amount, and schedule. Supply a fresh `X-GD-RequestId`.

## Conventions & error handling
- Reuse the same `X-GD-RequestId` + identical payload to safely retry a payment
  after a timeout; a changed payload is a new request.
- Retry only on `503` with exponential backoff (max 3); never on `400`/`500`.
- Subscribe to `Bill Payment Events` webhooks
  (`asyncapi/green-dot-webhooks.yml`) for debit/failed-debit/credit-due outcomes.
