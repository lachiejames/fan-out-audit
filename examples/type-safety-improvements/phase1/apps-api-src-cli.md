# Audit: apps-api-src-cli

**Files inspected**: 2
**Findings**: 3

## Summary

`seed-demo.helpers.ts` contains a type cast on the Supabase `listUsers` result and a local `AuthUserRecord` interface that partially duplicates the Supabase SDK's own `User` type. `seed-demo.ts` has a small `as Record<string, unknown>` cast in `toRecord()` and a `cloneJson` helper that uses an `as T` assertion.

## Findings

### Finding 1: Unnecessary type cast on Supabase admin `listUsers` result

- **File**: `apps/api/src/cli/seed-demo.helpers.ts:61`
- **Category**: type-cast
- **Impact**: medium
- **Description**: The result of `supabase.auth.admin.listUsers()` is cast via `as Array<{ email?: string; id: string }>`. The Supabase JS SDK's `AdminUserAttributes` / `User` type already contains `email?: string` and `id: string`. The cast is hiding the real SDK type and risks silently stripping any additional fields that the SDK returns (such as `email_confirmed_at`, `last_sign_in_at`) which might be relevant for future use.
- **Suggestion**: Import and use the `User` type from `@supabase/supabase-js` directly. The function return type can remain `{ email: string; id: string } | null` while the intermediate variable uses the SDK type, removing the cast entirely.
- **Evidence**:

```typescript
const users = data.users as Array<{ email?: string; id: string }>;
```

### Finding 2: Local `AuthUserRecord` interface duplicates Supabase SDK `User`

- **File**: `apps/api/src/cli/seed-demo.helpers.ts:80-83`
- **Category**: sdk-type-duplication
- **Impact**: low
- **Description**: `AuthUserRecord` (`{ email?: string; id: string }`) is a narrow hand-rolled copy of the Supabase SDK's `User` type. If the SDK's shape changes (e.g., `email` becomes non-optional), this local type will silently diverge.
- **Suggestion**: Either use `Pick<User, 'email' | 'id'>` from `@supabase/supabase-js` (which keeps the exact SDK optionality), or keep the local interface but add a comment explaining the intentional narrowing so reviewers know it is deliberate.
- **Evidence**:

```typescript
interface AuthUserRecord {
  email?: string;
  id: string;
}
```

### Finding 3: `as Record<string, unknown>` cast in `toRecord`

- **File**: `apps/api/src/cli/seed-demo.ts:153`
- **Category**: type-cast
- **Impact**: low
- **Description**: After verifying `typeof value === 'object' && value !== null`, the value is cast with `as Record<string, unknown>`. TypeScript already narrows `value` to `object` at that point, so the cast suppresses a type error rather than being redundant. Using `satisfies` or an explicit assertion with an explanation would make the intent clearer.
- **Suggestion**: Replace with `return value as Record<string, unknown>` kept but documented, or use a type predicate helper. Alternatively, parameter type could be widened to `Record<string, unknown> | null | undefined` to avoid the cast.
- **Evidence**:

```typescript
function toRecord(value: unknown): Record<string, unknown> | null {
  if (typeof value !== "object" || value === null) {
    return null;
  }
  return value as Record<string, unknown>;
}
```
