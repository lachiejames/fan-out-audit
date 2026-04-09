# Audit: packages-contracts-src-contracts-10

**Files inspected**: 8
**Findings**: 6

## Summary

This slice covers Jira, Linear, LinkedIn, and Microsoft Calendar. Linear's expanded-data file is a large, well-structured example of the explicit-interface pattern. The main concerns are: `reactionData: unknown[]` on `LinearIssue`, `Record<string, unknown>` for Jira custom fields (acceptable but could be constrained), the `LinkedinExpandedData` using `z.infer` on a simple schema (no explicit interface, which is fine for a flat schema), and the `linearSyncBodySchema` missing `.strict()`.

## Findings

### Finding 1: `reactionData: unknown[]` on `LinearIssue` — overly loose element type

- **File**: `packages/contracts/src/contracts/integrations/platforms/linear/expanded-data.schema.ts:181`
- **Category**: any-usage
- **Impact**: medium
- **Description**: The `LinearIssue` interface declares `reactionData?: unknown[] | undefined` and the corresponding Zod schema uses `z.array(z.unknown())`. The `reactions` field immediately below uses the strongly-typed `LinearReaction[]`. These two fields likely represent the same data (raw SDK reaction objects vs. typed reaction objects). Having both as `unknown[]` and `LinearReaction[]` is confusing and the raw one provides no type safety.
- **Suggestion**: Either remove `reactionData` in favour of the typed `reactions` field, or define a `LinearRawReaction` interface for it. If it serves a different purpose, add a comment explaining why it must remain untyped.
- **Evidence**:
  ```ts
  reactionData?: unknown[] | undefined;   // line 181
  reactions?: LinearReaction[] | undefined; // line 182
  ```

### Finding 2: `LinearState.type` uses `z.string()` — known enum values exist

- **File**: `packages/contracts/src/contracts/integrations/platforms/linear/expanded-data.schema.ts:63`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `LinearState` has a `type?: string` field. Linear's API returns a fixed set of state type values: `"triage"`, `"backlog"`, `"unstarted"`, `"started"`, `"completed"`, `"canceled"`. These are already defined as `LinearWorkflowStateType` in `linear/schemas.ts`. Using the enum instead of `string` would enable exhaustive switch statements in frontend components.
- **Suggestion**: Change `type?: string` on `LinearState` to `type?: LinearWorkflowStateType` and update the corresponding Zod schema field from `z.string().optional()` to `linearWorkflowStateTypeSchema.optional()`.
- **Evidence**: `type?: string | undefined;` at line 63 in the `LinearState` interface.

### Finding 3: `JiraExpandedData.customFields` is `Record<string, unknown>` — intentional but undocumented

- **File**: `packages/contracts/src/contracts/integrations/platforms/jira/expanded-data.schema.ts:89`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `customFields?: Record<string, unknown>` is used to pass arbitrary Jira custom field values through the contract. This is intentional (Jira custom fields are project-specific and cannot be enumerated), but unlike the passthrough index signatures on other interfaces, this field silently accepts any structure. A comment documenting why `Record<string, unknown>` is appropriate here would prevent future attempts to tighten it unnecessarily.
- **Suggestion**: Add a JSDoc comment on `customFields` explaining that Jira custom field schemas are project-specific and cannot be statically typed in the contract layer.
- **Evidence**: `customFields?: Record<string, unknown> | undefined;` at line 89.

### Finding 4: `linkedinExpandedDataSchema` uses `z.infer` (no explicit interface) — acceptable but inconsistent

- **File**: `packages/contracts/src/contracts/integrations/platforms/linkedin/expanded-data.schema.ts:10`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: Unlike Linear, Jira, and Notion, the LinkedIn expanded data schema uses `z.infer` directly rather than the explicit-interface pattern. The CLAUDE.md guidelines specify using explicit interfaces for schemas in discriminated unions (which `linkedinExpandedDataSchema` participates in via the platform-level union). However, since `linkedinExpandedDataSchema` is a flat 4-field object with no passthrough, the TS7056 risk is minimal.
- **Suggestion**: For consistency with all other platform expanded-data schemas, convert to the explicit-interface pattern (`export interface LinkedinExpandedData { ... }` then `export const linkedinExpandedDataSchema: z.ZodType<LinkedinExpandedData> = ...`).
- **Evidence**: `export type LinkedinExpandedData = z.infer<typeof linkedinExpandedDataSchema>;` at line 10.

### Finding 5: `linkedinMetadataSchema` missing `.strict()` on `linkedinUserIdentitySchema`

- **File**: `packages/contracts/src/contracts/integrations/platforms/linkedin/platform-metadata.schema.ts:10`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `linkedinUserIdentitySchema` is defined inline within the LinkedIn metadata file without `.strict()` or `.loose()`. Since this is a controlled identity schema (OAuth response fields are well-known), adding `.strict()` would catch unexpected fields. This contrasts with the outer `linkedinMetadataSchema` which correctly uses `.loose()` for forward-compatibility.
- **Suggestion**: Add `.strict()` to `linkedinUserIdentitySchema` since the LinkedIn OIDC identity fields (`sub`, `name`, `email`, `picture`) are a stable, known set.
- **Evidence**: `export const linkedinUserIdentitySchema = z.object({ ... });` at line 10, no `.strict()`.

### Finding 6: `microsoftCalendarExpandedDataSchema` uses `z.infer` — inconsistent with Linear/Jira pattern

- **File**: `packages/contracts/src/contracts/integrations/platforms/microsoft-calendar/expanded-data.schema.ts:9`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `MicrosoftCalendarExpandedData` is derived via `z.infer` rather than an explicit interface. The schema delegates to `unifiedCalendarEventSchema` which may be complex. For consistency with the explicit-interface rule and to prevent future TS7056 issues as the calendar schema grows, an explicit interface is preferable.
- **Suggestion**: Define `export interface MicrosoftCalendarExpandedData { event: UnifiedCalendarEvent; platform: "microsoft-calendar"; }` and annotate the schema with `z.ZodType<MicrosoftCalendarExpandedData>`.
- **Evidence**: `export type MicrosoftCalendarExpandedData = z.infer<typeof microsoftCalendarExpandedDataSchema>;` at line 9.
