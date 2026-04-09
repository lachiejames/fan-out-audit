# Type-Safety Audit: apps/api/src/interface (Slice 7)

**Files audited**: `calendar/calendar-response.utils.ts`, `chat/chat-citations.utils.ts`, `chat/chat-pipeline.ts`

---

## calendar/calendar-response.utils.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line     | Cast                                        | Concern                                                                                                                                                                                                                                                                     |
| -------- | ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 113      | `platform as CalendarEventItem["platform"]` | After a validated Zod parse or DB result, TypeScript cannot narrow `string` to the union literal. Acceptable if the value was already validated at an earlier boundary. If not, a `z.enum()` parse would eliminate the cast.                                                |
| 101, 107 | `event.integrationId!` (non-null assertion) | The integration ID is known to be non-null here because of prior filtering, but there is no in-scope type guard making this explicit. Extracting a typed sub-type (e.g. `type SyncedCalendarEvent = CalendarEvent & { integrationId: string }`) would remove the assertion. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## chat/chat-citations.utils.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line | Cast                    | Concern                                                                                                                                                                                                                     |
| ---- | ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 153  | `as CitationSourceType` | Cast follows a successful `citationPlatformSchema.safeParse(platform)`. After `parse.success === true`, `parse.data` is already narrowed; casting the original `platform` variable is redundant. Use `parse.data` directly. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## chat/chat-pipeline.ts

### 1. Custom Types Duplicating SDK Types

| Location | Duplicate                                                                                    | Notes                                                                                                                                                        |
| -------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Line 125 | `toolSafetyConversationHistory: Array<{ content: string; role: string }>` (inline anonymous) | This shape is an exact duplicate of `ConversationHistoryMessage` defined in `chat-tool-safety-history.utils.ts`. The named type should be imported and used. |

### 2. Type Casts That Could Be Avoided

| Line | Cast                                                                 | Concern                                                                                                                                                                                                                                                                                        |
| ---- | -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 261  | `model = deps.aiModelPort.createClaudeModel({...}) as LanguageModel` | `createClaudeModel` returns a Claude-specific model type. The `as LanguageModel` cast widens it to the Vercel AI SDK's generic interface. This is intentional to allow heterogeneous model selection, but the port's return type could declare `LanguageModel` directly to eliminate the cast. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

| Line | Usage                                                    | Concern                                                                                                                                                                        |
| ---- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 126  | `tools: Record<string, unknown>` in `ChatPipelineResult` | This represents the AI SDK tool set. The Vercel AI SDK exports a `ToolSet` type (`Record<string, Tool>`) that is more precise. Consider using `ToolSet` from the `ai` package. |

### 5. Duplicate Type Definitions

| Location    | Duplicate                                                 | Action                                                                                                                                                                                                                                                             |
| ----------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Lines 69–75 | `LoggerLike` interface (4 occurrences across chat module) | This exact interface appears identically in `chat-stream-callbacks.utils.ts`, `chat-stream-guards.utils.ts`, and `chat-stream.handler.ts`. Extract to a single shared location (e.g. `interface/http/chat/chat-logger.types.ts` or `shared/types/logger-like.ts`). |
| Line 125    | `Array<{ content: string; role: string }>`                | Duplicates `ConversationHistoryMessage` from `chat-tool-safety-history.utils.ts`. Import and use the named type.                                                                                                                                                   |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## Summary

| Severity | Finding                                                                              | File                              | Lines    |
| -------- | ------------------------------------------------------------------------------------ | --------------------------------- | -------- |
| High     | `LoggerLike` duplicated 4× across chat module — extract to shared file               | `chat-pipeline.ts` (and 3 others) | 69–75    |
| High     | `ConversationHistoryMessage` already defined; inline type at line 125 is a duplicate | `chat-pipeline.ts`                | 125      |
| Medium   | `tools: Record<string, unknown>` should use `ToolSet` from `ai` package              | `chat-pipeline.ts`                | 126      |
| Medium   | `as LanguageModel` cast avoidable if port return type is `LanguageModel`             | `chat-pipeline.ts`                | 261      |
| Low      | `as CitationSourceType` cast redundant — use `parse.data` directly                   | `chat-citations.utils.ts`         | 153      |
| Low      | `event.integrationId!` avoidable with a narrowed sub-type                            | `calendar-response.utils.ts`      | 101, 107 |
| Low      | `platform as CalendarEventItem["platform"]` should use Zod parse result              | `calendar-response.utils.ts`      | 113      |
