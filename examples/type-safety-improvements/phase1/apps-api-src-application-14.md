# Audit: apps-api-src-application-14

**Files inspected**: 8
**Findings**: 6

## Summary

These files implement the core-profile subsystem: building, serialising, and updating a user's AI context profile from todos, entities, writing patterns, and messages. The main type-safety issues are a type cast caused by a `filter(Boolean)` pattern that TypeScript cannot narrow automatically, an unsafe `as typeof ctx` cast inside `serializeCurrentContextForPrompt`, inline re-declarations of shapes already available from `ProfileContentFields`, a loose `Set<string | undefined>` that admits `undefined` into a set meant to hold project names, an untyped `eventType` field on `PendingUpdate` that could be a string literal union, and a `Record<string, string>` map in `demo-profile-loader.service.ts` that mixes two semantically different keys.

## Findings

### Finding 1: Unsafe `as string[]` cast after `filter(Boolean)`

- **File**: `apps/api/src/application/core-profile/utils/core-profile-builders.utils.ts:72`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `filter(Boolean)` on `(string | null)[]` returns `(string | null)[]` in TypeScript's type system; the result is then cast `as string[]` to paper over the gap. If a future refactor changes the upstream type the cast silently hides the mistake.
- **Suggestion**: Replace the cast with a proper type-narrowing predicate: `.filter((c): c is string => c !== null)`. This is both safe and self-documenting.
- **Evidence**:

```typescript
const recentTopics = [
  ...new Set(
    todos
      .slice(0, 5)
      .map((t) => t.category)
      .filter(Boolean),
  ),
] as string[];
```

---

### Finding 2: Unsafe `as typeof ctx` cast when narrowing `unknown` object

- **File**: `apps/api/src/application/core-profile/utils/core-profile-tokens.utils.ts:93`
- **Category**: type-cast
- **Impact**: medium
- **Description**: After confirming `typeof currentContext === "object"` the code casts to the local `ctx` shape with `currentContext as typeof ctx`. This bypasses any actual structural check — passing `{ recentTopics: 42 }` would satisfy the `typeof` guard but break the `.join()` calls below. The local inline type `{ recentTopics?: string[]; pendingItems?: string[]; upcomingEvents?: string[] }` is also a partial re-definition of `ProfileCurrentContext` from `@slopweaver/contracts`.
- **Suggestion**: (1) Import `ProfileCurrentContext` from `@slopweaver/contracts` and use it as the local type instead of the inline shape. (2) Add a lightweight structural guard (or use Zod's `profileCurrentContextSchema.safeParse`) before the cast, or at minimum assert with `satisfies` after the cast to catch drift.
- **Evidence**:

```typescript
let ctx: { recentTopics?: string[]; pendingItems?: string[]; upcomingEvents?: string[] };
if (typeof currentContext === "string") {
  try {
    ctx = JSON.parse(currentContext);
  } catch {
    return currentContext || null;
  }
} else if (typeof currentContext === "object") {
  ctx = currentContext as typeof ctx; // ← unsafe cast
} else {
  return null;
}
```

---

### Finding 3: Inline re-typed element shapes in `serializeProfileForPrompt` duplicate `ProfileContentFields`

- **File**: `apps/api/src/application/core-profile/utils/core-profile-tokens.utils.ts:141-154`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: The `.map()` callbacks in `serializeProfileForPrompt` annotate their parameters with explicit inline object types (`(p: string) => ...`, `(p: { name: string; relationship: string; notes?: string }) => ...`, `(p: { name: string; status: string; deadlines: string[] }) => ...`). These shapes duplicate the element types already inferred from `ProfileContentFields` (which is a `Pick` of `CoreProfileResponse`). If the contracts schema changes, the inline types silently drift.
- **Suggestion**: Remove the explicit parameter annotations and let TypeScript infer the element type from `profile.keyPeople` / `profile.activeProjects` / `profile.communication.writingPatterns`. The types are already known through `ProfileContentFields`.
- **Evidence**:

```typescript
profile.keyPeople.map(
  (p: { name: string; relationship: string; notes?: string }) =>
    `- ${p.name} (${p.relationship})${p.notes ? `: ${p.notes}` : ""}`,
);
// ...
profile.activeProjects.map(
  (p: { name: string; status: string; deadlines: string[] }) =>
    `- ${p.name}: ${p.status}${p.deadlines.length > 0 ? ` (deadlines: ${p.deadlines.join(", ")})` : ""}`,
);
// ...
profile.communication.writingPatterns.map((p: string) => `- ${p}`);
```

---

### Finding 4: `Set<string | undefined>` admits `undefined` as a valid project name

- **File**: `apps/api/src/application/core-profile/utils/core-profile-relationships.utils.ts:27`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `activeProjectNames` parameter is typed `Set<string | undefined>`. This means callers can accidentally include `undefined` values, and `activeProjectNames.has(tag.toLowerCase())` will never match `undefined` anyway — the union member is dead and misleading. The set is used purely for string membership testing.
- **Suggestion**: Tighten to `Set<string>`. Callers should filter out `undefined` before constructing the set (e.g. `new Set(projects.map(p => p.name).filter((n): n is string => n !== undefined))`).
- **Evidence**:

```typescript
export function deriveRelationship({
  entity,
  activeProjectNames,
}: {
  entity: EntityInput;
  activeProjectNames: Set<string | undefined>;  // ← should be Set<string>
}): string {
```

---

### Finding 5: `eventType` on `PendingUpdate` typed as bare `string` instead of a literal union

- **File**: `apps/api/src/application/core-profile/services/profile-update.service.ts:19-23`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `PendingUpdate.eventType` field is `string`, but every assignment in the file uses one of exactly three string literals: `"writing-pattern:created"`, `"entity:resolved"`, `"user:settings-updated"`. Using a bare `string` loses exhaustiveness checks and IDE completion, and allows invalid event types to be stored silently.
- **Suggestion**: Extract a union type and use it on the interface:

  ```typescript
  type ProfileUpdateEventType = "writing-pattern:created" | "entity:resolved" | "user:settings-updated";

  interface PendingUpdate {
    userId: string;
    eventType: ProfileUpdateEventType;
    timestamp: Date;
  }
  ```

- **Evidence**:

```typescript
interface PendingUpdate {
  userId: string;
  eventType: string;   // ← should be a literal union
  timestamp: Date;
}
// All three call sites:
eventType: "writing-pattern:created",
eventType: "entity:resolved",
eventType: "user:settings-updated",
```

---

### Finding 6: `Record<string, string>` mixes two semantically distinct keys

- **File**: `apps/api/src/application/demo-seed/demo-profile-loader.service.ts:89`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `integrationIdMap` is populated with both `fixture.id` (a UUID) and `fixture.platform` (a platform slug) as keys, both mapping to a new UUID. Typing this as `Record<string, string>` hides the dual-key strategy and makes it impossible for TypeScript to distinguish look-ups by ID from look-ups by platform name. A downstream consumer that passes a misspelled platform string will get `undefined` with no compile-time warning.
- **Suggestion**: Model the two index strategies explicitly, either as two separate maps (`byId: Record<string, string>` and `byPlatform: Partial<Record<PlatformId, string>>`) or as a tagged union lookup helper. At a minimum, import `PlatformId` from `@slopweaver/contracts` to type the platform key axis: `Record<string | PlatformId, string>` makes the intent visible. Splitting into two maps is the strongest option.
- **Evidence**:

```typescript
const integrationIdMap: Record<string, string> = {};
// ...
integrationIdMap[fixture.id] = newId; // keyed by UUID
integrationIdMap[fixture.platform] = newId; // keyed by platform slug
```
