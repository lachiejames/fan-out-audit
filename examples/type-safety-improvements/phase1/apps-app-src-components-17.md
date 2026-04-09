# Audit: apps-app-src-components-17

**Files inspected**:

- `apps/app/src/components/platform-views/linear-issue-view.tsx`
- `apps/app/src/components/platform-views/linear-priority-icon.tsx`
- `apps/app/src/components/platform-views/linkedin-post-view.tsx`
- `apps/app/src/components/platform-views/monday-item-view.tsx`
- `apps/app/src/components/platform-views/notion-page-view.tsx`
- `apps/app/src/components/platform-views/resource-card-view.tsx`
- `apps/app/src/components/platform-views/slack-thread-view.tsx`
- `apps/app/src/components/platform-views/teams-thread-view.tsx`

**Findings**: 2

---

## Summary

Two findings in this slice. `linear-issue-view.tsx` defines local `statusConfig` and `priorityConfig` objects whose key types (`"backlog" | "todo" | ...` and `"urgent" | "high" | ...`) duplicate the `LinearIssueViewProps` union types rather than being derived from them. `notion-page-view.tsx` has a `property as Record<string, unknown>` cast that could be replaced with a proper type guard. All other files (linkedin, monday, resource-card, slack, teams) are clean — they correctly import from `@slopweaver/contracts` and use contract types directly.

---

## Findings

### Finding 1: `statusConfig` and `priorityConfig` keys duplicate prop union types in `linear-issue-view.tsx`

- **File**: `apps/app/src/components/platform-views/linear-issue-view.tsx`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `LinearIssueViewProps` declares `status: "backlog" | "todo" | "in_progress" | "in_review" | "done" | "cancelled"` and `priority: "urgent" | "high" | "medium" | "low" | "none"`. The `statusConfig` object (line 68) and `priorityConfig` object (line 77) define the same keys inline as object properties. If a new status or priority is added to the prop union, the config objects must be updated manually.
- **Suggestion**: Use `Record<LinearIssueViewProps["status"], ...>` and `Record<LinearIssueViewProps["priority"], ...>` as the types for `statusConfig` and `priorityConfig`. TypeScript will then enforce that every union member has a config entry and will flag missing cases as errors.
- **Evidence**:
  ```typescript
  // line 68 — keys repeat the "status" prop union
  const statusConfig = {
    backlog: { ... },
    cancelled: { ... },
    done: { ... },
    in_progress: { ... },
    in_review: { ... },
    todo: { ... },
  };
  // line 77 — keys repeat the "priority" prop union
  const priorityConfig = {
    high: { ... },
    low: { ... },
    medium: { ... },
    none: { ... },
    urgent: { ... },
  };
  ```

---

### Finding 2: `property as Record<string, unknown>` cast in `notion-page-view.tsx`

- **File**: `apps/app/src/components/platform-views/notion-page-view.tsx`
- **Category**: type-cast
- **Impact**: low
- **Description**: At line 40, `(property as Record<string, unknown>)["title"]` casts a `NotionPropertyValue` to `Record<string, unknown>` to access the `"title"` field. If `NotionPropertyValue` already has a discriminated union shape, the cast bypasses the discriminant check.
- **Suggestion**: Check whether `NotionPropertyValue` has a `type: "title"` discriminant already guarded at line 37 (`if (property.type !== "title") continue`). If so, access the `.title` field directly after the `type` check, without casting to `Record<string, unknown>`. If the title field is not on the narrowed type, extend the contract type rather than casting.
- **Evidence**:
  ```typescript
  // line 40
  const title = flattenRichText({ richText: (property as Record<string, unknown>)["title"] });
  // Note: property.type === "title" is already confirmed at line 37
  ```
