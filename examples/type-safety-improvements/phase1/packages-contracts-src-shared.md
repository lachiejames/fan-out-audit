# Audit: packages-contracts-src-shared

**Files inspected**: 1
**Findings**: 2

## Summary

`shared/ids.ts` is a small but important file that defines branded ID types (`ContentId`, `ExternalId`, `IntegrationId`, `ThreadId`) and their constructor functions. The file is clean and well-structured. The main concern is that the constructor functions use type casts (`as ContentId`) and take named object params — both correct per the codebase patterns — but there is no Zod validation backing these brands at the boundary.

## Findings

### Finding 1: Brand constructor functions use type casts (`as ContentId`) without runtime validation

- **File**: `packages/contracts/src/shared/ids.ts:8`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `asContentId`, `asExternalId`, `asIntegrationId`, and `asThreadId` all use type casts (`value as ContentId`) to create branded types from plain strings. There is no runtime validation that the string is non-empty, a valid UUID, or otherwise meaningful before being branded. A caller can accidentally brand an empty string or malformed ID and TypeScript will not catch it.
- **Suggestion**: Add `z.string().min(1)` or `z.uuid()` validation before the cast where the ID is expected to be a UUID. For example:
  ```ts
  export const asContentId = ({ value }: { value: string }): ContentId => {
    if (!value) throw new Error("ContentId must be non-empty");
    return value as ContentId;
  };
  ```
  Alternatively, expose Zod schemas (`contentIdSchema = z.uuid().brand<"ContentId">()`) that clients can use at API boundaries to get both validation and branding in a single step.
- **Evidence**: `export const asContentId = ({ value }: { value: string }): ContentId => value as ContentId;` at line 8.

### Finding 2: No Zod schemas exported for branded ID types — brand is TypeScript-only

- **File**: `packages/contracts/src/shared/ids.ts`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The branded ID types (`ContentId`, `ExternalId`, `IntegrationId`, `ThreadId`) exist only as TypeScript types and imperative constructor functions. Zod v4 supports branded schemas via `.brand<"Tag">()`. Without Zod schemas, any component that receives `unknown` data (e.g., API response parsing, event handlers) cannot use Zod to create properly branded IDs — it must use the raw `asXxx()` constructors after manual type narrowing.
- **Suggestion**: Export Zod schemas for each branded type:
  ```ts
  export const contentIdSchema = z.string().min(1).brand<"ContentId">();
  export const integrationIdSchema = z.uuid().brand<"IntegrationId">();
  ```
  These can coexist with the existing constructor functions and provide a safer path for parsing branded IDs at API response boundaries.
- **Evidence**: No Zod schema exports in `shared/ids.ts`.
