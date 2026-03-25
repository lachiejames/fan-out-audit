# Audit: apps-api-src-application-push-notifications-1

Files audited:

- `apps/api/src/application/push-notifications/errors/apns-push.errors.ts`
- `apps/api/src/application/push-notifications/errors/fcm-push.errors.ts`
- `apps/api/src/application/push-notifications/errors/push-notification.errors.ts`
- `apps/api/src/application/push-notifications/push-notifications.module.ts`
- `apps/api/src/application/push-notifications/services/desktop-notification.service.ts`
- `apps/api/src/application/push-notifications/services/notification-preferences.service.ts`
- `apps/api/src/application/push-notifications/services/notification-router.service.ts`
- `apps/api/src/application/push-notifications/services/push-notification-history.service.ts`

---

## Findings

### 1. "Notifications disabled by user"

**File**: `notification-router.service.ts`, line 110
**Text**: `"Notifications disabled by user"`
**Issue**: Passive, slightly formal phrasing. The phrase "by user" reads like system log language rather than a reason string a developer or user might read. Low severity.

### 2. "Quiet hours active"

**File**: `notification-router.service.ts`, line 144
**Text**: `"Quiet hours active"`
**Issue**: Terse but acceptable. Not a trope violation. No finding.

### 3. "Rate limited"

**File**: `notification-router.service.ts`, line 163
**Text**: `rateLimitResult.reason ?? "Rate limited"`
**Issue**: Acceptable fallback. No trope.

### 4. "No active desktop connection"

**File**: `notification-router.service.ts`, line 368
**Text**: `"No active desktop connection"`
**Issue**: Internal suppression reason string, not user-facing. No finding.

### 5. "No provider failures recorded"

**File**: `notification-router.service.ts`, line 521
**Text**: `"No provider failures recorded"`
**Issue**: Internal diagnostic string used in failure summaries stored in the database `errorMessage` column. Not directly user-facing. No trope violation.

---

## Summary

No AI writing tropes found. All user-facing or near-user-facing strings in these files are terse, technical suppression reasons or internal diagnostic messages. None use AI-typical language patterns such as "I", "certainly", "of course", "I'd be happy to", "delve", "dive into", "let me", "sure", hedging filler, or sycophantic openers.
