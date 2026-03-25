# Trope Audit: apps/app/src/shared/context

Files audited:

- `PaywallContext.tsx`
- `SubscriptionStateContext.tsx`
- `SyncLimitContext.tsx`

## Findings

### PaywallContext.tsx

**"This feature is unlocked on paid plans."** (line 144, `showUpgrade` call for `TRIAL_FEATURE_LOCKED`)
Category: Unlock framing -- a common SaaS upsell trope that implies the user is being held back by a gate.
The word "unlocked" frames paid tiers as removing a restriction rather than providing a service. This is minor in the context of a billing modal but worth noting. Alternative: "This feature is available on paid plans."

**"You've reached your hourly token limit. Upgrade for unlimited usage."** (line 165)
Category: "Unlimited" promise.
"Unlimited usage" is a trope and potentially inaccurate -- paid plans likely have limits too, even if higher. This is a copy accuracy issue. Prefer specific language: "Upgrade to increase your limit" or "Upgrade for a higher limit."

**"You've reached your daily token limit. Upgrade for unlimited usage."** (line 169)
Same issue as above. Duplicate of the hourly variant.

**"You've reached your trial cost limit. Upgrade to continue using AI features."** (line 174)
Category: "AI features" as a vague category label. Minor. The phrase "AI features" is acceptable product terminology but is slightly vague. Not a severe trope.

**"Your trial has ended. Upgrade to continue using SlopWeaver."** (line 149, `TRIAL_EXPIRED` handler)
No issue. Direct.

**"Upgrade to connect more than 3 integrations during your trial."** (line 153, `TRIAL_INTEGRATION_LIMIT` handler)
No issue. Specific and accurate.

**"Billing is frozen for this account. Please contact support."** (line 157, `BILLING_FROZEN` handler)
No issue. Direct.

**"Upgrade anytime. Cancel anytime."** (line 225, `PlanSelectorModal` subtitle fallback)
Category: Standard SaaS reassurance cliche. "Cancel anytime" is ubiquitous upsell copy that sounds hollow. It is the default fallback when no `upgradeReason` is supplied, which means it appears in all generic upgrade prompts. Recommend replacing with a reason-specific message or removing the subtitle entirely when no reason is provided.

**"Choose a plan"** modal title (line 224)
No issue. Direct.

**"perform this action"** fallback for `actionDescription` (line 134)
Category: Vague fallback. If shown to users it would be meaningless. Should only appear if the caller forgets to pass a description, which is a developer error. Consider a more defensive default or a console warning.

### SubscriptionStateContext.tsx

No user-visible copy in this file. Pure state derivation logic. No findings.

### SyncLimitContext.tsx

No user-visible copy in this file. Pure state management and API integration. No findings.

## Summary

| File                         | Violations                                                                         |
| ---------------------------- | ---------------------------------------------------------------------------------- |
| PaywallContext.tsx           | 4 findings (2 "unlimited" promises, 1 "unlock" framing, 1 "Cancel anytime" cliche) |
| SubscriptionStateContext.tsx | 0                                                                                  |
| SyncLimitContext.tsx         | 0                                                                                  |

The most actionable findings are the two "Upgrade for unlimited usage" strings in `PaywallContext.tsx` -- they make a promise that is likely inaccurate (paid plans have limits too) and use a classic SaaS trope. These appear in user-facing upgrade prompts triggered by rate limits, so they have real user-visible impact.
