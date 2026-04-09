# Type-Safety Audit: apps-api-src-integrations-7

**Batch 68** — `apps/api/src/integrations/core/services/` (semantic-search.service, sync-run, content-state-updater, sync-cancellation, sync-checkpoint, sync-progress-pubsub, sync-recovery)

## Summary

7 files reviewed. This batch is predominantly clean, well-typed database services and sync lifecycle management. The main finding is in `sync-progress-pubsub.service.ts` where `payload: Record<string, unknown>` is the published event body. `sync-run.service.ts`, `content-state-updater.service.ts`, `sync-cancellation.service.ts`, `sync-checkpoint.service.ts`, and `sync-recovery.service.ts` have no significant type-safety issues. `semantic-search.service.ts` has a minor `attachmentResultsMap` cast.

---

## Findings

### 1

- **File**: `apps/api/src/integrations/core/services/semantic-search.service.ts`
- **Line**: 445
- **Category**: Type casts
- **Impact**: Low
- **Description**: `attachmentResultsMap as Map<string, AttachmentSearchHit>` — the map is typed as `Map<string, AttachmentSearchHit>` at construction (line 399) but TypeScript infers a wider type because `attachmentResults` may be empty. The cast could be eliminated by annotating the variable at construction.
- **Suggestion**: Annotate at line 399: `const attachmentResultsMap = new Map<string, AttachmentSearchHit>(...)` so the cast at line 445 becomes unnecessary.
- **Evidence**: `attachmentResultsMap: attachmentResultsMap as Map<string, AttachmentSearchHit>,`

---

### 2

- **File**: `apps/api/src/integrations/core/services/sync-progress-pubsub.service.ts`
- **Lines**: 36, 167–177
- **Category**: `Record<string, ...>`
- **Impact**: Low–Medium
- **Description**: `payload: Record<string, unknown>` on `SyncProgressPubSubEvent` and `publishByIntegration`. The payload's required fields (`platform`, `userId`) are accessed via runtime checks (`payload["platform"] as string | undefined`) rather than being expressed in the type.
- **Suggestion**: Define a narrower type for the payload that at least captures the required `platform: string` and `userId: string` fields: `{ platform: string; userId: string; [key: string]: unknown }`. This would make the runtime checks compile-time.
- **Evidence**: `payload: Record<string, unknown>;` on the interface; `const platform = payload["platform"] as string | undefined;`

---

### 3

- **File**: `apps/api/src/integrations/core/services/sync-run.service.ts`
- **Line**: 338
- **Category**: Missing strict typing
- **Impact**: Low
- **Description**: `(TERMINAL_PHASES as readonly string[]).includes(status)` — `TERMINAL_PHASES` is likely a typed const tuple; casting to `readonly string[]` loses the element type. `status` is already a union of sync run status values, so `TERMINAL_PHASES.includes(status as string)` or a direct lookup set would preserve types.
- **Suggestion**: Use a `Set<string>` for `TERMINAL_PHASES` membership checks, or type `TERMINAL_PHASES` as a type-level union and use `.includes` without casting.
- **Evidence**: `const isTerminal = (TERMINAL_PHASES as readonly string[]).includes(status);`

---
