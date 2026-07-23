---
name: Enroll an end user and service their account
description: >-
  Create a new Green Dot Arc BaaS banking relationship for an end user — run
  enrollment with identity capture and KYC, then read the resulting account and
  set up purses. Use when onboarding a new customer into an embedded-finance
  program.
api: Green Dot Arc BaaS
docs: https://developer.greendot.com/embedded-finance/docs/enrollments-api-endpoints
operations:
  - enrollments_post_createenrollment
  - kyc_post_createkyc1
  - account_get_getaccount
  - account_get_getauthcustomerenrollment
  - purse_post_createpurse
---

# Enroll an end user and service their account

Operate the Green Dot Arc Banking-as-a-Service API to open and set up an account
for a new end user. Every call is JSON over OAuth 2.0 and must carry the standard
headers.

## Preconditions
- OAuth 2.0 client-credentials token (`Authorization: Bearer <token>`), obtained
  from `{BaasUrl}/authentication` (see `authentication/green-dot-authentication.yml`).
- Partner IP ranges are allow-listed; field-level encryption keys are exchanged.
- You know your `programCode`.

## Required headers on every call
- `Authorization: Bearer <access_token>`
- `X-GD-RequestId: <new GUID>` — also the idempotency key (see below)
- `Content-Type: application/json`
- Encrypt any field flagged as encrypted in the API reference (identity/PII).

## Steps
1. **Create the enrollment** — `enrollments_post_createenrollment`
   (`POST /programs/{programCode}/enrollments`). Supply identity fields
   (respecting the valid-character rules in `errors/green-dot-error-codes.yml`)
   and a fresh `X-GD-RequestId`. This endpoint is **truly idempotent** and
   **API-locked on `requestId, ssn`** — a retry with the same requestId + same
   payload resumes/returns the same result; a concurrent overlapping call returns
   `409` (code `4091`).
2. **Run KYC** — `kyc_post_createkyc1` (and `kyc_post_createkyc2` / `kyc_post_createoow`
   as the program requires) to complete Know-Your-Customer / OFAC screening.
   Interpret the KYC/OFAC result and act on any cure/gate returned.
3. **Read the account** — `account_get_getaccount`
   (`GET /programs/{programCode}/accounts/{accountIdentifier}`) to confirm status
   and pull `kycPendingGate`. Use `account_get_getauthcustomerenrollment` to read
   enrollment state for the user.
4. **Set up purses** — `purse_post_createpurse` to add savings/spend buckets.
   This endpoint is API-locked on `accountIdentifier`.

## Conventions & error handling
- `termsAcceptanceDateTime` must be within 720 hours of submission (code `620`).
- Retry only on `503`, with exponential backoff (1–1000ms, 1000–5000ms,
  5000–30000ms), max 3 attempts; never retry on `400` or `500`.
- Watch for the `Account Updated` and `User Updated` webhooks
  (`asyncapi/green-dot-webhooks.yml`) rather than polling for status changes.
