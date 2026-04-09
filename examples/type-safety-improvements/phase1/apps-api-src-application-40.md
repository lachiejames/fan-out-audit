# Audit: apps-api-src-application-40

**Files inspected**: 8
**Findings**: 7

## Summary

The voice services layer is generally well-typed with good use of discriminated union errors, `neverthrow` Results, and named object params throughout. The main improvement areas are: (1) a custom `TimestampAlignment` interface that duplicates an already-exported SDK type (`CharacterAlignmentResponseModel`), (2) several unavoidable-looking type casts that could be tightened or eliminated, (3) a round-trip `JSON.parse(JSON.stringify(...))` escape hatch used to convert a typed SDK response to `Record<string, unknown>` that loses type safety unnecessarily, (4) repeated `Record<string, unknown>` narrowing inline patterns in two functions that could share a stricter helper, and (5) a `source` string cast to a union type that the compiler cannot verify.

---

## Findings

### Finding 1: Custom `TimestampAlignment` duplicates SDK's `CharacterAlignmentResponseModel`

- **File**: `apps/api/src/application/voice/services/voice.service.ts:56-60`
- **Category**: sdk-type-duplication
- **Impact**: medium
- **Description**: `TimestampAlignment` is a hand-rolled interface with snake_case fields (`character`, `character_start_times_seconds`, `character_end_times_seconds`). The ElevenLabs SDK already exports `CharacterAlignmentResponseModel` (in `@elevenlabs/elevenlabs-js`) with the same semantic content but camelCase field names (`characters: string[]`, `characterStartTimesSeconds: number[]`, `characterEndTimesSeconds: number[]`). The custom type also models each entry as a **single character** with scalar timestamps, whereas the SDK type represents an **array** of characters with parallel timestamp arrays. This means the service manually loops over the SDK array and pushes individual entries into the custom type — a mapping that must be maintained by hand and will silently break if the SDK field names change.
- **Suggestion**: Import and use `CharacterAlignmentResponseModel` directly from `@elevenlabs/elevenlabs-js` for intermediate use, or define the flattened per-character record as `type TimestampEntry = { character: string; startSeconds: number; endSeconds: number }` with a name that makes the flattening explicit. Either way, reference the SDK source type so the compiler catches field-name drift.
- **Evidence**:

```typescript
// voice.service.ts:56-60 — custom type
export interface TimestampAlignment {
  character: string;
  character_start_times_seconds: number;
  character_end_times_seconds: number;
}

// SDK type (CharacterAlignmentResponseModel) — already exported
export interface CharacterAlignmentResponseModel {
  characters: string[];
  characterStartTimesSeconds: number[];
  characterEndTimesSeconds: number[];
}
```

---

### Finding 2: `JSON.parse(JSON.stringify(...))` round-trip erases SDK response type

