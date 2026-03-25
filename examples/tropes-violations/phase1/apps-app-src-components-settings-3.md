# AI Writing Tropes Audit - Settings Components Slice 97

**Files audited** (8):

- `apps/app/src/components/settings/settings-account.tsx`
- `apps/app/src/components/settings/settings-billing-content.tsx`
- `apps/app/src/components/settings/settings-billing-empty.tsx`
- `apps/app/src/components/settings/settings-billing-error.tsx`
- `apps/app/src/components/settings/settings-billing-skeleton.tsx`
- `apps/app/src/components/settings/settings-billing.tsx`
- `apps/app/src/components/settings/settings-diagnostics.tsx`
- `apps/app/src/components/settings/settings-integrations.tsx`

---

## Findings

### 1. `settings-account.tsx` line 67

**Violation: Vague filler description**

```
Manage your subscription, security, and data
```

This is a boilerplate subtitle that lists categories without saying anything. Every settings page in every SaaS app uses the "Manage your X, Y, and Z" formula. It contributes no information the section headings don't already provide.

**Fix:** Remove it entirely, or replace with something specific to what users actually do here, e.g. "Export your data, delete your account, or log out."

---

### 2. `settings-billing.tsx` line 16

**Violation: Vague filler description**

```
Manage your subscription and usage
```

Same pattern as above. "Manage your X" is the default AI-generated subtitle for any settings section. It says nothing a user couldn't infer from the heading "Billing & Usage".

**Fix:** Remove the subtitle, or make it concrete, e.g. "Your current plan, action balance, and payment method."

---

### 3. `settings-billing-empty.tsx` line 11

**Violation: Padded onboarding copy**

```
Choose a plan to get started with SlopWeaver.
```

"Get started with" is a filler phrase. The context is already clear: there is no subscription. The CTA button says "Choose a Plan". The sentence adds nothing.

**Fix:** "Pick a plan to enable AI actions." or just remove the supporting sentence and let the button speak.

---

### 4. `settings-integrations.tsx` lines 157-159 (loading state subtitle)

**Violation: Vague filler description**

```
Connect your platforms to import messages and reply in your style - nothing sends without you.
```

"In your style" is a recurring AI-product trope. "Nothing sends without you" is the standard AI-safety reassurance that every AI product appends. Both phrases are expected boilerplate. The loading state is the wrong moment for a value-prop pitch anyway.

**Fix:** For the loading state, show nothing or a neutral label. If this copy appears in the main view as well (it doesn't appear elsewhere in this file), replace "in your style" with something concrete and drop the safety reassurance from the subtitle - it belongs in onboarding, not a loading spinner.

---

## Summary

| File                         | Line    | Trope                                                                |
| ---------------------------- | ------- | -------------------------------------------------------------------- |
| `settings-account.tsx`       | 67      | "Manage your X, Y, and Z" filler subtitle                            |
| `settings-billing.tsx`       | 16      | "Manage your X and Y" filler subtitle                                |
| `settings-billing-empty.tsx` | 11      | "Get started with" filler phrase                                     |
| `settings-integrations.tsx`  | 157-159 | "in your style" + "nothing sends without you" AI-product boilerplate |

`settings-billing-content.tsx`, `settings-billing-error.tsx`, `settings-billing-skeleton.tsx`, and `settings-diagnostics.tsx` contain no AI writing tropes. Their copy is functional and specific.
