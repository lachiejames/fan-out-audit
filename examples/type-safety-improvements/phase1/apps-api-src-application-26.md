# Audit: apps-api-src-application-26

**Files inspected**: 8
**Findings**: 4

## Summary

These files implement priority scoring utilities, discriminated-union error types, and the morning-brief context assembler (prompt builder + DB service + utility). The code is generally well-typed but has two instances of unnecessary `as` casts, a duplicate `DatabaseError` interface definition across two sibling error files, and a `Record<string, number>` return type in the utils function where a mapped type over known platform keys would be stricter.

## Findings

### Finding 1: Duplicate `DatabaseError` interface across sibling error files

- **File**: `apps/api/src/application/proactive/errors/morning-brief.errors.ts:38` and `apps/api/src/application/proactive/errors/proactive.errors.ts:36`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: Both files independently define an identical `DatabaseError` interface with `code: "DATABASE_ERROR"`, `message: string`, and `cause?: unknown`. The two error unions (`MorningBriefError` and `ProactiveError`) each include their own copy. Any code that handles both unions in the same switch must import from the correct file to avoid confusion, and future changes risk drifting the two shapes apart.
- **Suggestion**: Define `DatabaseError` once — either in a shared `proactive-shared.errors.ts` file or in the already-existing `proactive.errors.ts` — and re-export it from `morning-brief.errors.ts`. Both union types can then reference the same interface.
- **Evidence**:

```typescript
// morning-brief.errors.ts:38-43
export interface DatabaseError {
  readonly code: "DATABASE_ERROR";
  readonly message: string;
  readonly cause?: unknown;
}

// proactive.errors.ts:36-41  (identical shape)
export interface DatabaseError {
  readonly code: "DATABASE_ERROR";
  readonly message: string;
  readonly cause?: unknown;
}
```

---

### Finding 2: Unnecessary `as ResolvedEntity` casts in `enrichMessagesWithRelationships`

- **File**: `apps/api/src/application/proactive/services/morning-brief-context.service.ts:367` and `:370`
- **Category**: type-cast
- **Impact**: medium
- **Description**: The Drizzle `.select({ displayName, identities, isVip, relationship })` query returns a partial object — only the four explicitly selected columns — whose inferred type is narrower than the full `ResolvedEntity` (`typeof resolvedEntitiesTable.$inferSelect`). Casting this partial to `ResolvedEntity` silently suppresses a real type mismatch. If code later reads any unselected field from the map (e.g., `id`, `userId`, `avatarUrl`) it will receive `undefined` at runtime with no TypeScript error.
- **Suggestion**: Define an inline type or a named type alias for the partial projection (e.g., `EntityLookup`) and type the maps accordingly: `Map<string, EntityLookup>`. Only fields actually used downstream (`isVip`, `relationship`) are needed in the map value, so the cast is entirely avoidable.
- **Evidence**:

```typescript
// Lines 350-362
const entities = await this.database.db
  .select({
    displayName: resolvedEntitiesTable.displayName,
    identities: resolvedEntitiesTable.identities,
    isVip: resolvedEntitiesTable.isVip,
    relationship: resolvedEntitiesTable.relationship,
  })
  .from(resolvedEntitiesTable)
  .where(eq(resolvedEntitiesTable.userId, userId));

const entityByEmail = new Map<string, ResolvedEntity>(); // ← typed as full entity
const entityBySenderId = new Map<string, ResolvedEntity>(); // ← typed as full entity

for (const entity of entities) {
  for (const identity of entity.identities ?? []) {
    if (identity.email) {
      entityByEmail.set(identity.email.toLowerCase(), entity as ResolvedEntity); // ← cast hides mismatch
    }
    if (identity.platformId) {
      entityBySenderId.set(identity.platformId, entity as ResolvedEntity); // ← cast hides mismatch
    }
  }
}
```

---

### Finding 3: `priorityFactors` shape duplicated between DB table and `OvernightMessage`

- **File**: `apps/api/src/application/proactive/services/morning-brief-context.service.ts:34-41`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `OvernightMessage.priorityFactors` is typed as an inline object with five optional numeric fields. The Drizzle table column `contentsTable.priorityFactors` in `contents.table.ts:75-81` defines the same five fields as required (non-optional) numbers. The two shapes are subtly different (optional vs required) and defined independently. If a field is ever added to the DB column type, `OvernightMessage` will silently lag behind, causing downstream code (e.g., the prompt builder's factor threshold checks at `morning-brief.prompt.ts:184-193`) to miss the new field.
- **Suggestion**: Derive the `OvernightMessage.priorityFactors` type from the Drizzle column rather than redefining it. The table already exports the inferred type via `$inferSelect`; extract the `priorityFactors` field type with a utility type:

```typescript
import type { contentsTable } from "@/shared/db/tables/contents.table";
type PriorityFactors = NonNullable<(typeof contentsTable.$inferSelect)["priorityFactors"]>;
```

Then use `PriorityFactors | null` in `OvernightMessage` instead of the inline object.

- **Evidence**:

```typescript
// morning-brief-context.service.ts:34-41 — inline, all fields optional
priorityFactors: {
  senderImportance?: number;
  messageType?: number;
  urgencySignals?: number;
  waitingOnYou?: number;
  recencyDecay?: number;
} | null;

// contents.table.ts:75-81 — Drizzle column type, all fields required
priorityFactors: jsonb("priority_factors").$type<{
  senderImportance: number; // 0-30
  messageType: number;      // 0-20
  urgencySignals: number;   // 0-20
  waitingOnYou: number;     // 0-15
  recencyDecay: number;     // 0-15
}>(),
```

---

### Finding 4: `Record<string, number>` return type for `calculateUnreadByPlatform` and `MorningBriefContext.unreadByPlatform`

- **File**: `apps/api/src/application/proactive/services/morning-brief-context.utils.ts:16` and `morning-brief-context.service.ts:87`
- **Category**: record-weakening
- **Impact**: low
- **Description**: Both the utility function's return type and the `MorningBriefContext.unreadByPlatform` field use `Record<string, number>`, accepting any arbitrary string as a key. The set of valid platform source values is constrained by `PlatformId` from `@slopweaver/contracts`. Using `Partial<Record<PlatformId, number>>` (or `Record<PlatformId, number>`) would close the type to unknown platform strings, enable exhaustive access checks, and make the prompt builder's `Object.entries(context.unreadByPlatform)` iteration type-safe.
- **Suggestion**: Change the return type of `calculateUnreadByPlatform` and `MorningBriefContext.unreadByPlatform` to `Partial<Record<PlatformId, number>>`, importing `PlatformId` from `@slopweaver/contracts`. The internal accumulator can remain `Record<string, number>` and be cast on return, or be typed narrowly from the start if `msg.source` is typed as `PlatformId`.
- **Evidence**:

```typescript
// morning-brief-context.utils.ts:16
export function calculateUnreadByPlatform({ messages }: { messages: OvernightMessage[] }): Record<string, number> {

// morning-brief-context.service.ts:87
/** Platform breakdown */
unreadByPlatform: Record<string, number>;
```
