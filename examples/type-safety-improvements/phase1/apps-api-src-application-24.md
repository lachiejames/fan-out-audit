# Audit: apps-api-src-application-24

**Files inspected**: 8
**Findings**: 6

## Summary

These files implement the prediction engine (PRD 11): persona CRUD, prediction generation with Claude AI, feedback collection, confidence calibration, and ignore detection. The code is generally well-typed using neverthrow Result patterns and Drizzle-inferred types, but there are several avoidable type casts, an unsafe `Record<string, unknown>` update object, an inlined feedback enum that duplicates the contracts package, and a widened `type: string` field that should be a string-literal union.

## Findings

### Finding 1: `PersonaResponse` duplicates the exported `Persona` type from `@slopweaver/contracts`

- **File**: `apps/api/src/application/personas/services/persona.service.ts:25`
- **Category**: sdk-type-duplication
- **Impact**: medium
- **Description**: The file defines a local `PersonaResponse` interface (lines 25–42) whose shape is identical to `Persona` exported from `@slopweaver/contracts` (itself inferred from `personaResponseSchema`). Every field—`id`, `userId`, `createdAt`, `updatedAt`, `name`, `description`, `icon`, `color`, `avatar`, `isDefault`, `isCustom`, `traits`—is already part of the contract type. If the contract schema changes, this local copy will silently drift.
- **Suggestion**: Delete the local `PersonaResponse` interface and import `Persona` from `@slopweaver/contracts`. Change the return types of `toApiResponse`, `createPersona`, `getPersona`, `listPersonas`, and `updatePersona` to use `Persona`.
- **Evidence**:

```typescript
// persona.service.ts:25 — local copy
interface PersonaResponse {
  id: string;
  userId: string;
  createdAt: Date;
  updatedAt: Date;
  name: string;
  description: string;
  icon: "sparkles" | "zap" | "book" | "briefcase" | "custom";
  color: string;
  avatar: string;
  isDefault: boolean;
  isCustom: boolean;
  traits: {
    formality: number;
    brevity: number;
    technical: number;
  };
}

// packages/contracts/src/contracts/personas/types.ts:17 — already exported
export type Persona = z.infer<typeof personaResponseSchema>;
```

---

### Finding 2: `updateData: Record<string, unknown>` bypasses Drizzle's column-level type checking

- **File**: `apps/api/src/application/personas/services/persona.service.ts:203`
- **Category**: record-weakening
- **Impact**: high
- **Description**: The `updatePersona` method builds a mutable `Record<string, unknown>` object before passing it to Drizzle's `.set()`. This defeats Drizzle's compile-time validation of column names and value types—any misspelled key or wrong value type will silently pass TypeScript and only fail at runtime. Drizzle's `InferInsertModel` / partial update pattern is the correct approach here.
- **Suggestion**: Replace the `Record<string, unknown>` accumulator with a typed partial update object. Declare `const updateData: Partial<typeof personasTable.$inferInsert> & { updatedAt: Date } = { updatedAt: new Date() }` and assign named properties directly. Drizzle accepts a partial insert-typed object in `.set()`.
- **Evidence**:

```typescript
// persona.service.ts:203
const updateData: Record<string, unknown> = { updatedAt: new Date() };

if (data.avatar !== undefined) {
  updateData["avatar"] = data.avatar; // no type check on value
}
// ... 6 more unchecked assignments
await this.database.db.update(personasTable).set(updateData); // Drizzle accepts Record<string,unknown> but loses column typing
```

---

### Finding 3: Unsafe `as CompoundSuggestion | null` type cast without narrowing

- **File**: `apps/api/src/application/prediction/services/confidence-calibration.service.ts:612`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `suggestedAction` arrives as `unknown` (column type from Drizzle JSONB). Rather than using a type guard to verify the object actually conforms to `CompoundSuggestion`, it is blindly cast. If the stored JSON schema changes or the data is malformed, optional chaining silently returns `null` instead of surfacing the data issue, and future non-optional accesses would panic.
- **Suggestion**: Write a type predicate `isCompoundSuggestion(value: unknown): value is CompoundSuggestion` that checks `typeof value === 'object' && value !== null && 'steps' in value && Array.isArray((value as CompoundSuggestion).steps)`. Use that guard instead of the cast.
- **Evidence**:

