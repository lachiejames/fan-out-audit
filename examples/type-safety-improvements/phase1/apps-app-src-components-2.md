# Audit: apps/app/src/components — Slice 2 (ai-chat-widget area)

**Files inspected**: 8
**Findings**: 6

## Summary

The ai-chat-widget cluster has two high-impact issues: a `ReasoningState` type defined independently in two separate files (an exact duplicate), and unsafe ref casts that work around a loose prop type. Secondary issues include `Record<string, V>` usage with closed key sets and an overly broad update function signature.

## Findings

### Finding 1: Duplicate `ReasoningState` type across two files

- **File**: `apps/app/src/components/ai-chat-widget.utils.ts:229-232` and `apps/app/src/components/ai-chat-widget/model-settings-context.tsx:5-8`
- **Category**: Duplicate type definitions
- **Impact**: high
- **Description**: `ReasoningState = { active: boolean; isStreaming: boolean }` is defined identically in both files. Any future change to the shape must be made in two places.
- **Suggestion**: Extract `ReasoningState` to a shared location (e.g. `apps/app/src/components/ai-chat-widget/types.ts`) and import it in both consumers.
- **Evidence**: Both files contain `type ReasoningState = { active: boolean; isStreaming: boolean }` verbatim.

### Finding 2: Unsafe ref cast in `ai-chat-widget.messages.tsx`

- **File**: `apps/app/src/components/ai-chat-widget.messages.tsx:119` and `:131`
- **Category**: Type casts (`as SomeType`) that could be avoided
- **Impact**: high
- **Description**: The prop `scrollRef` is typed as `React.RefObject<HTMLElement | null>`, and is then immediately cast to `React.RefObject<HTMLDivElement | null>` before assigning `.current`. This cast silences a type error that should instead be fixed at the prop boundary.
- **Suggestion**: Narrow the prop type to `React.RefObject<HTMLDivElement | null>` (and `contentRef` to `React.RefObject<HTMLDivElement | null>`) at the component interface level, removing the need for casts entirely.
- **Evidence**: `(scrollRef as React.RefObject<HTMLDivElement | null>).current = el` at line 119; same pattern for `contentRef` at line 131.

### Finding 3: `Record<string, string[]>` for message-keyed attachment map

- **File**: `apps/app/src/components/ai-chat-widget.messages.tsx:97`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: low
- **Description**: `hiddenAttachmentKeysByMessage: Record<string, string[]>` uses plain `string` for message IDs. While there is no existing branded `MessageId` type, adding a comment or a nominal alias would make the intent explicit and prevent mixing with other string keys.
- **Suggestion**: At minimum add a type alias `type MessageId = string` and use `Record<MessageId, string[]>`. If message IDs become branded in contracts, update accordingly.
- **Evidence**: `const [hiddenAttachmentKeysByMessage, setHiddenAttachmentKeysByMessage] = useState<Record<string, string[]>>({})` at line 97.

### Finding 4: Overly broad `updateSettings` signature in voice hook

- **File**: `apps/app/src/components/ai-chat-widget/use-ai-chat-widget-voice.ts:54`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: medium
- **Description**: `updateSettings: (updates: Record<string, unknown>) => Promise<void>` accepts any string key. In practice, callers only pass `{ voicePrivacyConsentAt: Date }` or `{ voicePrivacyConsentGiven: boolean }`.
- **Suggestion**: Replace with a typed partial: `updateSettings: (updates: Partial<Pick<UserSettings, 'voicePrivacyConsentAt' | 'voicePrivacyConsentGiven'>>) => Promise<void>`, adjusting the type name to match whatever the settings interface is in contracts or the data layer.
- **Evidence**: `updateSettings: (updates: Record<string, unknown>) => Promise<void>` at line 54.

### Finding 5: `Record<string, React.ReactNode>` icon map with closed key set

- **File**: `apps/app/src/components/ai-suggestion-chips.tsx:26`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: low
- **Description**: `const iconMap: Record<string, React.ReactNode>` maps icon name strings to nodes. The set of valid keys is fixed at the call site. Using `string` allows silent misses (typos return `undefined` without a compile error).
- **Suggestion**: Define a union type `type SuggestionIconName = 'email' | 'calendar' | ...` (matching the actual keys used) and type the map as `Partial<Record<SuggestionIconName, React.ReactNode>>`.
- **Evidence**: `const iconMap: Record<string, React.ReactNode> = { ... }` at line 26.

### Finding 6: Loose `Record<string, unknown>` update param propagated from model-settings context

- **File**: `apps/app/src/components/ai-chat-widget/model-settings-context.tsx`
- **Category**: `any` or unsafe `unknown` in production code
- **Impact**: medium
- **Description**: The context exposes an `updateSettings` function accepting `Record<string, unknown>`, meaning callers get no compile-time feedback when passing wrong keys or value types. This pattern cascades from Finding 4 above.
- **Suggestion**: Align the context's update function signature with the narrowed type from Finding 4. The context value type should reflect the real shape of updatable settings.
- **Evidence**: Context `updateSettings` type mirrors the hook signature at line 54 of the voice hook, propagated through the context provider.
