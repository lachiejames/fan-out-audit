# Type-Safety Audit — Batch 95

**Files audited:**

- `apps/api/src/integrations/platforms/slack/utils/slack-sync.utils.ts`
- `apps/api/src/integrations/platforms/slack/utils/slack-transform.utils.ts`
- `apps/api/src/integrations/services/integrations.service.ts`
- `apps/api/src/integrations/services/pending-integrations.service.ts`

---

## Findings

### 1. slack-transform.utils.ts — type-cast

**Line:** 48
**Category:** type-cast
**Severity:** medium
**Description:** `Object.assign(msg, { files, reactions }) as ContractMessage` — cast after injecting `files` and `reactions` fields that are not in the base `ContractMessage` type. The `Object.assign` modifies the object in place but TypeScript doesn't track the augmented shape.
**Code:**

```typescript
const enriched = Object.assign(msg, { files, reactions }) as ContractMessage;
```

**Fix:** Use an explicit spread instead of `Object.assign`: `const enriched: ContractMessage = { ...msg, files, reactions }` and add `files` and `reactions` fields to the `ContractMessage` type in `@slopweaver/contracts`.

---

### 2. slack-transform.utils.ts — type-cast

**Lines:** 60, 68
**Category:** type-cast
**Severity:** medium
**Description:** `channel as ContractChannel` and `user as ContractUser` — casts after `typeof id === "string"` runtime validation. The runtime check narrows the `id` field but doesn't narrow the broader channel/user shape.
**Code:**

```typescript
if (typeof channel.id === "string") {
  return channel as ContractChannel;
}
```

**Fix:** Add type guards `isContractChannel(c: unknown): c is ContractChannel` and `isContractUser(u: unknown): u is ContractUser` that validate the required fields, replacing the casts.

---

### 3. integrations.service.ts — record-weakening

**Lines:** 522–528
**Category:** record-weakening
**Severity:** medium
**Description:** `const updateSet: Record<string, unknown>` used to build a dynamic DB column update object. The keys should be constrained to valid column names of the integrations table.
**Code:**

```typescript
const updateSet: Record<string, unknown> = {};
if (params.accessToken !== undefined) updateSet["accessToken"] = encrypt(...);
if (params.refreshToken !== undefined) updateSet["refreshToken"] = encrypt(...);
```

**Fix:** Use `Partial<typeof integrationsTable.$inferInsert>` as the type for the update object, which constrains keys to valid column names.

---

### 4. integrations.service.ts — type-cast

**Line:** 735
**Category:** type-cast
**Severity:** medium
**Description:** `updates.syncSettings as typeof integration.syncSettings` — cast to force merge of sync settings objects. The `syncSettings` column stores JSON and has a `typeof integration.syncSettings` type that may not fully align with the `updates.syncSettings` type.
**Code:**

```typescript
syncSettings: updates.syncSettings as typeof integration.syncSettings,
```

**Fix:** Define a `SyncSettings` Zod schema, parse the incoming settings with it, and use the parse result as the update value without casting.

---

## No Findings

- `slack-sync.utils.ts` — Clean utility functions with proper types, no findings.
- `pending-integrations.service.ts` — Clean service with properly typed Result returns, no significant findings.
