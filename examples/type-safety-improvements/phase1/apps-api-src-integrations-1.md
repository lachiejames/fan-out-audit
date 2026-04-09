# Type-Safety Audit: apps-api-src-integrations-1

**Batch 62** — `apps/api/src/integrations/core/abstractions/`, `actions/`, and `adapters/`

## Summary

10 files reviewed. The adapters layer has several loose `Record<string, unknown>` usages on message sync body, error code maps, and platform metadata. The most actionable finding is three nearly identical error-code `Record` maps in `project-management.adapter.ts` that should be unified.

---

## Findings

### 1

- **File**: `apps/api/src/integrations/core/abstractions/file-storage.types.ts`
- **Line**: 198
- **Category**: `Record<string, ...>`
- **Impact**: Low
- **Description**: `platformMetadata: Record<string, unknown>` on `StorageFile` — metadata shape differs per platform (Google Drive vs OneDrive). The field is display-only but accepting `unknown` values leaves access unguarded.
- **Suggestion**: Change to `Record<string, Json>` (using a `Json` utility type) to at least guarantee JSON-serializability.
- **Evidence**: `platformMetadata: Record<string, unknown>;`

---

### 2

- **File**: `apps/api/src/integrations/core/adapters/content-fetch-demo.utils.ts`
- **Line**: 110
- **Category**: Type casts
- **Impact**: Low
- **Description**: `(metadata["assignee"] as Record<string, unknown>)?.["name"]` — double-cast to access nested metadata. No runtime validation that `assignee` is an object.
- **Suggestion**: Use a Zod schema or an inline `typeof` guard before accessing `name` on `assignee`.
- **Evidence**: `const assignee = (metadata["assignee"] as Record<string, unknown>)?.["name"];`

---

### 3

- **File**: `apps/api/src/integrations/core/adapters/message-sync.adapter.ts`
- **Line**: 66
- **Category**: `Record<string, ...>`
- **Impact**: Medium
- **Description**: `const syncBody: Record<string, unknown> = { forceResync, messageLimit, platform }` — the object's shape is statically known but typed loosely, losing the types of the three known fields downstream.
- **Suggestion**: Use an inline named type `{ forceResync: boolean; messageLimit?: number; platform: string }` instead of `Record<string, unknown>`.
- **Evidence**: `const syncBody: Record<string, unknown> = { forceResync, messageLimit, platform };`

---

### 4

- **File**: `apps/api/src/integrations/core/adapters/project-management.adapter.ts`
- **Lines**: 441, 456, 472
- **Category**: Duplicate type definitions
- **Impact**: Medium
- **Description**: Three nearly identical error-code record types `Record<string, ProjectManagementError["code"]>` inside `mapNotionError`, `mapAsanaError`, and `mapJiraError`. These are structurally identical maps used only for lookup.
- **Suggestion**: Extract a single `PlatformErrorCodeMap = Record<string, ProjectManagementError["code"]>` type alias and reuse it.
- **Evidence**: `const errorCodeMap: Record<string, ProjectManagementError["code"]> = { ... }` repeated three times.

---
