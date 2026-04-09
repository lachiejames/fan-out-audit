# Cross-Cutting Patterns (Group 19)

**Slices analyzed**: packages-contracts-src-contracts-6, packages-contracts-src-contracts-7, packages-contracts-src-contracts-8, packages-contracts-src-contracts-9, packages-contracts-src-index.ts, packages-contracts-src-schemas.ts, packages-contracts-src-shared, packages-contracts-src-types.ts, packages-contracts-src-validation-helpers.ts, packages-ui-src-atoms-1, packages-ui-src-atoms-2, packages-ui-src-atoms-3

## Pattern 1: `z.string()` used for timestamps instead of `isoDateTimeStringSchema`

- **Seen in**: contracts-6 (email timestamp), contracts-7 (syncDiagnostics timestamps), contracts-8 (GitHub createdAt/updatedAt/submittedAt, Asana createdAt/updatedAt, Facebook Messenger created_time), contracts-9 (Google metadata timestamps)
- **Category**: missing-strict-typing
- **Combined impact**: high -- appears in at least 6 slices across 15+ individual timestamp fields; the codebase already has `isoDateTimeStringSchema` in `schemas.ts` but platform schemas do not use it
- **What's happening**: Timestamp fields across email, calendar, GitHub, Asana, Facebook Messenger, Google, and sync diagnostics schemas are declared as bare `z.string()` without ISO datetime format validation. The codebase has a canonical `isoDateTimeStringSchema` but it is not consistently applied to platform-specific schemas.
- **Suggestion**: Bulk-replace all `z.string()` timestamp fields (identifiable by field names like `createdAt`, `updatedAt`, `timestamp`, `submittedAt`, `created_time`) with `isoDateTimeStringSchema`. A codebase-wide grep for `(createdAt|updatedAt|timestamp|submittedAt|created_time):\s*z\.string\(\)` in the contracts package would surface all instances.
- **Estimated scope**: ~20-25 fields across 10-12 files in `packages/contracts/src/contracts/integrations/`

## Pattern 2: `z.string()` used for email fields instead of `z.email()`

- **Seen in**: contracts-6 (emailAddressSchema.email), contracts-9 (Google Calendar/Gmail/Docs/Drive metadata `email` fields)
- **Category**: missing-strict-typing
- **Combined impact**: medium -- 5+ email fields across Google platform metadata schemas and the core email schema lack format validation
- **What's happening**: Fields explicitly named `email` that hold email addresses use `z.string()` instead of `z.email()` (Zod 4 built-in). This means malformed email strings pass validation silently.
- **Suggestion**: Replace `email: z.string()` with `email: z.email()` in all platform metadata schemas and the email expanded data schema. Grep pattern: `email:\s*z\.string\(\)` in contracts.
- **Estimated scope**: ~6-8 fields across 5-6 files

## Pattern 3: Duplicate metadata schemas between `core/schemas.ts` and `content/schemas.ts`

- **Seen in**: contracts-7 (slackMessageMetadataSchema vs slackContentMetadataSchema, gmailMessageMetadataSchema vs gmailContentMetadataSchema), contracts-9 (gmailUserIdentitySchema vs googleUserIdentitySchema)
- **Category**: duplicate-type
- **Combined impact**: high -- three pairs of near-identical schemas for Slack, Gmail, and Google identity metadata create drift risk where one is updated but not the other
- **What's happening**: Platform metadata is defined in two places -- once in the core integration schemas (for sync/messaging context) and once in the content schemas (for content rendering context). The schemas overlap heavily but have slight differences (e.g., one adds `messageCount`, the other adds `snippet`). Google identity schemas are split between `gmailUserIdentitySchema` and `googleUserIdentitySchema` with unclear distinction.
- **Suggestion**: Extract a shared base schema for each platform's metadata and extend it for context-specific needs. For Google identity, consolidate into a single `googleUserIdentitySchema` used by all Google integrations. This prevents field drift and halves the maintenance surface.
- **Estimated scope**: 3 schema pairs across 4-5 files

## Pattern 4: `Record<string, unknown>` for platform metadata fields

- **Seen in**: contracts-6 (storageFileSchema.platformMetadata, storageFolderSchema.platformMetadata), contracts-8 (AsanaExpandedData.customFields), schemas.ts (contentDtoSchema.metadata, errorResponseSchema.details)
- **Category**: record-weakening
- **Combined impact**: medium -- opaque metadata records appear in at least 5 locations; the pattern bypasses the typed metadata decode infrastructure that already exists
- **What's happening**: Several schemas use `z.record(z.string(), z.unknown())` for metadata fields. The codebase has `decodeContentMetadataStrict` and per-platform metadata interfaces, but the base schemas where metadata first appears are fully untyped. This means any code path that accesses metadata directly from the DTO (rather than through the decode function) gets no type safety.
- **Suggestion**: For `contentDtoSchema.metadata`, add JSDoc pointing to `decodeContentMetadataStrict` as the canonical typed accessor (the TODO at line 205 already acknowledges this). For file storage and Asana custom fields, define lightweight typed shapes for the known keys per platform.
- **Estimated scope**: 5-6 fields across 3-4 files

