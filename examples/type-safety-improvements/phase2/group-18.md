# Cross-Cutting Patterns (Group 18)

**Slices analyzed**: packages-contracts-src-contracts-2, packages-contracts-src-contracts-3, packages-contracts-src-contracts-4, packages-contracts-src-contracts-5, packages-contracts-src-contracts-11, packages-contracts-src-contracts-12, packages-contracts-src-contracts-13, packages-contracts-src-contracts-14, packages-contracts-src-contracts-15, packages-contracts-src-contracts-16, packages-contracts-src-contracts-17, packages-contracts-src-contracts-18

## Pattern 1: `z.record(z.string(), z.unknown())` as the default metadata/payload shape

- **Seen in**: slices 2, 3, 5, 12, 13, 15, 16, 18
- **Category**: any-usage / record-weakening
- **Combined impact**: high -- this is the single most pervasive type-safety gap in the contracts package
- **What's happening**: Across nearly every domain (billing, chat, content, entities, integrations, knowledge-sources, sync-progress, test-fixtures, work-items), `z.record(z.string(), z.unknown())` is used for metadata, payload, and configuration fields. Examples include `actionTransactionSchema.metadata`, `baseContentSchema.metadata`, `integrationActionPayloadSchema`, `captureMetadata`, `knowledgeSourceItemResponseSchema.metadata`, `createMessageBodySchema.platformIdentifiers`, `baseWorkItemSchema.aiSources`, and `selectedScopeSchema.params`. In most cases, the actual runtime shapes are well-known (platform-specific metadata, action payloads, AI source references) but the schema treats them as opaque blobs.
- **Suggestion**: Adopt a systematic approach: (1) For fields where the shape varies by a discriminant (e.g., platform, action type), define discriminated union sub-schemas and narrow via the discriminant. (2) For fields with a single known shape, define an explicit object schema. (3) For genuinely dynamic fields (e.g., Asana custom fields, Monday column values), narrow the value type to `z.union([z.string(), z.number(), z.boolean(), z.null()])` instead of `z.unknown()`. Start with the highest-traffic paths: `integrationActionPayloadSchema`, `baseContentSchema.metadata`, and `baseWorkItemSchema.aiSources`.
- **Estimated scope**: 15-20 schema fields across 10+ files

## Pattern 2: `z.string()` for timestamp fields instead of `isoDateTimeStringSchema`

- **Seen in**: slices 4, 14, 15, 17, 18
- **Category**: missing-strict-typing
- **Combined impact**: medium -- timestamps pass without format validation at API boundaries
- **What's happening**: Multiple contracts use raw `z.string()` for timestamp fields (createdAt, updatedAt, completedAt, startedAt, exportedAt, approvedAt, executedAt, expiresAt, scheduledFor, undoDeadline, lastInteraction, linkedAt) instead of the repo-wide `isoDateTimeStringSchema` alias. The convention exists and is used in most contracts, but entities, memory export, push-notifications, refresh-all, and work-items all deviate. Slice 15 also found `z.iso.datetime()` used directly instead of the alias.
- **Suggestion**: Search-and-replace all `z.string()` timestamp fields to use `isoDateTimeStringSchema` from `@/schemas.ts`. Also replace direct `z.iso.datetime()` calls with the alias. This is a mechanical refactor that can be done in a single PR.
- **Estimated scope**: 15-20 timestamp fields across 5-6 files (entities/schemas.ts, memory/export-all.ts, push-notifications/schemas.ts, user/settings/refresh-all.ts, work-items/schemas.ts)

## Pattern 3: `z.string()` for platform ID fields instead of a validated platform schema

