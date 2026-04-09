# Type-Safety Audit: apps-app-src-hooks-4

Files audited:

- `apps/app/src/hooks/useChatToolsTimeline.ts`
- `apps/app/src/hooks/useChatTransport.ts`
- `apps/app/src/hooks/useEntitiesData.ts`
- `apps/app/src/hooks/useExpandedContent.ts`

---

## Finding 1

**File:** `apps/app/src/hooks/useChatToolsTimeline.ts` (lines 35, 88)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Medium

**Description:**
`msg.parts` is cast to `MessagePart[]` twice — once in each of two helper functions. The `parts` field on an AI SDK message is typed as a union of part types, but the cast bypasses that union and assumes the caller knows which variant is present. If the AI SDK changes its parts shape, these casts will silently break runtime behavior.

**Suggestion:**
Use a type predicate or discriminated union check to narrow each part before mapping. Alternatively, augment the `UIMessage` type at its declaration site so `parts` carries the correct type without needing a cast here.

**Evidence:**

```typescript
const parts = msg.parts as MessagePart[]; // line 35
const parts = msg.parts as MessagePart[]; // line 88
```

---

## Finding 2

**File:** `apps/app/src/hooks/useChatToolsTimeline.ts` (lines 68, 116)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Medium

**Description:**
The AI SDK's `UIMessage` is extended via inline intersection casts to access `serverTimestamp` and `isVoiceTranscript` fields. This is a symptom of the `UIMessage` type not being augmented at a central location — instead, the same pattern must be repeated in every hook that needs these fields.

**Suggestion:**
Augment `UIMessage` at the `ai-chat-types.ts` level or via module augmentation so that `serverTimestamp?: string` and `isVoiceTranscript?: boolean` are official fields on the local message type. Eliminate all inline intersection casts to these properties.

**Evidence:**

```typescript
(msg as UIMessage & { serverTimestamp?: string })(msg as UIMessage & { isVoiceTranscript?: boolean });
```

---

## Finding 3

**File:** `apps/app/src/hooks/useChatTransport.ts`

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Medium

**Description:**
The `currentContext` parameter is typed as `unknown` (or similarly loose), requiring all downstream access to go through an `isRecord` helper for narrowing. While technically sound, this pushes the type-narrowing burden to every consumer instead of declaring the known shape upfront.

**Suggestion:**
Type `currentContext` explicitly using `CurrentContext` from `@/lib/ai-chat-types`, which already describes the full shape. This eliminates the need for downstream narrowing calls.

**Evidence:**

```typescript
currentContext?: unknown
```

---

## Finding 4

**File:** `apps/app/src/hooks/useEntitiesData.ts`

**Category:** `any` or unsafe `unknown` in production code

**Impact:** High

**Description:**
Five mutation methods — `acceptMerge`, `rejectMerge`, `toggleVIP`, `updateNotes`, and `manualMerge` — expose `Promise<unknown>` as their `mutateAsync` return type. Callers that need to inspect the resolved value (e.g., to extract the updated entity ID) have no type-safe way to do so.

**Suggestion:**
Derive explicit return types from the ts-rest contract response schemas. For example, `acceptMerge` should return `Promise<EntityMergeResponse>` (or whatever the 200 body type is). Use `typeof tsr.entities.acceptMerge.mutate` result type as the authoritative source.

**Evidence:**

```typescript
mutateAsync: (...) => Promise<unknown>  // acceptMerge, rejectMerge, toggleVIP, updateNotes, manualMerge
```

---

## Finding 5

**File:** `apps/app/src/hooks/useEntitiesData.ts`

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Low

**Description:**
The return type includes an `internal?: unknown` field, which is an opaque escape hatch that hides whatever data is actually stored there. Callers can't use this field without a cast.

**Suggestion:**
Either type `internal` explicitly with its actual shape, or remove it if it is unused.

**Evidence:**

```typescript
internal?: unknown
```

---

## Finding 6

**File:** `apps/app/src/hooks/useExpandedContent.ts` (lines 57–58, 62)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Medium

**Description:**
Three casts appear in the query function: `contentId as string` (because the parameter type may be `string | undefined`), `platform as SupportedPlatform` (narrowing from a broader string), and `data.body as ExpandedData` (unwrapping the ts-rest envelope). The `(undefined as never)` workaround on an early-return path suppresses a TypeScript narrowing gap.

**Suggestion:**

- Guard `contentId` with an explicit `if (!contentId) return` check before use, which eliminates the `as string` cast.
- Import `SupportedPlatform` from `@slopweaver/contracts` and validate with a type predicate rather than casting.
- Use `select: (d) => d.body` on the `tsr.*.useQuery` call to unwrap the ts-rest envelope at the query layer, removing the `as ExpandedData` cast.

**Evidence:**

```typescript
contentId as string;
platform as SupportedPlatform;
undefined as never;
data.body as ExpandedData;
```
