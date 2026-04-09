# Type-Safety Audit: apps-api-src-integrations-13

**Batch 74** — `apps/api/src/integrations/platforms/facebook-messenger/utils/` (oauth, transform, webhook), `github/` (errors, plugin, client provider, actions, fetch services)

**Files inspected**: 8
**Findings**: 5

## Summary

GitHub fetch service has state casts (`pr.state as "open" | "closed"` and `issue.state as "open" | "closed"`) because the Octokit SDK returns `string` not a union. GitHub plugin uses bracket string-index access on `Record<string, string>` identifiers. The `stateMap` in github-fetch uses `Record<string, GithubReview["state"]>` where a `const` map would be stricter. Facebook messenger transform and oauth utils are clean. GitHub client provider correctly aliases `Octokit` directly.

## Findings

### Finding 1: pr.state as "open" | "closed" cast in github-fetch

- **File**: `src/integrations/platforms/github/services/github-fetch.service.ts:619`
- **Category**: type-cast
- **Impact**: low
- **Description**: `pr.state as "open" | "closed"` — the Octokit SDK types `state` as `string` even though the API guarantees it is one of a few values. The cast is safe but reflects the SDK's weak typing.
- **Suggestion**: Use a runtime check (`if (pr.state === "open" || pr.state === "closed")`) rather than a cast, or assert the value using a helper that throws if it's unexpected.
- **Evidence**:

```typescript
let state: "open" | "closed" | "merged" = pr.state as "open" | "closed";
```

---

### Finding 2: issue.state as "open" | "closed" cast

- **File**: `src/integrations/platforms/github/services/github-fetch.service.ts:686`
- **Category**: type-cast
- **Impact**: low
- **Description**: Same pattern as Finding 1 for issue state.
- **Suggestion**: Same as Finding 1 — use a runtime guard instead of a cast.
- **Evidence**:

```typescript
state: issue.state as "open" | "closed",
```

---

### Finding 3: stateMap Record<string, GithubReview["state"]>

- **File**: `src/integrations/platforms/github/services/github-fetch.service.ts:741`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `const stateMap: Record<string, GithubReview["state"]> = { ... }` — using `string` as the key type loses the constraint that keys should be specific review states. Any typo in a key lookup would silently return `undefined`.
- **Suggestion**: Use `Partial<Record<GithubReviewState, GithubReview["state"]>>` where `GithubReviewState` is an exhaustive union, or use `as const` on the object and derive the key type.
- **Evidence**:

```typescript
const stateMap: Record<string, GithubReview["state"]> = {
  APPROVED: "approved",
  CHANGES_REQUESTED: "changes_requested",
  ...
};
```

---

### Finding 4: Bracket-access on Record<string, string> identifiers in github.plugin.ts

- **File**: `src/integrations/platforms/github/github.plugin.ts:75`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `params.parsed.identifiers["owner"]`, `["repo"]`, `["number"]` — accessing `identifiers` (typed `Record<string, string>`) with string keys bypasses TypeScript's safety. If a key is absent, the access returns `undefined` silently even though the type says `string`.
- **Suggestion**: Define a `GithubIdentifiers` interface with explicit named fields and use it instead of `Record<string, string>`.
- **Evidence**:

```typescript
const owner = params.parsed.identifiers["owner"];
const repo = params.parsed.identifiers["repo"];
const number = params.parsed.identifiers["number"];
```

---

### Finding 5: GithubClient = Octokit alias

- **File**: `src/integrations/platforms/github/providers/github-client.provider.ts:30`
- **Category**: sdk-type-duplication
- **Impact**: none
- **Description**: `export type GithubClient = Octokit` — correctly re-exports the SDK type directly rather than duplicating its shape. This is the right pattern.
- **Suggestion**: No changes needed.
