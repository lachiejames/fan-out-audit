# Audit: apps-api-src-application-billing

**Files audited**

- `apps/api/src/application/billing/errors/billing.errors.ts`
- `apps/api/src/application/billing/services/app-store-notification.service.ts`
- `apps/api/src/application/billing/utils/error-mapper.ts`

---

## No findings

All user-facing text in these files consists of terse, technical error messages intended for developer/API consumers, not end users. No AI writing tropes are present.

**Notes on scope:**

- `billing.errors.ts`: Error messages are short, literal, and factual (`"subscription not found"`, `"Invalid action pack: ${packId}"`, `"Subscription period dates are missing"`). No filler language, no hedging, no corporate softening.
- `app-store-notification.service.ts`: Error strings are all prefixed with `[APPLE]` and describe the specific technical failure (`"Missing x5c certificate chain"`, `"JWS signature verification failed"`, `"Bundle ID does not match"`). Internal/developer-facing only.
- `error-mapper.ts`: Contains no string literals at all. Maps error codes to HTTP status codes.
