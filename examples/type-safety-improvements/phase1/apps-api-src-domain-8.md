# Audit: apps-api-src-domain-8

**Files inspected**: 6
**Findings**: 3

## Summary

Two actionable type-safety issues were found. The most impactful is a redundant runtime guard combined with a type cast in `action-identifier-requirements.utils.ts` that can be replaced with a direct index-access pattern. The second is that `IPluginRegistryPort.getRegisteredPlatforms()` returns `string[]` instead of `PlatformId[]`, which loses platform-identity information through the entire `validatePlatform` / `validatePlatforms` call chain. The remaining four files (`oauth.types.ts`, `user.errors.ts`, `api-call-error-logging.utils.ts`, `database-error-logging.utils.ts`) are clean.

## Findings

### Finding 1: Redundant runtime guard + type cast for typed object

- **File**: `apps/api/src/domain/utils/action-identifier-requirements.utils.ts:109`
- **Category**: type-cast
- **Impact**: low
- **Description**: `ActionTargetIdentifiers` is a Zod-inferred discriminated union of objects — it is always a non-null object at the call site. The guard `identifiers && typeof identifiers === "object"` can never be false, and the subsequent cast `identifiers as Record<string, unknown>` is needed only because TypeScript cannot index a discriminated union by a dynamic string key. Both the guard and the cast can be eliminated by using `Object.prototype.hasOwnProperty` or by casting once without the redundant guard.
- **Suggestion**: Remove the dead runtime guard. Replace the single-use `record` variable with a direct `(identifiers as Record<string, unknown>)[key]` access, making the intent explicit and keeping the surface area of the cast minimal.
- **Evidence**:

```typescript
// current (line 109-112)
const record = identifiers && typeof identifiers === "object" ? (identifiers as Record<string, unknown>) : {};
return required.filter((key) => {
  const value = record[key];
  return typeof value !== "string" || value.length === 0;
});

// suggested
return required.filter((key) => {
  const value = (identifiers as Record<string, unknown>)[key];
  return typeof value !== "string" || value.length === 0;
});
```

---

### Finding 2: `getRegisteredPlatforms()` returns `string[]` instead of `PlatformId[]`

- **File**: `apps/api/src/domain/ports/services/plugin-registry.port.ts:212`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `IPluginRegistryPort.getRegisteredPlatforms()` is declared to return `string[]`. This loose return type propagates into `validatePlatform` and `validatePlatforms` in `platform-validation.utils.ts`, where `supportedPlatforms` is therefore `string[]`. As a result, `InvalidPlatformError.supportedPlatforms` is also typed as `string[]`. All registered platforms are `PlatformId` values at runtime, so narrowing to `PlatformId[]` would make the validation utilities and the error type more precise without any runtime change.
- **Suggestion**: Change the port method signature to `abstract getRegisteredPlatforms(): PlatformId[]` (importing `PlatformId` from `@slopweaver/contracts`). Update the `InvalidPlatformError` interface in `platform-validation.utils.ts` to reflect `supportedPlatforms: PlatformId[]`. The domain port is allowed to import from `@slopweaver/contracts` (shared layer), so there is no layer-rule violation.
- **Evidence**:

```typescript
// plugin-registry.port.ts (line 212) — current
abstract getRegisteredPlatforms(): string[];

// platform-validation.utils.ts (lines 22-23) — current
export interface InvalidPlatformError {
  readonly supportedPlatforms: string[];  // loses PlatformId precision
  ...
}
```

---

### Finding 3: `IdentifierRequirements` uses `readonly string[]` for key names instead of typed keys

- **File**: `apps/api/src/domain/utils/action-identifier-requirements.utils.ts:3`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The value type of `IdentifierRequirements` is `readonly string[]`, representing required identifier key names (e.g. `"itemId"`, `"threadId"`, `"channelId"`). These key names are a finite set drawn from `ActionTargetIdentifiers`. If a key name is misspelled in `ACTION_IDENTIFIER_REQUIREMENTS` the compiler will not catch it. A union literal type for the valid keys would make the data structure self-validating.
- **Suggestion**: Derive a `ActionTargetIdentifierKey` union from `ActionTargetIdentifiers` (e.g. `keyof ActionTargetIdentifiers`) and use it as the element type: `type IdentifierRequirements = Partial<Record<IntegrationActionType, readonly ActionTargetIdentifierKey[]>>`. Because `ActionTargetIdentifiers` is a discriminated union, `keyof ActionTargetIdentifiers` would only yield the common key `"platform"`. A more practical approach is to define a manual union of the known identifier key names (`"itemId" | "threadId" | "messageId" | "channelId" | "messageTs"`) and use that as the element type. This narrows the set of valid values and catches typos at compile time.
- **Evidence**:

```typescript
// current (line 3)
type IdentifierRequirements = Partial<Record<IntegrationActionType, readonly string[]>>;

// suggested
type IdentifierKey = "itemId" | "threadId" | "messageId" | "channelId" | "messageTs" | "conversationId";
type IdentifierRequirements = Partial<Record<IntegrationActionType, readonly IdentifierKey[]>>;
```
