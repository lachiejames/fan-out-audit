# Type Safety Audit — apps/api/src/integrations (Batch 23)

**Files inspected**: 8  
**Findings**: 7

## Summary

Batch 23 covers the remaining Linear sync/write/util files and the start of the LinkedIn integration. The Linear sync service repeats the `parsedItems as ResolvedIssue[]` pattern from the Jira sync service (same `BaseSyncService` abstraction). The `.filter(Boolean) as Issue[]` pattern appears again. `linear-oauth.utils.ts` has the same unnecessary `as Date` cast as the Google equivalent. The LinkedIn client provider exposes `Promise<Result<unknown, LinkedinError>>` return types that force all callers to cast the successful value. The LinkedIn OAuth service casts `platformMetadata` without validation.

## Findings

---

### Finding 1

**File**: `apps/api/src/integrations/platforms/linear/services/linear-sync.service.ts:578`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `onContentInserted` and `getEmbeddingTexts` receive `parsedItems: unknown[]` (the `BaseSyncService` abstract method signature) and immediately cast to `ResolvedIssue[]` without validation. This is the same pattern as `jira-sync.service.ts:624/655` — it is caused by the `BaseSyncService` using `unknown[]` as the item type.

**Suggestion**: Thread a generic item type parameter through `BaseSyncService<TItem>` so that concrete subclasses (`LinearSyncService extends BaseSyncService<ResolvedIssue>`) receive typed items without needing a cast.

**Evidence**:

```typescript
// linear-sync.service.ts:578
const resolvedIssues = parsedItems as ResolvedIssue[];

// linear-sync.service.ts (getEmbeddingTexts override)
protected override getEmbeddingTexts(parsedItems: unknown[]): string[] {
  return (parsedItems as ResolvedIssue[]).map((r) => r.content);
}
```

---

### Finding 2

**File**: `apps/api/src/integrations/platforms/linear/services/linear-sync.service.ts:445`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `.filter(Boolean) as Issue[]` narrows out `undefined` values from `issueCache.get()`. Using a type predicate filter is safer and communicates intent more clearly.

**Suggestion**: Replace with `.filter((x): x is Issue => x !== undefined)`.

**Evidence**:

```typescript
// linear-sync.service.ts (same pattern as jira)
ctx.allIssues = selection.selected.map((item) => issueCache.get(item.id)).filter(Boolean) as Issue[];
```

---

### Finding 3

**File**: `apps/api/src/integrations/platforms/linear/utils/linear-oauth.utils.ts:117`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `calculateTokenExpiry` casts `_calculateTokenExpiry({ expiresIn, now })` to `Date` even though `_calculateTokenExpiry` already declares a `Date` return type. This is identical to the issue in `google-oauth.utils.ts:75`.

**Suggestion**: Remove the `as Date` cast. The return type is already `Date`.

**Evidence**:

```typescript
// linear-oauth.utils.ts:117
export function calculateTokenExpiry({
  expiresIn,
  now = Date.now(),
}: {
  expiresIn: number;
  now?: number | undefined;
}): Date {
  return _calculateTokenExpiry({ expiresIn, now }) as Date; // unnecessary cast
}
```

---

### Finding 4

**File**: `apps/api/src/integrations/platforms/linear/services/linear-sync.service.ts:436`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `ctx.resumeState = nextResumeState as LinearCursorState` casts a locally built `Record`-typed object without structural validation. The same issue exists in `jira-sync.service.ts:396`.

**Suggestion**: Use `satisfies LinearCursorState` on the object literal construction site to get compile-time validation.

**Evidence**:

```typescript
// linear-sync.service.ts:436
ctx.resumeState = nextResumeState as LinearCursorState;
```

---

### Finding 5

**File**: `apps/api/src/integrations/platforms/linkedin/providers/linkedin-client.provider.ts`  
**Category**: Missing strict typing  
**Impact**: Medium

**Description**: `LinkedinApiClient.get()`, `.post()`, and `handleResponse()` all return `Promise<Result<unknown, LinkedinError>>`. Because callers receive `unknown` on success, they must either cast or perform runtime checks for every endpoint response. This pushes the type unsafety into every call site.

**Suggestion**: Make the client generic: `get<T>(path: string, schema: z.ZodType<T>): Promise<Result<T, LinkedinError>>` and parse the response with the provided schema. Alternatively, define typed wrappers per endpoint.

**Evidence**:

```typescript
// linkedin-client.provider.ts (approximate)
async get(path: string, options?: RequestInit): Promise<Result<unknown, LinkedinError>>
async post(path: string, body?: unknown, options?: RequestInit): Promise<Result<unknown, LinkedinError>>
private handleResponse(response: Response): Promise<Result<unknown, LinkedinError>>
```

---

### Finding 6

**File**: `apps/api/src/integrations/platforms/linkedin/services/linkedin-oauth.service.ts:326`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `buildMasterUserInfo` casts `integration.platformMetadata` (typed `unknown`) to `LinkedinPlatformMetadata | null` without runtime validation. This is the same pattern found in `jira-oauth.service.ts` and `google-oauth-base.service.ts`.

**Suggestion**: Validate using a Zod schema or type guard before accessing metadata fields.

**Evidence**:

```typescript
// linkedin-oauth.service.ts:326
const metadata = integration.platformMetadata as LinkedinPlatformMetadata | null;
```

---

### Finding 7

**File**: `apps/api/src/integrations/platforms/linkedin/linkedin.plugin.ts:108`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `result.value.platformMetadata` is cast to `Record<string, unknown> | null` and then its `linkedinId` property is accessed via bracket notation. The cast succeeds on any object, so a malformed metadata value would not be caught.

**Suggestion**: Define a `LinkedinPlatformMetadata` type with a `linkedinId?: string` field and validate the metadata with a Zod schema or type guard before accessing it.

**Evidence**:

```typescript
// linkedin.plugin.ts:108
const metadata = result.value.platformMetadata as Record<string, unknown> | null;
const linkedinId = typeof metadata?.["linkedinId"] === "string" ? metadata["linkedinId"] : result.value.platformUserId;
```