## Pattern 5: `Record<string, ...>` maps in UI where `PlatformId` union keys would be safer

- **Seen in**: atoms-2 (PLATFORM_STYLES, PLATFORM_ICONS, ExtraIconMap, PlatformIconMap cast), atoms-2 (ContextChip `platform: string` prop)
- **Category**: record-weakening
- **Combined impact**: medium -- platform-keyed lookup maps in the UI layer use `Record<string, ...>` which defeats compile-time checking of platform identifiers and allows typos to fail silently at runtime
- **What's happening**: UI components that map platform IDs to styles, icons, or other config use `Record<string, ...>` instead of `Record<PlatformId, ...>` or `Partial<Record<PlatformId, ...>>`. The `PlatformId` type is available from `@slopweaver/contracts` but not imported. Additionally, `getPlatformIcon` casts the typed map to `Record<string, ...>` to perform lookups because the incoming `platform` parameter is `string`.
- **Suggestion**: Type platform lookup maps as `Partial<Record<PlatformId, ...>>` and use `isPlatformId` type guard before lookups. For component props like `ContextChipProps.platform`, use `PlatformId | (string & {})` to preserve autocomplete while accepting arbitrary strings.
- **Estimated scope**: 3-4 maps and 1-2 component props across 2 files

## Pattern 6: Duplicate expanded-data shapes across platforms with only `platform` literal differing

- **Seen in**: contracts-9 (Google Docs, Google Drive -- and noted as also appearing in OneDrive and Monday), contracts-7 (finding 4 on duplicate calendar event shape)
- **Category**: duplicate-type
- **Combined impact**: medium -- at least 4 platform expanded-data schemas share identical field structures with only the `platform` discriminant differing; adding a new field requires updating all copies
- **What's happening**: Google Docs, Google Drive, Microsoft OneDrive, and Monday expanded-data schemas all define `{ platform: z.literal("..."), sourceUrl, summary, title }` independently. Similarly, `UnifiedCalendarEvent` maintains both an explicit interface and a Zod schema in parallel.
- **Suggestion**: Extract a `documentCardExpandedDataBaseSchema` with the shared fields and extend per platform with the `platform` literal. For `UnifiedCalendarEvent`, use `z.infer<typeof schema>` instead of maintaining a parallel interface.
- **Estimated scope**: 4-5 expanded-data schema files + 1 calendar schema file

## Pattern 7: Polymorphic `as` prop requires type casts in UI atoms

- **Seen in**: atoms-1 (Box `as React.ElementType` cast, Text `ref as never` cast)
- **Category**: type-cast
- **Combined impact**: low -- only 2 components affected, and the casts are structurally necessary due to TypeScript limitations with polymorphic components
- **What's happening**: The polymorphic `as` prop pattern in `Box` and `Text` requires type casts because TypeScript cannot infer the element type from a generic `as` prop at the render site. `Text` uses the particularly unsafe `as never` cast for the ref prop.
- **Suggestion**: Replace `ref as never` with `ref as React.Ref<HTMLElement>` in `Text` to retain at least structural expectations. Both casts should have comments explaining the TypeScript limitation. No broader refactor is warranted since this is a known TS limitation.
- **Estimated scope**: 2 files, 3 cast sites

## Pattern 8: `z.record(keySchema, valueSchema)` Zod limitation where key schema is ignored in TypeScript output

- **Seen in**: contracts-6 (budgetStatusResponseSchema.platformBreakdown with `integrationPlatformSchema` key), contracts-7 (OAuthConfig.scopesByAccessMode with `OAuthAccessMode` key), contracts-8 (PLATFORMS `satisfies Record<string, ...>`)
- **Category**: record-weakening
- **Combined impact**: low -- this is a Zod framework limitation, not a codebase bug; the key schemas provide runtime validation but TypeScript always types `z.record` keys as `string`
- **What's happening**: Several schemas use `z.record(enumOrUnionSchema, valueSchema)` intending to constrain keys to a specific set. Zod validates keys at runtime but TypeScript always infers the key type as `string`, losing compile-time key safety.
- **Suggestion**: Add comments documenting this Zod limitation at each usage site. For cases where compile-time key safety matters, consider using a manually typed `Record<SpecificKey, Value>` interface alongside the Zod schema.
- **Estimated scope**: 3-4 instances across contracts files
