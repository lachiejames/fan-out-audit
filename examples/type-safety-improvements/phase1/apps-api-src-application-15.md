# Audit: apps-api-src-application-15

**Files inspected**: 8
**Findings**: 5

## Summary

These files implement the digest feature (aggregating daily stats and insights), entity resolution/discovery (finding and linking contacts across platforms), and a domain event handler for content actions. The dominant issue is duplicate type definitions: `DigestUrgentItem`, `DigestStats`, `DigestInsights`, and `DigestResponse` are defined locally in both `digest.service.ts` and `digest-insights.utils.ts` despite identical types being exported from `@slopweaver/contracts`. A secondary issue is an unsafe type cast in the event handler and an inline interface defined inside a private method body.

## Findings

### Finding 1: Duplicate digest types — defined locally, already exported from contracts

- **File**: `apps/api/src/application/digest/services/digest.service.ts:17-44`
- **Category**: duplicate-type
- **Impact**: high
- **Description**: `DigestUrgentItem`, `DigestStats`, `DigestInsights`, and `DigestResponse` are all defined as local interfaces in `digest.service.ts`. The exact same four types exist in `@slopweaver/contracts` (see `packages/contracts/src/contracts/digest/types.ts` and exported from `packages/contracts/src/index.ts`). If the contract shapes evolve, the local copies silently drift and the service's return type no longer matches what the controller/client expects.
- **Suggestion**: Remove all four local interface declarations and import them from `@slopweaver/contracts`:
  ```typescript
  import type { DigestUrgentItem, DigestStats, DigestInsights, DigestResponse } from "@slopweaver/contracts";
  ```
- **Evidence**:

```typescript
// digest.service.ts lines 17-44
interface DigestUrgentItem {
  createdAt: Date;
  id: string;
  platform: string;
  priority: "urgent" | "high" | "medium" | "low";
  subtitle: string;
  title: string;
  type: "message" | "todo";
}

interface DigestStats {
  messagesHandledToday: number;
  todosCompletedToday: number;
  todosRemaining: number;
  unreadMessages: number;
}

interface DigestInsights {
  productivityScore: number;
  suggestedAction: string;
  topPriority: string;
}

interface DigestResponse {
  insights: DigestInsights;
  stats: DigestStats;
  urgentItems: DigestUrgentItem[];
}
```

---

### Finding 2: Same four digest types duplicated again in the utils file

- **File**: `apps/api/src/application/digest/utils/digest-insights.utils.ts:8-29`
- **Category**: duplicate-type
- **Impact**: high
- **Description**: `digest-insights.utils.ts` re-declares its own copies of `DigestUrgentItem`, `DigestStats`, and `DigestInsights` — the same shapes already in `@slopweaver/contracts` and already duplicated in `digest.service.ts`. Three definitions of the same types exist across two files and the contracts package. A shape change in contracts requires updating three places, and the compiler won't catch mismatches between the local copies.
- **Suggestion**: Import the three types from `@slopweaver/contracts` and delete the local declarations. Because `generateInsights` is already exported, callers will automatically get the contracts-aligned types through inference.
- **Evidence**:

```typescript
// digest-insights.utils.ts lines 8-29
interface DigestUrgentItem {
  createdAt: Date;
  id: string;
  platform: string;
  priority: "urgent" | "high" | "medium" | "low";
  subtitle: string;
  title: string;
  type: "message" | "todo";
}

interface DigestStats {
  messagesHandledToday: number;
  todosCompletedToday: number;
  todosRemaining: number;
  unreadMessages: number;
}

interface DigestInsights {
  productivityScore: number;
  suggestedAction: string;
  topPriority: string;
}
```

---

### Finding 3: Unsafe `as` cast on event payload — bypasses runtime narrowing

- **File**: `apps/api/src/application/events/handlers/content-action-taken.handler.ts:41`
- **Category**: type-cast
- **Impact**: high
- **Description**: `job.payload` is cast directly to `ContentActionTakenPayload` with `as`, then immediately destructured. `DomainEventJob.payload` is likely typed as `unknown` or a wide union; casting without any narrowing means a malformed payload (wrong event type queued, replay of an old schema, etc.) will silently pass TypeScript and explode at runtime when accessing fields like `contentId` or `action`. This is exactly the class of bug that event consumers should guard against with a Zod parse or type guard.
- **Suggestion**: Parse `job.payload` through the corresponding Zod schema from contracts (e.g., `contentActionTakenPayloadSchema.parse(job.payload)`) rather than casting. If a schema doesn't exist yet, add one or use a type guard. This also makes handler tests more meaningful since you can pass arbitrary payloads.
- **Evidence**:

```typescript
// content-action-taken.handler.ts line 41
const payload = job.payload as ContentActionTakenPayload;
const { userId, contentId, action, source, workItemId } = payload;
```

---

### Finding 4: Inline interface declared inside a private method body

- **File**: `apps/api/src/application/entities/jobs/entity-resolution.job.ts:108-115`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `RecentMessageSender` is declared as an `interface` inside the body of an async arrow function inside `Promise.allSettled(…).map(…)`. Interfaces in function bodies are legal TypeScript but are unusual, uncollectable by the type system from outside (they have no module-level name), and can't be reused. More critically, the interface is used to annotate the result of a Drizzle `.select()` call — but Drizzle already infers the exact column types from the table schema. The explicit annotation can hide type mismatches (e.g., `senderAvatar` actually being `string | null` vs the interface's `string | null`) because the cast `as` is implicit via annotation.
- **Suggestion**: Remove the inline interface. Either rely on Drizzle's inferred type for the select result directly, or hoist `RecentMessageSender` to module level so it can be inspected and reused. If an explicit annotation is desired, use `typeof contentsTable.$inferSelect` narrowed to the selected columns via a `Pick<>`.
- **Evidence**:

```typescript
// entity-resolution.job.ts lines 108-116
interface RecentMessageSender {
  senderId: string | null;
  senderName: string | null;
  senderEmail: string | null;
  senderAvatar: string | null;
  source: string;
}

const recentMessages: RecentMessageSender[] = await this.database.db
  .select({ ... })
  .from(contentsTable)
  ...
```

---

### Finding 5: Signal type literal union re-typed inline as `string` in two places

- **File**: `apps/api/src/application/entities/jobs/entity-resolution.job.ts:166`, `262`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `signals` is typed inline as `{ type: string; weight: number; evidence: string }[]`. Later, when constructing `newLink.signals`, each element's `type` is cast with `s.type as "email_match" | "name_similarity" | "interaction_pattern" | "mentioned_together"`. The cast is necessary precisely because the original array was typed with `type: string` instead of the narrower union. This creates a round-trip: a wide `string` is assigned, then asserted back to a narrow union, defeating type safety — invalid signal types can be pushed into the array without complaint and only the `as` at line 262 masks the problem.
- **Suggestion**: Define a module-level type alias for the signal type union and use it throughout:
  ```typescript
  type SignalType = "email_match" | "name_similarity" | "interaction_pattern" | "mentioned_together";
  ```
  Then type `signals` as `{ type: SignalType; weight: number; evidence: string }[]`, remove the `as` cast at line 262, and let the compiler enforce valid values at push time.
- **Evidence**:

```typescript
// Line 166 — wide string used for type field
const signals: { type: string; weight: number; evidence: string }[] = [];

// Line 261-264 — as-cast back to narrow union, only possible because the original was too wide
signals: bestMatch.signals.map((s) => ({
  evidence: s.evidence,
  type: s.type as "email_match" | "name_similarity" | "interaction_pattern" | "mentioned_together",
  weight: s.weight,
})),
```
