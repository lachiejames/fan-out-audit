# Audit: apps-api-src-application-triage

**Files audited:**

- `apps/api/src/application/triage/errors/triage.errors.ts`
- `apps/api/src/application/triage/utils/error-mapper.ts`

## Findings

### 1. Sycophantic congratulation in error message

**File:** `triage.errors.ts`, line 82

```typescript
message: "Congratulations! You're already at triage - no items to process.",
```

**Trope:** Unprompted praise. "Congratulations!" is sycophantic filler that adds no information. This is an error message surfaced when the user tries to do something that has no effect. The appropriate tone is neutral and informative, not celebratory.

**Fix:**

```typescript
message: "No items to process.",
```

---

`error-mapper.ts` contains no user-facing text. No findings.
