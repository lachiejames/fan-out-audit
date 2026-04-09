# Audit: apps-app-src-components-16

**Files inspected**:

- `apps/app/src/components/platform-content/queue-preview-renderer.tsx`
- `apps/app/src/components/platform-views/asana-task-view.tsx`
- `apps/app/src/components/platform-views/github-view.tsx`
- `apps/app/src/components/platform-views/github-view.utils.ts`
- `apps/app/src/components/platform-views/google-calendar-event-view.tsx`
- `apps/app/src/components/platform-views/google-gmail-thread-view.tsx`
- `apps/app/src/components/platform-views/inline-reply-composer.tsx`
- `apps/app/src/components/platform-views/jira-issue-view.tsx`

**Findings**: 4

---

## Summary

Four findings. A double cast for platform narrowing in `queue-preview-renderer.tsx` would be avoidable with a better type guard return type. Two config maps in GitHub views use `Record<string, T>` where known string literal unions would be safer. A metadata field access in `google-calendar-event-view.tsx` uses an unsafe `as string[]` cast.

---

## Findings

### Finding 1: Double cast `platform as "google-docs"` in `queue-preview-renderer.tsx`

- **File**: `apps/app/src/components/platform-content/queue-preview-renderer.tsx`
- **Category**: type-cast
- **Impact**: medium
- **Description**: At lines 194–195, `platform` is double-cast via `platform as "google-docs"` (or a two-step cast through `string`). This is a symptom of `isResourceCardPlatform` returning `boolean` instead of a type predicate. If the guard returned `platform is ResourceCardPlatform`, no cast would be needed afterward.
- **Suggestion**: Update `isResourceCardPlatform` (or wherever it is defined) to be a proper type predicate: `function isResourceCardPlatform(platform: string): platform is ResourceCardPlatform`. After the guard check, `platform` would already be narrowed to `ResourceCardPlatform` and the cast can be removed.
- **Evidence**:
  ```typescript
  // lines 194-195
  platform as "google-docs";
  ```

---

### Finding 2: `fileStatusConfig: Record<string, {...}>` should use a known status literal union

- **File**: `apps/app/src/components/platform-views/github-view.utils.ts`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `fileStatusConfig` is typed `Record<string, { bg: string; color: string; label: string }>`. The keys are git file statuses (`"added"`, `"modified"`, `"deleted"`, `"renamed"`, `"copied"`, `"unchanged"`). Using `Record<GitFileStatus, {...}>` where `type GitFileStatus = "added" | "modified" | "deleted" | "renamed" | "copied" | "unchanged"` would make the map exhaustive and catch missing cases.
- **Suggestion**: Define `type GitFileStatus = keyof typeof fileStatusConfig` after defining the config as a plain object, or define the union first and annotate accordingly.
- **Evidence**:
  ```typescript
  const fileStatusConfig: Record<string, { bg: string; color: string; label: string }> = { ... }
  ```

---

### Finding 3: `TERMINAL_STATUS_BADGES: Record<string, string>` should use a known status union

- **File**: `apps/app/src/components/platform-views/google-gmail-thread-view.tsx`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `TERMINAL_STATUS_BADGES` at line 57 is `Record<string, string>`. The keys are terminal work-item status strings. Using `Partial<Record<WorkItemStatus, string>>` from `@slopweaver/contracts` would prevent stale or misspelled status keys.
- **Suggestion**: Import `WorkItemStatus` from `@slopweaver/contracts` and type the map as `Partial<Record<WorkItemStatus, string>>`.
- **Evidence**:
  ```typescript
  // line 57
  TERMINAL_STATUS_BADGES: Record<string, string>;
  ```

---

### Finding 4: Unsafe `meta["attendees"] as string[]` in `google-calendar-event-view.tsx`

- **File**: `apps/app/src/components/platform-views/google-calendar-event-view.tsx`
- **Category**: type-cast
- **Impact**: medium
- **Description**: At line 54, `meta["attendees"] as string[]` casts an `unknown` value to `string[]` without any runtime check. If `attendees` is not an array of strings (e.g., it is an array of objects), the component will render incorrectly or crash at runtime.
- **Suggestion**: Use a runtime guard before the cast: `Array.isArray(meta["attendees"]) && meta["attendees"].every((a) => typeof a === "string")`. Or use a narrowing utility: `const attendees = Array.isArray(meta["attendees"]) ? (meta["attendees"] as unknown[]).filter((a): a is string => typeof a === "string") : []`.
- **Evidence**:
  ```typescript
  // line 54
  meta["attendees"] as string[];
  ```