- **Seen in**: slices 3, 4, 5, 15, 17
- **Category**: missing-strict-typing
- **Combined impact**: medium -- platform IDs are unvalidated strings throughout the contract layer
- **What's happening**: `entityPlatformSchema`, `citationPlatformSchema`, domain event platform fields, `suggestionSourceItemSchema.platform`, `filterConfigSchema.platforms`, `baseSessionSchema.platforms`, and `startSessionBodySchema.platforms` all use bare `z.string()` for platform identifiers. The comment "string for 100+ integration scalability (ADR-002)" appears in several places, indicating this is an intentional design decision. However, the existing `integrationPlatformSchema` (which is `z.string().min(1)`) is not even used consistently -- it is re-declared locally in `sync-progress/schemas.ts` (slice 16).
- **Suggestion**: (1) Consolidate on the canonical `integrationPlatformSchema` from `contracts/integrations/schemas.ts` -- remove the duplicate in sync-progress. (2) At minimum, add `.min(1)` to all bare `z.string()` platform fields to prevent empty strings. (3) At the TypeScript level, use `PlatformId | (string & {})` to provide autocomplete for known platforms while remaining open. (4) For array fields like `platforms: z.array(z.string())`, use `z.array(integrationPlatformSchema)`.
- **Estimated scope**: 8-12 platform fields across 6-8 files

## Pattern 4: Duplicate schema/type definitions across module boundaries

- **Seen in**: slices 2, 4, 11, 13, 16, 17
- **Category**: duplicate-type
- **Combined impact**: high -- silent drift risk when enums or schemas are updated in one location but not the other
- **What's happening**: Multiple schemas and types are defined independently in two or more places with identical values: (1) `billingActionSchema` vs `ACTION_COSTS` keys (slice 2), (2) `chatRoleSchema` in conversations vs chat (slice 4), (3) `teamsChatTypeSchema` vs inline enum in expanded-data (slice 11), (4) `KnowledgeStatus` in knowledge/schemas.ts vs knowledge/types.ts (slice 13), (5) `integrationPlatformSchema` in integrations/schemas.ts vs sync-progress/schemas.ts (slice 16), (6) `knowledgeFactSchema.category` inline enum vs `knowledgeCategorySchema` (slice 16), (7) `TriageSessionStatus`/`TriageItemStatus` in triage/schemas.ts vs triage/constants.ts (slice 17), (8) `SubscriptionProductConfig.tier` inline union vs `SubscriptionTier` type (slice 2).
- **Suggestion**: For each duplicate pair, choose one canonical source and have the other import from it. The pattern should be: schemas own the Zod definition, types files re-export `z.infer<>`, and constants files import from schemas. Derive enums from source-of-truth objects where possible (e.g., `billingActionSchema` from `ACTION_COSTS` keys). This is a medium-effort refactor but eliminates a class of silent bugs.
- **Estimated scope**: 8 duplicate pairs across 12+ files

## Pattern 5: Structurally identical expanded-data schemas without a shared base

- **Seen in**: slices 11, 12
- **Category**: duplicate-type
- **Combined impact**: medium -- four or more platform schemas share identical field sets
- **What's happening**: (1) `outlookExpandedDataSchema` and `gmailExpandedDataSchema` have identical email thread shapes differing only in the `platform` literal (slice 11). (2) `mondayExpandedDataSchema`, `googleDocsExpandedDataSchema`, `googleDriveExpandedDataSchema`, and `oneDriveExpandedDataSchema` are four copies of the same document-card shape (`platform`, `sourceUrl`, `summary`, `title`) differing only in the platform literal (slice 12).
- **Suggestion**: Extract shared base schemas: (1) `baseEmailExpandedDataSchema` with `anchorMessageId`, `pagination`, `thread` fields, extended per platform with `platform: z.literal(...)`. (2) `baseDocumentCardExpandedDataSchema` with `sourceUrl`, `summary`, `title` fields, extended per platform. This reduces duplication from 6 schemas to 2 bases + 6 thin extensions.
- **Estimated scope**: 6 files (2 email platforms, 4 document-card platforms)

## Pattern 6: `.loose()` used pervasively without consistent rationale

