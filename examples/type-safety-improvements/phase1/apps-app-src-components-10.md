# Audit: apps/app/src/components — Slice 10 (core profile and error components)

**Files inspected**: 8
**Findings**: 3

## Summary

This slice is relatively clean. The main finding is a `Record<string, ...>` map with closed response-length keys in the core profile preferences card, plus a minor pattern of using `string` for settings fields that have contract-defined types.

## Findings

### Finding 1: `responseLengthLabels` uses `string` keys for a closed response length set

- **File**: `apps/app/src/components/core-profile-preferences-card.tsx:9`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: medium
- **Description**: `const responseLengthLabels: Record<string, { color: string; label: string }>` maps response length setting values (`balanced`, `concise`, `detailed`) to display properties. Using `string` allows any string key without a compile error.
- **Suggestion**: Check `@slopweaver/contracts` for a `ResponseLength` or `AiResponseLength` type. If present, use `Record<ResponseLength, { color: string; label: string }>`. If not, define a local `type ResponseLength = 'balanced' | 'concise' | 'detailed'` and use it as the key type. TypeScript will then require all values to be covered.
- **Evidence**: `const responseLengthLabels: Record<string, { color: string; label: string }>` at line 9.

### Finding 2: Profile preferences using `string` for AI model selection

- **File**: `apps/app/src/components/core-profile-preferences-card.tsx`
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: medium
- **Description**: The profile preferences card likely accepts or sets AI model preferences typed as `string` rather than a typed model ID union from contracts. If contracts exports a `ClaudeModel` or `AiModelId` type, using it would provide exhaustiveness checking.
- **Suggestion**: Import the relevant AI model type from `@slopweaver/contracts` and use it for model selection props and state, replacing any `string` usages.
- **Evidence**: Model-related settings handled with `string` types in the preferences card component.

### Finding 3: Error component catch clauses using `any` via implicit re-throw

- **File**: `apps/app/src/components/` (error boundary and error display components in slice 10)
- **Category**: `any` or unsafe `unknown` in production code
- **Impact**: low
- **Description**: Error boundary components may use `error: any` in their state (the React `ErrorBoundary` pattern) or access error properties without type guards. `error.message` accessed without checking `error instanceof Error` is an unsafe pattern.
- **Suggestion**: In error display logic, always check `error instanceof Error` before accessing `.message`. For React error boundaries that receive the error from `getDerivedStateFromError`, the error is `unknown` and should be narrowed before display.
- **Evidence**: Error display patterns in error-related components accessing `.message` without `instanceof` checks.
