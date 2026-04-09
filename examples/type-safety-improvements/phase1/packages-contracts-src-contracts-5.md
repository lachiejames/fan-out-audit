# Audit: packages-contracts-src-contracts-5

**Files inspected**: 8
**Findings**: 6

## Summary

This slice covers inbox types, the main contract router, domain events, outbox schemas, integration actions, connection sync settings, available platforms, and the calendar expanded data schema. The main contract router (`index.ts`) is clean infrastructure. Key findings: `integrationActionPayloadSchema` is a fully opaque record, `pendingPlatformMetadataSchema` uses `.loose()` with minimal structure, and there are loose `string` fields in event/action payloads where narrower types exist.

## Findings

### Finding 1: `integrationActionPayloadSchema` is `z.record(z.string(), z.unknown())` — fully opaque action payload

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/actions.ts:16`
- **Category**: any-usage
- **Impact**: high
- **Description**: `IntegrationActionPayload` is `Record<string, unknown>`. Every integration action (reply, archive, create_event, etc.) flows through this type. The lack of typing means backend handlers receive untyped payloads and must perform ad-hoc validation internally.
- **Suggestion**: Define per-action-type payload schemas (e.g., `SendReplyPayloadSchema = z.object({ body: z.string().min(1), ... })`) and create a discriminated union on `type`. This would allow Zod to validate the entire `IntegrationAction` at the boundary rather than just the wrapper. This is a significant but high-value refactor.
- **Evidence**: `export const integrationActionPayloadSchema = z.record(z.string(), z.unknown());` at line 16.

### Finding 2: `oauthCallbackCompletedPayloadSchema.platform` is `z.string()` in domain events

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/infrastructure/domain-events.schema.ts:42`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `platform: z.string()` in `oauthCallbackCompletedPayloadSchema`. This event is emitted after OAuth and consumed by multiple services. A wrong platform string would propagate silently.
- **Suggestion**: Same pattern as other platform fields — use `z.string().refine(isPlatformId)` or accept the intentional openness and add `.min(1)` at minimum.
- **Evidence**: `platform: z.string(),` at line 42.

### Finding 3: `syncContentIndexedPayloadSchema.platform` is also `z.string()`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/infrastructure/domain-events.schema.ts:68`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Same as Finding 2 for the sync content indexed payload.
- **Suggestion**: `platform: z.string().refine(isPlatformId)` or at minimum `.min(1)`.
- **Evidence**: `platform: z.string(),` at line 68.

### Finding 4: `pendingPlatformMetadataSchema` uses `.loose()` with a single required field — no structural guarantees

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/integrations/pending/schemas.ts:16`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `pendingPlatformMetadataSchema = z.object({ platform: integrationPlatformSchema }).loose()`. The `.loose()` means any extra fields pass through unvalidated. The metadata is stored in a JSONB column; without stricter validation, bad data can persist.
- **Suggestion**: This pattern is intentionally forward-compatible (comment says "Pending metadata can be partial during OAuth handshakes"). Consider adding a version of this schema without `.loose()` used strictly during write operations to catch unexpected fields, and keeping the loose version for reads.
- **Evidence**: `export const pendingPlatformMetadataSchema = z.object({ platform: integrationPlatformSchema }).loose();` at lines 16-19.

### Finding 5: `calendarEventItemSchema` uses `z.ZodType<CalendarEventItem>` annotation — interface/schema mismatch risk

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/calendar/schemas.ts:39`
- **Category**: type-cast
- **Impact**: low
- **Description**: The schema is annotated as `z.ZodType<CalendarEventItem>` where `CalendarEventItem` is an inline TypeScript `interface`. This is the correct explicit interface pattern. However, `CalendarEventItem.status` uses an inline literal union `"confirmed" | "tentative" | "cancelled" | "free"` while the Zod schema correctly references `calendarEventStatusSchema`. If `calendarEventStatusSchema` is ever updated, the interface definition must also be updated manually.
- **Suggestion**: Derive `CalendarEventItem` via `z.infer` from the schema rather than maintaining a parallel interface, or at minimum add a comment noting that the `status` union in the interface must match `calendarEventStatusSchema`.
- **Evidence**: Interface definition at lines 15-37 manually duplicates the `calendarEventStatusSchema` enum values.

### Finding 6: `ContractRouter` type in `index.ts` is manually maintained and could drift from `c.router()`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/index.ts:74-130`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `ContractRouter` is a manually declared type with 40+ fields, then assigned to `export const contract: ContractRouter = c.router({...})`. If a new contract is added to `c.router()` but not to `ContractRouter`, TypeScript will catch it because the `ContractRouter` type is used as a constraint. However, the type is redundant — `typeof contract` would provide the same information automatically.
- **Suggestion**: Remove the manual `ContractRouter` type and use `typeof contract` directly wherever the type is needed. Or use `satisfies ContractRouter` on the router object to get the constraint check without the annotation overhead.
- **Evidence**: `type ContractRouter = { accountDeletion: typeof accountDeletionContract; ... }` at lines 74-130, followed by `export const contract: ContractRouter = c.router({...})`.
