# Type Safety Audit — apps/api/src/integrations (Batch 27)

**Files inspected**: 8  
**Findings**: 4

## Summary

Batch 27 covers the Microsoft Teams integration and the Microsoft shared OAuth/subscription base services. The Teams fetch service is well-typed using `@microsoft/microsoft-graph-types` `Chat` and `ChatMessage` types. The Teams sync service uses the `BaseSyncService` generic correctly with `ParsedTeamsMessage`. The `microsoft-oauth-base.service.ts` has one notable cast in `buildPlatformMetadata`. The `microsoft-refresh-scope.utils.ts` uses an inline cast to access `accessMode` from `syncSettings: unknown`. The `microsoft-graph-subscription-base.service.ts` is clean.

## Findings

---

### Finding 1

**File**: `apps/api/src/integrations/platforms/microsoft-teams/services/microsoft-teams-oauth.service.ts:29`  
**Category**: `Record<string, ...>` weakening  
**Impact**: Low

**Description**: `MicrosoftTeamsPlatformMetadata` has the same redundant `extends Record<string, unknown>` and `[key: string]: unknown` index signature pattern as OneDrive and Outlook. This is a codebase-wide pattern across all three Microsoft platform OAuth service metadata interfaces.

**Suggestion**: Remove the duplicate index signature. Track as a systemic fix shared with OneDrive and Outlook.

**Evidence**:

```typescript
// microsoft-teams-oauth.service.ts:29-32
interface MicrosoftTeamsPlatformMetadata extends MicrosoftPlatformMetadata<"microsoft-teams">, Record<string, unknown> {
  [key: string]: unknown;
  platform: "microsoft-teams";
}
```

---

### Finding 2

**File**: `apps/api/src/integrations/platforms/microsoft/services/microsoft-oauth-base.service.ts:322`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `buildPlatformMetadata` constructs an object literal and casts it to `TPlatformMetadata` with `as TPlatformMetadata`. The generic `TPlatformMetadata` is constrained to `MicrosoftPlatformMetadata`, so the base fields should be assignable, but any additional fields declared by subclass types are not validated by this cast.

**Suggestion**: Use `satisfies MicrosoftPlatformMetadata` on the object literal construction and let TypeScript verify structural compatibility. If subclasses need additional fields, they should override `buildPlatformMetadata` rather than receiving the base cast.

**Evidence**:

```typescript
// microsoft-oauth-base.service.ts:311-322
return {
  displayName: userInfo.displayName ?? undefined,
  email,
  platform: this.platformId,
  userIdentity: { ... },
} as TPlatformMetadata;  // cast bypasses subclass field validation
```

---

### Finding 3

**File**: `apps/api/src/integrations/platforms/microsoft/utils/microsoft-refresh-scope.utils.ts:33`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `resolveMicrosoftScopeMode` receives `syncSettings: unknown` and accesses `(syncSettings as { accessMode?: unknown }).accessMode`. The cast is made after a `typeof syncSettings === "object" && !Array.isArray(syncSettings)` check, so it is technically safe, but a type guard function would be cleaner.

**Suggestion**: Extract `function hasAccessMode(value: unknown): value is { accessMode?: unknown }` so TypeScript narrows via the type guard without a cast.

**Evidence**:

```typescript
// microsoft-refresh-scope.utils.ts:33-36
if (syncSettings && typeof syncSettings === "object" && !Array.isArray(syncSettings)) {
  const normalizedSyncAccessMode = normalizeMicrosoftAccessMode({
    value: (syncSettings as { accessMode?: unknown }).accessMode,
  });
```

---

### Finding 4

**File**: `apps/api/src/integrations/platforms/microsoft-teams/services/microsoft-teams-fetch.service.ts:47`  
**Category**: Missing strict typing  
**Impact**: Low

**Description**: `microsoftTeamsItemSchema: z.ZodType<unknown> = z.object({}).loose()` is declared but appears to not be used for any actual validation — it is defined as validating to `unknown` and uses `.loose()`. If this schema is being used for cache validation, it provides no protection (any object passes).

**Suggestion**: Either remove the unused schema or replace with a proper schema that validates the minimum `id` field of a Teams chat message.

**Evidence**:

```typescript
// microsoft-teams-fetch.service.ts:47
const microsoftTeamsItemSchema: z.ZodType<unknown> = z.object({}).loose();
```
