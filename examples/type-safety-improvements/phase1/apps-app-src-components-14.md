# Audit: apps-app-src-components-14

**Files inspected**:

- `apps/app/src/components/integrations/connection-wizard/SyncStep.tsx`
- `apps/app/src/components/integrations/connection-wizard/types.ts`
- `apps/app/src/components/integrations/connection-wizard/useConnectionWizard.ts`
- `apps/app/src/components/integrations/core-integration-card.tsx`
- `apps/app/src/components/integrations/directory-view.tsx`
- `apps/app/src/components/integrations/integration-catalog.tsx`
- `apps/app/src/components/integrations/integration-connect-modal.tsx`
- `apps/app/src/components/integrations/integration-connection-row.tsx`

**Findings**: 4

---

## Summary

Four findings across integration components. Two are `Record<string, T>` props where `PlatformId` would be the correct key type. One is an avoidable `as PlatformId` cast where `isPlatformId()` would be cleaner. One is an unnecessary cast on `supportedAccessModes` that circumvents the type system to call `.includes()`.

---

## Findings

### Finding 1: `CORE_INTEGRATION_COPY: Record<string, string>` should use `PlatformId` keys

- **File**: `apps/app/src/components/integrations/core-integration-card.tsx`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `CORE_INTEGRATION_COPY` at line 16 is typed as `Record<string, string>`. The keys are platform IDs but any string is accepted. If a platform ID is misspelled or removed from the object, there is no compile error.
- **Suggestion**: Change to `Partial<Record<PlatformId, string>>` from `@slopweaver/contracts`. Lookup sites become `CORE_INTEGRATION_COPY[platformId]` where `platformId: PlatformId`, with proper `undefined` fallback.
- **Evidence**:
  ```typescript
  // line 16
  CORE_INTEGRATION_COPY: Record<string, string>;
  ```

---

### Finding 2: `platformSyncStates: Record<string, IntegrationHubSyncState>` should use `PlatformId` keys

- **File**: `apps/app/src/components/integrations/directory-view.tsx`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `platformSyncStates` prop at line 36 accepts any string key, but callers only pass platform IDs. Using `Partial<Record<PlatformId, IntegrationHubSyncState>>` would make the shape exhaustive and catch stale platform references.
- **Suggestion**: Change the prop type to `Partial<Record<PlatformId, IntegrationHubSyncState>>` and import `PlatformId` from `@slopweaver/contracts`.
- **Evidence**:
  ```typescript
  // line 36
  platformSyncStates: Record<string, IntegrationHubSyncState>;
  ```

---

### Finding 3: `PLATFORMS[platformId as PlatformId]` cast avoidable with `isPlatformId()` guard

- **File**: `apps/app/src/components/integrations/integration-catalog.tsx`
- **Category**: type-cast
- **Impact**: low
- **Description**: At line 131, `platformId` (which is typed `string`) is cast `as PlatformId` to index into `PLATFORMS`. This cast bypasses the type system — if `platformId` is not a valid key, the result is `undefined` at runtime with no error at compile time.
- **Suggestion**: Import `isPlatformId` from `@slopweaver/contracts` and use a guard: `if (!isPlatformId(platformId)) continue;` before the `PLATFORMS[platformId]` access. This makes the narrowing explicit and eliminates the unsafe cast.
- **Evidence**:
  ```typescript
  // line 131
  PLATFORMS[platformId as PlatformId];
  ```

---

### Finding 4: Unnecessary `as readonly string[]` cast on `supportedAccessModes`

- **File**: `apps/app/src/components/integrations/integration-connection-row.tsx`
- **Category**: type-cast
- **Impact**: medium
- **Description**: At line 67, `(platformConfig?.oauth.supportedAccessModes as readonly string[])?.includes("full")` casts `OAuthAccessMode[]` to `readonly string[]` just to call `.includes("full")`. The cast is there because TypeScript's `Array.includes()` signature only accepts the array's element type. However, the correct fix is to cast the argument, not the array.
- **Suggestion**: Use `(platformConfig?.oauth.supportedAccessModes as string[])?.includes("full")` — or better, import `OAuthAccessMode` from contracts and compare against a typed constant: `platformConfig?.oauth.supportedAccessModes.includes("full" as OAuthAccessMode)`. The cleanest fix is `platformConfig?.oauth.supportedAccessModes.includes("full" satisfies OAuthAccessMode)`.
- **Evidence**:
  ```typescript
  // line 67
  (platformConfig?.oauth.supportedAccessModes as readonly string[])?.includes("full");
  ```
