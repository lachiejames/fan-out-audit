# Type-Safety Audit: apps-api-src-integrations-3

**Batch 64** — `apps/api/src/integrations/core/interfaces/` (continued) and `processors/`

## Summary

8 files reviewed. Key findings: positional params on `fetchFullTranscript` in the transcription provider interface violate the codebase's named-params convention; the `SyncMetrics` interface is duplicated between `sync-progress-tracker.ts` and `sync-helpers.utils.ts`; `TranscriptionProvider<TError = unknown>` has an overly wide default generic.

---

## Findings

### 1

- **File**: `apps/api/src/integrations/core/interfaces/sync-context.interface.ts`
- **Line**: 81
- **Category**: Unsafe `unknown`
- **Impact**: Medium
- **Description**: `resumeState?: unknown` on `ExtendedSyncContext` — typed as `unknown` for all platforms. A generic parameter `TResumeState = unknown` on `ExtendedSyncContext` would let platforms express their specific resume-state shape and catch misuse at compile time.
- **Suggestion**: Add `TResumeState = unknown` generic to `ExtendedSyncContext<TResumeState>` so platforms can type their cursor/resume state without casting.
- **Evidence**: `resumeState?: unknown;`

---

### 2

- **File**: `apps/api/src/integrations/core/interfaces/transcription-provider.interface.ts`
- **Line**: 111
- **Category**: Unsafe `unknown`
- **Impact**: Low–Medium
- **Description**: `TranscriptionProvider<TError = unknown>` — the default for `TError` is `unknown`, which means all error-handling code that uses the default must cast or narrow `TError` at every call site.
- **Suggestion**: Default to `Error` or a project-specific `BaseError` type to provide better type safety by default.
- **Evidence**: `export interface TranscriptionProvider<TError = unknown>`

---

### 3

- **File**: `apps/api/src/integrations/core/interfaces/transcription-provider.interface.ts`
- **Lines**: 136–141
- **Category**: Missing strict typing (named-params violation)
- **Impact**: Medium
- **Description**: `fetchFullTranscript?(integrationId: string, userId: string, meetingId: string)` uses positional params, violating the codebase's mandatory named-object-params convention (`.claude/rules/typescript-patterns.md`).
- **Suggestion**: Change to `fetchFullTranscript?(params: { integrationId: string; userId: string; meetingId: string })`.
- **Evidence**: `fetchFullTranscript?(integrationId: string, userId: string, meetingId: string): Promise<...>`

---

### 4

- **File**: `apps/api/src/integrations/core/processors/sync-progress-tracker.ts`
- **Lines**: 28–34
- **Category**: Duplicate type definitions
- **Impact**: Medium
- **Description**: Local `SyncMetrics` interface in this file is structurally identical to `SyncMetrics` exported from `integrations/core/utils/sync-helpers.utils.ts` (same five numeric fields: `messagesIndexed`, `messagesSkipped`, `messagesSynced`, `threadsIndexed`, `tokensUsed`).
- **Suggestion**: Remove the local definition and import `SyncMetrics` from `sync-helpers.utils.ts`.
- **Evidence**: Both files define `interface SyncMetrics { messagesIndexed: number; messagesSkipped: number; messagesSynced: number; threadsIndexed: number; tokensUsed: number; }`.

---
