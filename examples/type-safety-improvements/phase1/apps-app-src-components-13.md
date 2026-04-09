# Audit: apps-app-src-components-13

**Files inspected**:

- `apps/app/src/components/integrations/connection-wizard/connection-wizard-resume-bootstrap.ts`
- `apps/app/src/components/integrations/connection-wizard/connection-wizard-steps.tsx`
- `apps/app/src/components/integrations/connection-wizard/ConnectionWizard.tsx`
- `apps/app/src/components/integrations/connection-wizard/ErrorStep.tsx`
- `apps/app/src/components/integrations/connection-wizard/LearnStep.tsx`
- `apps/app/src/components/integrations/connection-wizard/pre-connect-platform-content.ts`
- `apps/app/src/components/integrations/connection-wizard/PreConnectStep.tsx`
- `apps/app/src/components/integrations/connection-wizard/StepProgress.tsx`

**Findings**: 1

---

## Summary

One significant finding: multiple unnecessary `as Record<string, string>` casts in `pre-connect-platform-content.ts`. All other files in this slice are clean — `ErrorStep.tsx` uses `OAuthErrorType` correctly, `ConnectionWizard.tsx` is well-typed orchestration, and the other step components are clean.

---

## Findings

### Finding 1: Unnecessary `as Record<string, string>` casts in `getVisibleReasons()`

- **File**: `apps/app/src/components/integrations/connection-wizard/pre-connect-platform-content.ts`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `getVisibleReasons()` returns `Record<string, string>`. At lines 220, 228, 244, 258, 263, 269, 275, 281, 283, and 286, object literals are cast `as Record<string, string>`. These casts are redundant — TypeScript can infer that `{ key: "value", ... }` satisfies `Record<string, string>` without an explicit cast. Using `as` suppresses any typo or shape errors that TypeScript would otherwise catch on the literal.
- **Suggestion**: Remove the `as Record<string, string>` casts from every object literal return. Alternatively, annotate the function's return type and let TypeScript verify each branch against it. If the return type is already annotated, the casts are entirely superfluous.
- **Evidence**:
  ```typescript
  // Example at line 220 (pattern repeated ~10 times)
  return {
    allPlatformData: "...",
    replyDraftPermission: "...",
  } as Record<string, string>;
  // The cast is redundant — the literal already satisfies the return type.
  ```
