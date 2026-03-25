# Audit: apps-api-src-application-prediction

Files reviewed:

- `apps/api/src/application/prediction/errors/prediction.errors.ts`
- `apps/api/src/application/prediction/utils/error-mapper.ts`

---

## Findings

### 1. Redundant phrasing in error message

**File**: `prediction.errors.ts`, line 131

**Text**: `"Insufficient actions for this action. Required: ${requiredActions} actions. Current balance: ${currentBalance} actions."`

**Issue**: "actions for this action" is an awkward repetition. "actions" is used three times in one sentence as both the noun for the feature currency and the noun for the operation. The sentence reads as if it was assembled from template fragments without a final read-through.

**Suggested replacement**: `"Not enough actions to run this prediction. You have ${currentBalance}, but ${requiredActions} are required."`

---

### 2. Vague feature name in trial-locked message

**File**: `prediction.errors.ts`, line 140

**Text**: `"This feature is not available during the trial. Upgrade to unlock it."`

**Issue**: "This feature" is generic and tells the user nothing about what they were trying to do. A user who hits this error mid-flow may not know which feature triggered it. "Upgrade to unlock it" is a mild call-to-action cliche that adds no specifics about what upgrading gives them.

**Suggested replacement**: `"Response predictions are not available on the trial. Upgrade to access them."`

---

`error-mapper.ts` contains no user-facing text (HTTP status codes only).
