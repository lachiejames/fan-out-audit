# Tropes Audit: apps/app/src/components/billing

Files audited:

- `plan-selector-modal.tsx`
- `soft-paywall.tsx`
- `subscription-cancelled-banner-wrapper.tsx`
- `subscription-cancelled-banner.tsx`
- `subscription-expired-modal-wrapper.tsx`
- `subscription-expired-modal.tsx`
- `sync-limit-paywall-footer.tsx`
- `sync-limit-paywall.tsx`

---

## Violations

### 1. "Choose Your Plan" — generic SaaS header copy

**File**: `plan-selector-modal.tsx`, line 185

**Offending text**:

```
{title ?? "Choose Your Plan"}
```

**Why it's a violation**: "Choose Your Plan" is the default title on the modal when no override is provided. It's the most common headline on every SaaS pricing modal ever built. The content playbook bans generic in-app copy and requires concise, specific language that reflects what's actually happening. This modal is surfaced in multiple contexts (trial upgrade, post-expiry, soft upsell), and the default should reflect the product's voice.

**Suggested fix**: Something concrete like "Pick your tier" or, in trial contexts where the modal is shown proactively, a more pointed title tied to the specific prompt that opened it.

---

### 2. "Upgrade anytime. Cancel anytime." — friction-reduction boilerplate

**File**: `plan-selector-modal.tsx`, line 188

**Offending text**:

```
{subtitle ?? "Upgrade anytime. Cancel anytime."}
```

**Why it's a violation**: This is standard SaaS paywall reassurance copy found on nearly every pricing page. It says nothing specific to SlopWeaver's product or value. The content playbook requires in-app copy to be concise and specific, not generic reassurance filler. The "anytime" framing is especially empty.

**Suggested fix**: Use the default subtitle slot to communicate something real about the plan the user is about to pick. If the modal was triggered by a specific action, the subtitle should reference that context. A neutral fallback like "Billed monthly. Cancel from settings." is more honest and less generic than the current copy.

---

### 3. "Recommended" badge on the Pro tier

**File**: `plan-selector-modal.tsx`, line 262

**Offending text**:

```
Recommended
```

**Why it's a violation**: "Recommended" on the middle tier of a three-tier pricing modal is a ubiquitous SaaS dark pattern. It implies an objective recommendation while actually just marking the highest-margin tier. The content playbook bans copy that implies claims without proof. There is no basis stated for why Pro is "recommended" over Starter or Max for any given user.