```typescript
// confidence-calibration.service.ts:611-613
private getSuggestedPrimaryAction({ suggestedAction }: { suggestedAction: unknown }): string | null {
  const suggestion = suggestedAction as CompoundSuggestion | null;  // unsafe cast
  return suggestion?.steps?.[0]?.action ?? null;
}
```

---

### Finding 4: `PredictionFeedbackRecord.explicitFeedback` inlines the enum instead of using `ExplicitFeedback` from contracts

- **File**: `apps/api/src/application/prediction/types/prediction.types.ts:202`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `PredictionFeedbackRecord.explicitFeedback` is typed as a manual union `"helpful" | "not_helpful" | "wrong_tone" | "wrong_content" | "too_long" | null`. This is exactly `ExplicitFeedback | null` (exported from `@slopweaver/contracts`). If a new feedback value is added to the contract enum, this field must be updated manually and will otherwise silently accept values outside the contract.
- **Suggestion**: Replace the inline union with `ExplicitFeedback | null` after importing from `@slopweaver/contracts`.
- **Evidence**:

```typescript
// prediction.types.ts:202
explicitFeedback?: "helpful" | "not_helpful" | "wrong_tone" | "wrong_content" | "too_long" | null;

// packages/contracts/src/contracts/predictions/schemas.ts:37 — source of truth
export const explicitFeedbackSchema = z.enum(["helpful", "not_helpful", "wrong_tone", "wrong_content", "too_long"]);
// packages/contracts/src/contracts/predictions/types.ts:25
export type ExplicitFeedback = z.infer<typeof explicitFeedbackSchema>;
```

---

### Finding 5: `TriageOverrideEvidence.type` is `string` instead of a string-literal union

- **File**: `apps/api/src/application/prediction/types/prediction.types.ts:103`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The JSDoc comment explicitly documents the two valid values `"sender_domain" | "sender_email"` but the field is typed as plain `string`. Any other string value will pass TypeScript without error, and callers cannot use exhaustive switch/pattern matching on this discriminant.
- **Suggestion**: Change the type to `"sender_domain" | "sender_email"`.
- **Evidence**:

```typescript
// prediction.types.ts:95-104
export interface TriageOverrideEvidence {
  learnedAction: string;
  occurrences: number;
  pattern: string;
  /** "sender_domain" | "sender_email" */
  type: string; // should be: "sender_domain" | "sender_email"
}
```

---

### Finding 6: Bracket-indexed access on `Record<string, unknown>` for well-known keys in `prediction.service.ts`

- **File**: `apps/api/src/application/prediction/services/prediction.service.ts:126`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `violation.details` is typed as `Record<string, unknown>` (from `TrialPolicyViolation`). The service then accesses the well-known keys `"code"`, `"currentBalance"`, and `"requiredActions"` via string-indexed lookup with `?.["code"]`. All downstream comparisons use `=== "TRIAL_FEATURE_LOCKED"` etc., so if the key names in `TrialPolicyViolation.details` are renamed, TypeScript will not catch the mismatch. Giving the narrowly-known shapes explicit types in `TrialPolicyViolation` (a discriminated union per `code`) would eliminate all three bracket accesses and the `Number()` coercions.
- **Suggestion**: In `trial-policy.service.ts`, replace `details?: Record<string, unknown>` with a discriminated union per violation code (e.g., `| { code: "TRIAL_FEATURE_LOCKED" } | { code: "INSUFFICIENT_ACTIONS"; currentBalance: number; requiredActions: number } | ...`). Then all accesses in `prediction.service.ts` become type-safe property accesses without bracket notation or `Number()` coercion.
- **Evidence**:

```typescript
// prediction.service.ts:126-133
if (violation.details?.["code"] === "TRIAL_FEATURE_LOCKED") {
  return err(PredictionErrors.paymentRequiredTrialLocked());
}
if (violation.details?.["code"] === "INSUFFICIENT_ACTIONS") {
  return err(
    PredictionErrors.paymentRequiredInsufficientActions({
      currentBalance: Number(violation.details["currentBalance"] ?? 0),
      requiredActions: Number(violation.details["requiredActions"] ?? 0),
    }),
  );
}

// trial-policy.service.ts:28
details?: Record<string, unknown>;  // too wide — well-known keys are undiscoverable
```
