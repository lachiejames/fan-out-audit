# Audit: apps-app-src-sentry.ts

**Files inspected**: 1
**Findings**: 2

## Summary

`sentry.ts` is a small, focused initialization file. It contains one notable type-safety issue with a double-cast pattern inside the `PerformanceObserver` callback, and one minor unsafe string cast on the `SLOPWEAVER_ENV` env var. The rest of the file is clean.

---

## Findings

### Finding 1: Double cast on PerformanceObserver entries

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/sentry.ts:44-48`
- **Category**: type-cast
- **Impact**: low
- **Description**: `list.getEntries()` is first cast to `PerformanceEventTiming[]` and then each entry is immediately re-cast to a freehand `{ name?: string; duration?: number; interactionId?: number }`. The second cast is redundant — `PerformanceEventTiming` already has `name`, `duration`, and `interactionId` as standard DOM properties. The double cast also provides weaker IDE support than accessing the known `PerformanceEventTiming` members directly.
- **Suggestion**: Remove the second `anyEntry` cast. Access `entry.duration`, `entry.name`, and `entry.interactionId` directly on the `PerformanceEventTiming` typed `entry`. Ensure `tsconfig.json` targets `ES2019+` or includes `"dom.iterable"` so that `PerformanceEventTiming.interactionId` is available.
- **Evidence**:
  ```ts
  for (const entry of list.getEntries() as PerformanceEventTiming[]) {
    const anyEntry = entry as { name?: string; duration?: number; interactionId?: number };
  ```

### Finding 2: Unsafe `as string | undefined` cast for env var

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/sentry.ts:22`
- **Category**: type-cast
- **Impact**: low
- **Description**: `import.meta.env["SLOPWEAVER_ENV"] as string | undefined` widens to `string` without narrowing to the known valid values `"staging" | "prod"`. The subsequent function `getDeployEnv()` does perform the narrowing check, so the practical risk is contained — but the cast allows any arbitrary string to reach the function before being filtered.
- **Suggestion**: The narrowing in `getDeployEnv()` is already correct. The minor improvement is to type the constant as `string | undefined` without `as` (it is already `string | undefined` from `import.meta.env`) or to do the narrowing inline. This is low-priority given the downstream guard.
- **Evidence**: `const env = import.meta.env["SLOPWEAVER_ENV"] as string | undefined;`
