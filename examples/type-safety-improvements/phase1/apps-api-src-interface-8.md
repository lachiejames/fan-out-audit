# Type-Safety Audit: apps/api/src/interface (Slice 8)

**Files audited**: `chat/chat-stream-callbacks.utils.ts`, `chat/chat-stream-guards.utils.ts`, `chat/chat-stream.handler.ts`, `chat/chat-stream.utils.ts`

---

## chat/chat-stream-callbacks.utils.ts

### 1. Custom Types Duplicating SDK Types

| Location | Duplicate                                             | Notes                                                                                                 |
| -------- | ----------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Line 85  | `tools: Record<string, unknown>` in `OnFinishContext` | Should be `ToolSet` from the Vercel AI SDK `ai` package, matching the actual structure of tool calls. |

### 2. Type Casts That Could Be Avoided

| Line | Cast                                                                      | Concern                                                                                                                                                                                                                                                                              |
| ---- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 333  | `const toolInput = (tr as { args?: Record<string, unknown> }).args ?? {}` | `tr` is typed as `ToolResult` from the AI SDK but the `args` field is accessed without a typed discriminant. If `ToolResult` exposes `args` via a typed union, access `tr.args` directly. If the field is missing from the type, file a narrower wrapper type instead of a raw cast. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

| Line | Usage                                                 | Concern                                                                                                                      |
| ---- | ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| 85   | `tools: Record<string, unknown>` in `OnFinishContext` | Should reference `ToolSet` from the `ai` package (or the specific tool dictionary type used elsewhere in the chat pipeline). |

### 5. Duplicate Type Definitions

| Location    | Duplicate              | Action                                                                                                                                                  |
| ----------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Lines 44–51 | `LoggerLike` interface | Identical copy of the definition in `chat-pipeline.ts`, `chat-stream-guards.utils.ts`, and `chat-stream.handler.ts`. Extract to a single shared module. |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## chat/chat-stream-guards.utils.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

| Location    | Duplicate              | Action                                        |
| ----------- | ---------------------- | --------------------------------------------- |
| Lines 15–21 | `LoggerLike` interface | Third identical copy. Extract to shared file. |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## chat/chat-stream.handler.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line | Cast                                                      | Concern                                                                                                                                                            |
| ---- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 374  | `import("...").StreamWriter` (inline dynamic import type) | Using a dynamic `import()` type reference is unconventional and harder to navigate. Import the type at the top of the file with a regular `import type` statement. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

| Location    | Duplicate              | Action                                                                                                                                        |
| ----------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Lines 52–58 | `LoggerLike` interface | Fourth identical copy across the chat module. This is the primary actionable finding — extract to `interface/http/chat/chat-logger.types.ts`. |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## chat/chat-stream.utils.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line     | Cast                                                    | Concern                                                                                                                                                                                                                                                                                                     |
| -------- | ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 20       | `value as Record<string, unknown>` in `getObjectRecord` | The function receives `unknown` and casts. This is a safe widening after a `typeof value === "object"` guard. Consider adding `&& value !== null` in the guard to make the narrowing explicit before casting.                                                                                               |
| 114, 300 | `as UIMessage["parts"]`                                 | The Vercel AI SDK types `UIMessage["parts"]` as an array union. After building the array, TypeScript cannot infer it satisfies the union. The cast is structurally sound but could be replaced with `satisfies UIMessage["parts"]` which gives a compiler error instead of silently accepting wrong shapes. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## Summary

| Severity | Finding                                                                         | File                                                                                      | Lines               |
| -------- | ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------- |
| High     | `LoggerLike` duplicated for the 2nd, 3rd, and 4th time across these three files | `chat-stream-callbacks.utils.ts`, `chat-stream-guards.utils.ts`, `chat-stream.handler.ts` | 44–51, 15–21, 52–58 |
| Medium   | `tools: Record<string, unknown>` in `OnFinishContext` should use `ToolSet`      | `chat-stream-callbacks.utils.ts`                                                          | 85                  |
| Medium   | `(tr as { args?: ... }).args` cast — use proper AI SDK type                     | `chat-stream-callbacks.utils.ts`                                                          | 333                 |
| Low      | `as UIMessage["parts"]` should use `satisfies` for compile-time shape checking  | `chat-stream.utils.ts`                                                                    | 114, 300            |
| Low      | `value as Record<string, unknown>` — add `&& value !== null` guard before cast  | `chat-stream.utils.ts`                                                                    | 20                  |
| Low      | Inline `import("...").StreamWriter` — prefer top-level `import type`            | `chat-stream.handler.ts`                                                                  | 374                 |
