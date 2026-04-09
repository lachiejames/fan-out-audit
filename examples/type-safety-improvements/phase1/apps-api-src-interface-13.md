# Type-Safety Audit: apps/api/src/interface (Slice 13)

**Files audited**: `knowledge/knowledge-graphs.controller.ts`, `knowledge/knowledge.controller.ts`, `media-files/media-files-upload.controller.ts`, `media-files/media-files.controller.ts`, `memory/memory.controller.ts`, `navigation/navigation.controller.ts`, `network/network.controller.ts`, `notifications/notification.controller.ts`

---

## knowledge/knowledge-graphs.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. Follows gold-standard ts-rest + neverthrow pattern exactly.

---

## knowledge/knowledge.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. Gold-standard pattern with full CRUD + semantic search.

---

## media-files/media-files-upload.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. Correctly uses `@UploadedFile()` to receive the typed `Express.Multer.File` without casting — in contrast to the pattern noted in `knowledge-sources-upload-session.controller.ts`.

---

## media-files/media-files.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. Minimal, well-typed CRUD + signed-URL controller.

---

## memory/memory.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line(s)       | Cast                                                   | Concern                                                                                                                                                                                                                                                                                                                           |
| ------------- | ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 169, 237, 399 | `(READ_ONLY_PATHS as readonly string[]).includes(...)` | `READ_ONLY_PATHS` is imported as a `readonly` array from `@slopweaver/contracts`. The cast is needed because `includes()` on a `readonly string[]` is not widened to check any `string`. The cast is minimal and correct, but `READ_ONLY_PATHS` could be typed as `ReadonlyArray<string>` in contracts to avoid needing the cast. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

| Location      | Note                                          |
| ------------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Lines 374–382 | Inline type for the `files` array accumulator | The element shape `{ content: string; isReadOnly: boolean; lastModified: string; name: string; path: string; size: number; type: MemoryFileType }` is built identically in both `listFiles` and `exportAll`. Extracting this as a named `MemoryFileResponseItem` type would reduce repetition and allow type-checking the response shape in one place. |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## navigation/navigation.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean, minimal controller. Gold-standard pattern.

---

## network/network.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. Gold-standard pattern.

---

## notifications/notification.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. Note: uses `error.type` instead of `error.code` for log fields (consistent with the notification error discriminant — no safety issue).

---

## Summary

| Severity | Finding                                                                                                              | File                   | Lines         |
| -------- | -------------------------------------------------------------------------------------------------------------------- | ---------------------- | ------------- |
| Low      | `(READ_ONLY_PATHS as readonly string[])` — type `READ_ONLY_PATHS` as `ReadonlyArray<string>` in contracts            | `memory.controller.ts` | 169, 237, 399 |
| Low      | Inline `files` accumulator element type duplicated in `listFiles` and `exportAll` — extract `MemoryFileResponseItem` | `memory.controller.ts` | 374–382       |

All other files in this slice are clean with no type-safety issues.
