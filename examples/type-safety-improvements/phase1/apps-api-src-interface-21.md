# Audit: apps-api-src-interface-21

**Findings count**: 3
**Summary**: Three issues in `webhook.processor.ts`. The most impactful is that job routing via `switch (job.name)` does not narrow the discriminated union, forcing ten explicit `as Job<SpecificType>` casts. A single unsafe cast accesses `payload.data.organizationId` on a `Record<string, unknown>` field. Two casts widen `platformMetadata` from DB JSON to `Record<string, unknown>`.

---

## Finding 1

**File**: `apps/api/src/interface/webhooks/processors/webhook.processor.ts`
**Lines**: 210–253 (representative; casts appear for each `case` branch)
**Category**: Type cast (`as`)
**Impact**: High — ten job-specific casts (`job as Job<SlackWebhookJob>`, `job as Job<GmailWebhookJob>`, etc.) are required because TypeScript cannot narrow a `Job<UnionType>` through `switch (job.name)`. This is a structural gap: adding a new job type requires manually tracking both the switch case and the cast.

**Description**: The `WebhookJob` union is defined correctly, but `Job<WebhookJob>` is not discriminated on `job.name` because `Job<T>` wraps the data rather than spreading it. After each `case "process-slack":` branch the processor must cast the job to the specific subtype to access typed `data` fields.

**Suggestion**: The cleanest fix is to define the union on the `name` field itself as a discriminant alongside the data, and use a type-narrowing helper:

```typescript
function asSlackJob(job: Job<WebhookJob>): Job<SlackWebhookJob> {
  return job as Job<SlackWebhookJob>;
}
```

…or accept the casts as unavoidable given BullMQ's generic `Job<T>` and document them with a single comment per cast. The lower-effort mitigation is to centralise all casts in one type-narrowing module rather than inlining them in the processor.

**Evidence**:

```typescript
case "process-slack": {
  const slackJob = job as Job<SlackWebhookJob>;
  // ...
}
case "process-gmail": {
  const gmailJob = job as Job<GmailWebhookJob>;
  // ...
}
// ... repeated for 8 more job types
```

---

## Finding 2

**File**: `apps/api/src/interface/webhooks/processors/webhook.processor.ts`
**Line**: 348
**Category**: Type cast (`as`)
**Impact**: Medium — `payload.data` is typed as `Record<string, unknown>` (the BullMQ job data field after the job cast). Accessing `.organizationId` via a structural cast `as { organizationId?: string }` is valid TypeScript but skips runtime validation. A missing or wrong-typed field returns `undefined` silently.

**Description**: After casting the Lemon Squeezy job payload data to a structural inline type to extract `organizationId`, there is no runtime check that the field is actually a string.

**Suggestion**: Use a type guard or optional-chaining with `typeof` check:

```typescript
const orgId = typeof payload.data?.organizationId === "string" ? payload.data.organizationId : undefined;
```

**Evidence**:

```typescript
const { organizationId } = payload.data as { organizationId?: string };
```

---

## Finding 3

**File**: `apps/api/src/interface/webhooks/processors/webhook.processor.ts`
**Lines**: 647, 704
**Category**: Type cast (`as`)
**Impact**: Low — `integration.platformMetadata` is a DB JSON column that Drizzle infers as `unknown`. Casting to `Record<string, unknown> | null` is the correct approach to safely access it, and is standard in this codebase. Flagged for completeness; no fix needed unless the metadata shape is formalized.

**Description**: Two casts widen the untyped `platformMetadata` DB JSON to `Record<string, unknown> | null` before property access. This is an accepted pattern for untyped JSON columns; however if the metadata shape is known for each platform, typed interfaces would add compile-time safety.

**Suggestion**: Define per-platform metadata interfaces (e.g. `GmailPlatformMetadata`, `SlackPlatformMetadata`) and narrow via a type guard after the `Record<string, unknown>` cast.

**Evidence**:

```typescript
const meta = integration.platformMetadata as Record<string, unknown> | null;
```