**Suggested fix**: Either remove the badge, or replace it with something data-driven if available (e.g., "Most chosen" if that's true and measurable). If neither is available, leave the tier card without a badge rather than using a meaningless label.

---

### 4. "POPULAR" badge on the Pro tier in the expired modal

**File**: `subscription-expired-modal.tsx`, line 81

**Offending text**:

```
POPULAR
```

**Why it's a violation**: Same issue as the "Recommended" badge above, compounded by the use of ALL CAPS, which is explicitly banned in the content playbook ("ALL CAPS for emphasis: banned"). The expired modal is a high-stakes context where the user has lost access to the app. Leading with marketing badge copy in that moment is off-brand.

**Suggested fix**: Remove the badge. The expired modal should help the user pick a plan to regain access, not run SaaS pricing-page tactics.

---

### 5. "Something went wrong creating your checkout. Please try again or contact support."

**File**: `plan-selector-modal.tsx`, line 369

**Offending text**:

```
Something went wrong creating your checkout. Please try again or contact support.
```

**Why it's a violation**: "Something went wrong" is explicitly called out in the content playbook's error message guidelines as the wrong pattern. The playbook says error messages should "say what happened, what the user can do, and when it might resolve" and specifies "No 'Oops!' or 'Something went wrong'". This error is vague and provides no actionable specifics.

**Suggested fix**: Be specific. If the checkout URL generation failed, say that. Something like: "Checkout unavailable. Try again, or go to Settings > Billing to manage your subscription." If multiple failure modes exist, differentiate them.

---

### 6. "Usage running low" — unnecessary softening

**File**: `soft-paywall.tsx`, line 112

**Offending text**:

```
Usage running low
```

**Why it's a violation**: The heading is technically accurate only when the user's balance is not yet zero, but the user is trying to perform an action they don't have enough balance for. "Running low" is imprecise in that moment. More importantly, it's a softening phrase. The content playbook says in-app copy should be "concise, helpful" and show real numbers. The modal already shows the exact deficit below this heading, so the heading should be specific.

**Suggested fix**: "Not enough actions" or "Action balance too low" directly describes the situation. Alternatively, since the exact numbers are shown in the subline, the heading could simply be "Insufficient balance" or even reference the action: e.g., "Can't {action}: balance too low."

---

### 7. "Maybe later" — dismissal copy with hedging tone

**File**: `soft-paywall.tsx`, line 207; `sync-limit-paywall.tsx`, line 202

**Offending text**:

```
Maybe later
```

**Why it's a violation**: "Maybe later" is wheedling. It mimics the tone of a salesperson offering to come back. The content playbook says in-app copy should be "concise, helpful, never sycophantic." A plain "Dismiss" or "Close" is cleaner and treats the user as an adult who knows what they're doing.

**Suggested fix**: "Dismiss" or "Close."

---

### 8. "Your data will be permanently deleted after 30 days without an active subscription"

**File**: `subscription-expired-modal.tsx`, line 118

**Offending text**:

```
Your data will be permanently deleted after 30 days without an active subscription
```

**Why it's a violation**: This is a fear-based retention line placed at the bottom of a blocking paywall modal. While factually accurate (and transparency is good), pairing a data deletion threat with the subscribe CTA is a pressure pattern. The content playbook says "No humor in errors. No 'Oops!'" and specifically bans jokes in "billing, or security contexts." The deletion threat functions as a scare tactic in a billing context, which conflicts with the brand's anti-hype, matter-of-fact tone.

The modal already communicates that data is preserved for 30 days in the header ("Your data is preserved for 30 days."). Repeating the deletion threat in the footer converts the reassurance into a countdown warning.

**Suggested fix**: Remove the footer line. The header already covers the 30-day grace period positively. If the deletion policy must appear, reframe it neutrally: "Data is retained for 30 days from expiry date."

---

### 9. "Choose a Plan to Start Importing" / "Your trial has ended" — inconsistent framing

**File**: `sync-limit-paywall.tsx`, lines 102-104

**Offending text**:

```
message: "Your trial has ended. Choose a plan to import messages and unlock reply features.",
title: "Choose a Plan to Start Importing",
```

**Why it's a violation**: Two issues here. First, "unlock" is in the banned phrases list ("unlock, supercharge, turbocharge"). Second, "import messages and unlock reply features" strings together two benefits in a way that sounds like marketing copy rather than an explanation of what the user specifically needs to do right now. The content playbook says in-app copy should be "Concise, helpful, never sycophantic. Focus on what's happening and what the user can do."

**Suggested fix**: "Trial ended. Choose a plan to continue importing messages and using AI replies." Drop "unlock." Keep it factual.

---

## Summary

| #   | File                                          | Line(s)  | Category                                | Severity |
| --- | --------------------------------------------- | -------- | --------------------------------------- | -------- |
| 1   | `plan-selector-modal.tsx`                     | 185      | Generic SaaS copy                       | Medium   |
| 2   | `plan-selector-modal.tsx`                     | 188      | Generic SaaS copy                       | Medium   |
| 3   | `plan-selector-modal.tsx`                     | 262      | Unsupported claim / dark pattern        | Medium   |
| 4   | `subscription-expired-modal.tsx`              | 81       | ALL CAPS ban + unsupported claim        | High     |
| 5   | `plan-selector-modal.tsx`                     | 369      | Banned error message pattern            | High     |
| 6   | `soft-paywall.tsx`                            | 112      | Imprecise / softening copy              | Low      |
| 7   | `soft-paywall.tsx` + `sync-limit-paywall.tsx` | 207, 202 | Sycophantic dismissal copy              | Low      |
| 8   | `subscription-expired-modal.tsx`              | 118      | Pressure/fear copy in billing context   | Medium   |
| 9   | `sync-limit-paywall.tsx`                      | 102-104  | Banned word ("unlock") + marketing tone | Medium   |

Files with no violations: `subscription-cancelled-banner-wrapper.tsx`, `subscription-expired-modal-wrapper.tsx`, `sync-limit-paywall-footer.tsx`, `subscription-cancelled-banner.tsx`.
