# Audit: apps-app-src-components-15

**Files inspected**:

- `apps/app/src/components/integrations/integration-row.tsx`
- `apps/app/src/components/integrations/IntegrationHub.tsx`
- `apps/app/src/components/knowledge-edit-modal.tsx`
- `apps/app/src/components/markdown-content.tsx`
- `apps/app/src/components/markdown-content.utils.ts`
- `apps/app/src/components/message-slide-over.tsx`
- `apps/app/src/components/message-slide-over.utils.ts`
- `apps/app/src/components/platform-content/platform-content-renderer.tsx`

**Findings**: 4

---

## Summary

Four findings. `BackendIntegrationRecord` in `IntegrationHub.tsx` is a near-duplicate of `BackendIntegration` in `integration-connect-modal.tsx` — these should be consolidated. `markdown-content.tsx` uses `as Components` where `satisfies` is the correct operator. `message-slide-over.tsx` has two unsafe error casts that should use type guards. `message-slide-over.utils.ts` has an acceptable narrowing cast.

---

## Findings

### Finding 1: Near-duplicate `BackendIntegrationRecord` / `BackendIntegration` interfaces

- **File**: `apps/app/src/components/integrations/IntegrationHub.tsx` (lines 43–49) and `apps/app/src/components/integrations/integration-connect-modal.tsx`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `BackendIntegrationRecord` in `IntegrationHub.tsx` and `BackendIntegration` in `integration-connect-modal.tsx` define the same shape (platform, connectionName, integrationId, etc.). When the shape changes, both must be updated. This violates the single-source-of-truth principle.
- **Suggestion**: Extract a single `IntegrationRecord` type to a shared location (e.g., `apps/app/src/shared/types/integration.ts` or, if it mirrors a contract type, re-export from `@slopweaver/contracts`). Remove the duplicate local definitions.
- **Evidence**:
  ```typescript
  // IntegrationHub.tsx ~lines 43-49
  interface BackendIntegrationRecord {
    platform: string;
    connectionName?: string;
    integrationId: string;
    ...
  }
  // integration-connect-modal.tsx — nearly identical shape
  ```

---

### Finding 2: `as Components` in `markdown-content.tsx` should be `satisfies Components`

- **File**: `apps/app/src/components/markdown-content.tsx`
- **Category**: type-cast
- **Impact**: low
- **Description**: At line 416, a large components object is cast `as Components`. `as` suppresses excess-property checks and type mismatches. `satisfies Components` would validate the object against the type while preserving the inferred literal types for each field.
- **Suggestion**: Replace `} as Components` with `} satisfies Components`. If TypeScript reports errors after the change, they reveal genuine type mismatches that should be fixed.
- **Evidence**:
  ```typescript
  // line 416
  } as Components
  // Should be:
  } satisfies Components
  ```

---

### Finding 3: Unsafe error casts in `message-slide-over.tsx`

- **File**: `apps/app/src/components/message-slide-over.tsx`
- **Category**: type-cast
- **Impact**: medium
- **Description**: Two casts on errors at lines 179 and 210:
  1. `(error as Error & { statusCode?: number }).statusCode` — `error` is `unknown` in a catch block; casting it `as Error & { statusCode?: number }` is unsafe. If `error` is a string or plain object, the access is silently `undefined`.
  2. `error.body as ErrorResponse | undefined` — after checking `error instanceof ApiResponseError`, `error.body` is of type `unknown` and is cast directly. No structural check is performed.
- **Suggestion**:
  1. Use a type guard: `if (error instanceof Error && "statusCode" in error) { ... }` to safely access `statusCode`.
  2. Use `extractErrorMessage(error.body)` from `@slopweaver/contracts` which already handles `unknown` body parsing, or validate the body shape before casting.
- **Evidence**:
  ```typescript
  // line 179
  (error as Error & { statusCode?: number }).statusCode;
  // line 210
  error.body as ErrorResponse | undefined;
  ```

---

### Finding 4: Local `EmailExpandedData`, `CalendarExpandedData`, `ResourceExpandedData` type aliases

- **File**: `apps/app/src/components/platform-content/platform-content-renderer.tsx`
- **Category**: sdk-type-duplication
- **Impact**: low
- **Description**: Lines 64–66 define local type aliases using `Extract<ExpandedData, { platform: ... }>`. These are valid narrowing helpers, not duplicates of contract types, so they are technically appropriate. However, if the contracts package exports equivalent narrowed types, the local aliases become redundant.
- **Suggestion**: Check whether `@slopweaver/contracts` exports named narrowed types like `EmailExpandedData`, `CalendarExpandedData`, or `ResourceExpandedData`. If they exist, replace the local `Extract<>` aliases with the contract exports to reduce local type maintenance.
- **Evidence**:
  ```typescript
  // lines 64-66
  type EmailExpandedData = Extract<ExpandedData, { platform: "google-gmail" | "microsoft-outlook" }>;
  type CalendarExpandedData = Extract<ExpandedData, { platform: CalendarPlatform }>;
  type ResourceExpandedData = Extract<ExpandedData, { platform: ResourceCardPlatform }>;
  ```
