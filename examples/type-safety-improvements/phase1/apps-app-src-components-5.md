# Audit: apps/app/src/components — Slice 5 (billing components part 1)

**Files inspected**: 8
**Findings**: 4

## Summary

The billing components in this slice have two main issues: a cast-after-string-parse pattern on a `Select` component's `onValueChange` that should use schema validation instead, and two `Record<string, string>` maps with closed key sets.

## Findings

### Finding 1: `as ActionPackId` cast on unvalidated Select value

- **File**: `apps/app/src/components/billing/auto-refill-settings.tsx:94`
- **Category**: Type casts (`as SomeType`) that could be avoided
- **Impact**: high
- **Description**: `const newPackId = value as ActionPackId` casts the raw string from `Select`'s `onValueChange` callback directly to `ActionPackId` without runtime validation. If the option values in the JSX and the valid `ActionPackId` values diverge, the cast silently passes an invalid ID to the mutation.
- **Suggestion**: Use the Zod schema from contracts to parse and validate: `const result = actionPackIdSchema.safeParse(value)`. Only proceed if `result.success` is true, using `result.data` (which is typed as `ActionPackId`). This eliminates the cast and adds a runtime safety net.
- **Evidence**: `const newPackId = value as ActionPackId` at line 94 inside `onValueChange`.

### Finding 2: `MODEL_COLORS` using `string` keys for a closed AI model set

- **File**: `apps/app/src/components/billing/ai-transparency-card.tsx:22`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: low
- **Description**: `const MODEL_COLORS: Record<string, string>` maps Anthropic model names (`haiku`, `sonnet`, `opus`) to color strings. The key set is closed. Using `string` allows silent misses when a new model tier is added.
- **Suggestion**: Define `type ClaudeModelTier = 'haiku' | 'sonnet' | 'opus'` (or import an equivalent from contracts if one exists) and type as `Record<ClaudeModelTier, string>`. TypeScript will then require all tiers to be present.
- **Evidence**: `const MODEL_COLORS: Record<string, string> = { haiku: ..., sonnet: ..., opus: ... }` at line 22.

### Finding 3: Billing component props using `string` for tier instead of `SubscriptionTier`

- **File**: `apps/app/src/components/billing/` (multiple files in slice 5)
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: medium
- **Description**: Several billing components accept tier-related props typed as `string` rather than `SubscriptionTier` from `@slopweaver/contracts`. This means a tier value passed as a prop is not validated at the call site.
- **Suggestion**: Import `SubscriptionTier` from `@slopweaver/contracts` and use it as the prop type for any tier-related prop. This surfaces type errors at call sites rather than at runtime.
- **Evidence**: Tier props typed as `string` in billing component interfaces across the slice.

### Finding 4: `BillingProvider` string literals used without importing the contract type

- **File**: `apps/app/src/components/billing/` (multiple files in slice 5)
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: medium
- **Description**: Billing provider comparisons (e.g. `provider === 'lemon_squeezy'`) use string literals rather than the `BillingProvider` union type from `@slopweaver/contracts`. A provider value rename in contracts will not surface as a compile error.
- **Suggestion**: Import `BillingProvider` from `@slopweaver/contracts`. Use it for prop types and comparisons. If exhaustiveness checking is needed for a switch statement, add a `default: assertNever(provider)` arm.
- **Evidence**: String literal provider comparisons in billing component conditional rendering logic.