- **File**: `apps/api/src/application/voice/services/voice.service.ts:231`
- **Category**: type-cast
- **Impact**: high
- **Description**: After a successful transcription response, the typed SDK value (`firstAttempt.value`, which is typed as the SDK's STT response) is immediately round-tripped through `JSON.parse(JSON.stringify(...))` and assigned to `Record<string, unknown>`. The comment says "SDK returns typed response but we need generic access for duration extraction". However, `extractDurationSeconds` and `extractDurationSecondsFromWords` in `voice-service.utils.ts` both accept `Record<string, unknown>` — so this round-trip is only necessary because those helpers take a `Record<string, unknown>` rather than the SDK type. This loses all compile-time guarantees on the response fields.
- **Suggestion**: Inspect the SDK's STT response type for the fields accessed (`text`, `words[].end`, `audio_duration_seconds`). If the SDK types them, update `extractDurationSeconds` to accept the typed SDK response (or a subset interface `{ audio_duration_seconds?: number; words?: Array<{ end?: number }> }`) and eliminate the round-trip. If the SDK response type is too loose, a narrower local interface would still be safer than `Record<string, unknown>`.
- **Evidence**:

```typescript
// voice.service.ts:231-232
// SDK returns typed response but we need generic access for duration extraction
const payload: Record<string, unknown> = JSON.parse(JSON.stringify(firstAttempt.value));
const transcript = typeof payload["text"] === "string" ? payload["text"].trim() : "";
```

---

### Finding 3: `candidateModels[index] as string` — unnecessary cast from `readonly string[]`

- **File**: `apps/api/src/application/voice/services/voice.service.ts:275, 417, 565`
- **Category**: type-cast
- **Impact**: low
- **Description**: In all three synthesis methods (`synthesize`, `synthesizeWithTimestamps`, `synthesizeStream`), the loop index accesses `candidateModels[index]` and immediately casts it `as string`. TypeScript infers `string | undefined` for indexed access on `readonly string[]` (when `noUncheckedIndexedAccess` is on) — but the cast silently drops the `undefined` without a guard. If `noUncheckedIndexedAccess` is off, the cast is a no-op. Either way it adds noise.
- **Suggestion**: Replace `candidateModels[index] as string` with an explicit guard: `const candidateModelId = candidateModels[index]; if (!candidateModelId) continue;` This makes the intent clear and doesn't suppress a potential `undefined`.
- **Evidence**:

```typescript
// voice.service.ts:275 (same pattern at 417 and 565)
const candidateModelId = candidateModels[index] as string;
```

---

### Finding 4: `chars[ci] as string` — cast on `string[]` element

- **File**: `apps/api/src/application/voice/services/voice.service.ts:480`
- **Category**: type-cast
- **Impact**: low
- **Description**: `chars` is derived from `a.characters` which is `string[]` per the SDK type. Indexing `chars[ci]` yields `string | undefined` under `noUncheckedIndexedAccess`, so the `as string` cast suppresses the `undefined` case instead of guarding it. The alignment entry pushed would contain `character: undefined as string` if the index is out of bounds.
- **Suggestion**: Either guard with `const ch = chars[ci]; if (ch === undefined) continue;` or use array destructuring inside the loop to avoid the cast.
- **Evidence**:

```typescript
// voice.service.ts:478-484
for (let ci = 0; ci < chars.length; ci += 1) {
  alignment.push({
    character: chars[ci] as string,
    character_end_times_seconds: ends[ci] ?? 0,
    character_start_times_seconds: starts[ci] ?? 0,
  });
}
```

---

### Finding 5: `STT_MIME_TO_EXTENSION` uses `Record<string, string>` where a stricter key type exists

- **File**: `apps/api/src/application/voice/services/voice-service.utils.ts:15-28`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `STT_MIME_TO_EXTENSION` is typed as `Record<string, string>`. Since the set of supported MIME types is fixed and finite, using a plain `Record<string, string>` means: (a) the lookup `STT_MIME_TO_EXTENSION[normalizedMimeType]` returns `string` rather than `string | undefined`, making callers falsely believe every key is present; (b) typos in keys are not caught at compile time.
- **Suggestion**: Use `satisfies Record<string, string>` with a string-literal union for the keys, or declare the type with `as const` and derive the key union. At minimum, change the type annotation to `Record<string, string | undefined>` (or `Partial<Record<string, string>>`) so callers are forced to handle the `undefined` case — which `voice.service.ts:143` already does with an `if (!extension)` check, meaning the current type annotation is lying.
- **Evidence**:

```typescript
// voice-service.utils.ts:15
export const STT_MIME_TO_EXTENSION: Record<string, string> = {
  "audio/flac": "flac",
  // ...
};

// voice.service.ts:143 — already guarding for undefined, contradicting the type
const extension = STT_MIME_TO_EXTENSION[normalizedMimeType];
if (!extension) {
  return err(VoiceErrors.audioInvalid(`Unsupported audio mime type: ${trimmedMimeType}.`));
}
```

---

### Finding 6: Repeated inline `Record<string, unknown>` narrowing pattern duplicated across `isKeytermsProviderError` and `isAudioFormatProviderError`

- **File**: `apps/api/src/application/voice/services/voice-service.utils.ts:228-248, 272-294`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: Both functions parse a JSON body and then apply an identical five-step narrowing chain: `parsed && typeof parsed === "object" && "detail" in parsed && Array.isArray((parsed as Record<string, unknown>)["detail"])`. Each step re-casts to `Record<string, unknown>` to access the next property. The same pattern is duplicated verbatim in both functions, and each uses two separate `(parsed as Record<string, unknown>)["detail"]` casts plus the same `(entry as Record<string, unknown>)["loc"]` chain.
- **Suggestion**: Extract a shared helper `parseFastApiValidationLocs(body: string): string[][] | null` that returns the array of `loc` arrays from a FastAPI 422 body (or `null` if not parseable), eliminating both the duplication and the four `as Record<string, unknown>` casts.
- **Evidence**:

```typescript
// voice-service.utils.ts:228-248 (isKeytermsProviderError)
const parsed: unknown = JSON.parse(responseBody);
if (
  parsed &&
  typeof parsed === "object" &&
  "detail" in parsed &&
  Array.isArray((parsed as Record<string, unknown>)["detail"])
) {
  return ((parsed as Record<string, unknown>)["detail"] as unknown[]).some(
    (entry) =>
      entry !== null &&
      typeof entry === "object" &&
      "loc" in (entry as Record<string, unknown>) &&
      Array.isArray((entry as Record<string, unknown>)["loc"]) &&
      ((entry as Record<string, unknown>)["loc"] as unknown[]).includes("keyterms"),
  );
}

// voice-service.utils.ts:272-294 (isAudioFormatProviderError) — identical structure, "file" instead of "keyterms"
```

---

### Finding 7: `source` string cast to enum union in `ensureSeeded`

- **File**: `apps/api/src/application/voice/services/voice-vocabulary.service.ts:572-575, 582-585`
- **Category**: type-cast
- **Impact**: medium
- **Description**: Inside `ensureSeeded`, the `terms` map stores values as `{ canonical: string; source: string }` (the `source` value is narrowed to `string` because the `Map` type is `Map<string, { canonical: string; source: string }>`). When building the `values` array for the database insert, the code casts `source as "seed_profile" | "seed_entity" | "seed_integration" | "seed_content" | "seed_knowledge"` to satisfy Drizzle's column type. This cast silently suppresses any typo introduced when seeding into the map (e.g. `"seed_profil"`). The cast appears twice (for the `< MIN_SEED_THRESHOLD` early-return path and the main path).
- **Suggestion**: Declare a `SeedSource` literal union type and store it in the `terms` map: `const terms = new Map<string, { canonical: string; source: SeedSource }>()`. Then every assignment to `terms.set(...)` is checked at compile time and the downstream casts become unnecessary.
- **Evidence**:

```typescript
// voice-vocabulary.service.ts:463 — Map with loose string source
const terms = new Map<string, { canonical: string; source: string }>();

// voice-vocabulary.service.ts:572-575 — cast required because source is string
source: source as "seed_profile" | "seed_entity" | "seed_integration" | "seed_content" | "seed_knowledge",

// Same cast at line 582-585 for the main insert path
```