- **Seen in**: slices 3, 5, 11, 14
- **Category**: missing-strict-typing
- **Combined impact**: medium -- unknown fields silently pass through API boundaries
- **What's happening**: `.loose()` (Zod passthrough) is applied to chat UI part schemas, chat stream request schemas, pending platform metadata, OAuth authorize query schemas, and most Teams/Slack expanded-data sub-schemas. The rationale varies: forward-compat with AI SDK (chat), OAuth provider extensions (OAuth), SDK extra fields (Teams/Slack). However, `teamsReactionSchema` is missing `.loose()` while all sibling schemas have it (slice 11), and `chatStreamRequestSchema` uses `.loose()` where it arguably should be strict (slice 3).
- **Suggestion**: (1) Establish a documented policy: `.loose()` is for schemas representing external data (SDK responses, OAuth callbacks); internal request schemas should be `.strict()`. (2) Add `.loose()` to `teamsReactionSchema` for consistency with siblings. (3) Remove `.loose()` from `chatStreamRequestSchema` (an internal request, not external data). (4) Add inline comments on every `.loose()` call explaining why it is needed.
- **Estimated scope**: 10-15 schemas across 5-6 files

## Pattern 7: Anonymous inline `userIdentity` schemas on platform metadata

- **Seen in**: slices 12 (Notion, Monday)
- **Category**: missing-strict-typing
- **Combined impact**: low -- pattern affects only 2 platforms currently but will grow with new integrations
- **What's happening**: `notionMetadataSchema` and `mondayMetadataSchema` both define `userIdentity` as anonymous inline `z.object(...)` schemas rather than named, exported, reusable schemas. Other platforms (Jira, Linear) follow the pattern of exporting a named `*UserIdentitySchema`. The Monday schema also uses `z.string()` for the email field instead of `z.email()`.
- **Suggestion**: Extract `notionUserIdentitySchema` and `mondayUserIdentitySchema` as named exports with `.strict()`. Use `z.email()` for email fields. Consider defining a `baseUserIdentitySchema` with common fields (`userId`, `name`) that platform-specific schemas extend.
- **Estimated scope**: 2 files currently, but establishing the pattern prevents the issue in future integrations

## Pattern 8: `z.string()` for email fields instead of `z.email()`

- **Seen in**: slices 11, 12
- **Category**: missing-strict-typing
- **Combined impact**: low -- email format validation missing on a handful of fields
- **What's happening**: `oneDriveMetadataSchema`, `outlookMetadataSchema`, `microsoftCalendarMetadataSchema`, and `mondayMetadataSchema` all use `email: z.string()` for fields that contain validated email addresses from platform APIs. Zod v4 provides `z.email()` for format validation.
- **Suggestion**: Replace `email: z.string()` with `email: z.email()` across all platform metadata schemas. This is a one-line change per file.
- **Estimated scope**: 4-5 fields across 3-4 files

## Pattern 9: `z.string().url()` vs `z.url()` inconsistency

- **Seen in**: slice 15
- **Category**: missing-strict-typing
- **Combined impact**: low -- cosmetic inconsistency but signals incomplete Zod v4 migration
- **What's happening**: Suggested-actions step detail schemas use `z.string().url()` (Zod v3 pattern) rather than `z.url()` (Zod v4 canonical form). This was only found in one slice but likely exists elsewhere in the codebase given the size of the contracts package.
- **Suggestion**: Audit all `z.string().url()` usages codebase-wide and replace with `z.url()`. This ensures consistent Zod v4 idioms and slightly improves error messages (Zod v4's `z.url()` provides better diagnostics).
- **Estimated scope**: Unknown codebase-wide; at least 5-6 instances in suggested-actions alone

## Pattern 10: Empty object schemas without `.strict()`

- **Seen in**: slices 15, 18
- **Category**: missing-strict-typing
- **Combined impact**: low -- sentinel schemas silently accept extra fields
- **What's happening**: `archiveStepDetailsSchema`, `ignoreStepDetailsSchema` (slice 15), and `deleteVocabularyItem` body (slice 18) use `z.object({})` without `.strict()`. The repo convention `emptyBodySchema` from `@/schemas.ts` provides `z.object({}).strict().optional().default({})`. Without `.strict()`, these schemas silently strip unexpected fields rather than rejecting them, which can mask bugs in discriminated union construction.
- **Suggestion**: Replace bare `z.object({})` with `emptyBodySchema` from `@/schemas.ts` for endpoint bodies, and add `.strict()` for empty sentinel schemas in discriminated unions.
- **Estimated scope**: 3-4 instances across 2 files
