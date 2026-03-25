# Phase 2: Cross-Cutting Patterns -- Group 7 (Billing, Command Palette, Contacts, Inbox)

Input slices: banners, billing-1 through billing-4, bootstrap, citations, command-palette, common, contacts, inbox-1, inbox-2

---

## Pattern 1: Generic SaaS boilerplate copy in billing flows (7 instances across 4 slices)

Billing modals and paywalls use stock SaaS phrases found verbatim across hundreds of products, with no SlopWeaver voice or specificity.

| File                                          | Text                               |
| --------------------------------------------- | ---------------------------------- |
| `plan-selector-modal.tsx`                     | "Choose Your Plan"                 |
| `plan-selector-modal.tsx`                     | "Upgrade anytime. Cancel anytime." |
| `plan-selector-modal.tsx`                     | "Recommended" badge                |
| `subscription-expired-modal.tsx`              | "POPULAR" badge (ALL CAPS, banned) |
| `cancel-subscription-modal.tsx`               | "Sorry to see you go"              |
| `soft-paywall.tsx` / `sync-limit-paywall.tsx` | "Maybe later" dismissal            |
| `sync-limit-paywall.tsx`                      | "Choose a Plan to Start Importing" |

**Why this is a pattern**: Every billing touchpoint uses copy that could belong to any SaaS product. "Choose Your Plan", "Sorry to see you go", "Maybe later", and tier badges like "Recommended" / "POPULAR" are industry defaults that signal zero product identity. The content playbook explicitly bans generic SaaS copy and ALL CAPS for emphasis.

**Systemic fix**: Audit all billing modals as a single copy pass. Replace generic headers with context-aware titles that reference why the modal appeared. Remove "Recommended"/"POPULAR" badges or replace with data-driven claims. Replace "Maybe later" with "Dismiss" or "Close". Drop "Sorry to see you go" for a neutral "Cancel subscription".

---

## Pattern 2: Fear-based retention and loss framing (4 instances across 3 slices)

Cancellation and expiry flows use loss-aversion language to pressure users into staying.

| File                             | Text                                                                                          |
| -------------------------------- | --------------------------------------------------------------------------------------------- |
| `cancel-subscription-modal.tsx`  | "You'll lose access to:"                                                                      |
| `downgrade-modal.tsx`            | "You'll lose access to:"                                                                      |
| `subscription-expired-modal.tsx` | "Your data will be permanently deleted after 30 days without an active subscription"          |
| `action-warning-banner.tsx`      | "Upgrade to keep working without interruptions" / "Consider upgrading to avoid interruptions" |

**Why this is a pattern**: "You'll lose access to" appears identically in two separate modals. The data deletion threat in the expired modal repeats information already stated positively in the header, converting reassurance into a countdown warning. "Avoid interruptions" is a vague stakes-inflation phrase used in upsell banners. Together these create a billing experience built on fear rather than transparency.

**Systemic fix**: Replace "You'll lose access to:" with neutral framing ("Features included in your current plan:"). Remove the deletion threat footer (the header already covers the 30-day grace period). Replace "avoid interruptions" with specific consequences ("Your action balance will run out").

---

## Pattern 3: Redundant subtitles that restate the heading (5 instances across 4 slices)

Multiple components pair a heading with a subtitle that adds no new information.

| File                             | Heading               | Subtitle                                       |
| -------------------------------- | --------------------- | ---------------------------------------------- |
| `usage-chart.tsx`                | "Usage Trends"        | "Usage over time"                              |
| `invoice-history.tsx`            | "Invoices & Receipts" | "Download past invoices and receipts"          |
| `action-dashboard.tsx`           | "Usage Breakdown"     | "Where your usage went this period"            |
| `auto-refill-settings.tsx`       | Card subheading       | Toggle description repeats subheading verbatim |
| `queue-filter-bar.tsx` (Group 8) | "Queue"               | "Proposed actions ready for your review"       |

**Why this is a pattern**: Each subtitle restates the heading in slightly different words without adding value. This is a hallmark of AI-generated copy where a subtitle is reflexively added to every section. The pattern wastes visual space and dilutes the heading's clarity.

**Systemic fix**: Remove subtitles that restate the heading. If a subtitle is needed, it should add information the heading does not contain (e.g., date range, count, filter state).

---

## Pattern 4: Vague "suggested" / "AI suggested" framing without evidence (5 instances across 3 slices)

AI-sourced content is labeled with generic "suggested" or "AI suggested" framing that asserts AI involvement without showing confidence, source, or mechanism.

| File                   | Text                                               |
| ---------------------- | -------------------------------------------------- |
| `PredictionBadge.tsx`  | "AI suggested a reply based on your writing style" |
| `PredictionPanel.tsx`  | "This response was generated from your context"    |
| `message-card.tsx`     | "Reply ready" badge (static assertion, no proof)   |
| `suggestion-chips.tsx` | "Suggested actions" section label                  |
| `suggestion-chips.tsx` | "Suggested: ${chip.reasoning}" tooltip prefix      |

**Why this is a pattern**: Every AI-generated output is labeled with the word "suggested" or a vague provenance claim ("from your context", "based on your writing style") that tells the user nothing actionable. The content playbook requires showing real numbers from the user's data, not claiming adaptation. "Suggested actions" is especially problematic when applied to static fallback chips where no AI suggestion was made.

**Systemic fix**: Replace "suggested" labels with either (a) nothing (let the UI affordance speak for itself), (b) a confidence score where available, or (c) a concrete source citation ("Based on your last 3 replies to this sender"). Remove the "Suggested actions" section label or replace with "Actions" when chips are static.

---

## Pattern 5: Theatrical AI loading/searching indicators (3 instances across 2 slices)

The command palette and chat widget display loading badges that dramatize or misrepresent what the system is doing.

| File                         | Text                                                               |
| ---------------------------- | ------------------------------------------------------------------ |
| `command-palette.tsx`        | "Interpreting your question..." (fires on regex match, no AI call) |
| `command-palette.tsx`        | "Searching live APIs..." (infrastructure jargon)                   |
| `command-palette-footer.tsx` | "Cmd+K does everything" (marketing hype in UI hint)                |

**Why this is a pattern**: "Interpreting your question" fires before any AI call is made, making the system appear to be thinking when it is not. "Searching live APIs" exposes infrastructure terminology. "Does everything" is a vague superlative. All three inject marketing or theater into a utility UI where functional accuracy matters most.

**Systemic fix**: Remove theatrical badges. The spinner is sufficient for loading states. If a live-vs-cached distinction matters, surface it on individual results (the "Live" chip already does this). Remove the "does everything" footer.

---

## Pattern 6: Passive/vague error messages that provide no actionability (4 instances across 3 slices)

Error toasts and banners describe failures without telling users what happened or what to do.

| File                        | Text                                                                                      |
| --------------------------- | ----------------------------------------------------------------------------------------- |
| `plan-selector-modal.tsx`   | "Something went wrong creating your checkout" (banned pattern)                            |
| `payment-failed-banner.tsx` | "Your payment failed. Update your payment method to continue." (self-evident consequence) |
| `payment-failed-modal.tsx`  | "Payment couldn't be processed. Please update..." (passive, hedging)                      |
| `citation-chip.tsx`         | "From:" prefix used for all content types regardless of platform                          |

**Why this is a pattern**: Error messages either use the explicitly banned "something went wrong" pattern or pad functional messages with unnecessary consequence clauses ("to continue") and hedging ("please", "couldn't be processed"). The citation chip applies a one-size-fits-all label across different content types.

**Systemic fix**: Error messages should state what failed and what the user can do, in one sentence. Remove "something went wrong" everywhere. Drop consequence clauses when they are self-evident. Make platform-aware labels where the content type differs (messages vs documents).

---

## Pattern 7: Empty-state copy that markets rather than helps (3 instances across 2 slices)

Empty states use the moment to pitch the product rather than orient the user.

| File                       | Text                                                                                   |
| -------------------------- | -------------------------------------------------------------------------------------- |
| `contacts-empty-state.tsx` | "SlopWeaver builds your contact graph as it learns who you interact with across tools" |
| `contacts-empty-state.tsx` | "messages will get priority treatment"                                                 |
| `contacts-empty-state.tsx` | "You'll be notified when duplicates are detected"                                      |

**Why this is a pattern**: Empty states explain the product's mechanism ("learns who you interact with") rather than helping the user take the next step. "Priority treatment" is vague. "Detected" is passive machine language. These empty states read as feature pitches rather than useful guidance.

**Systemic fix**: Empty states should lead with the outcome and the next action. "Connect your tools and contacts will appear as messages come in." Concrete verbs, no mechanism descriptions.

---

## Summary Table

| #   | Pattern                                    | Instance Count | Slices Affected | Severity |
| --- | ------------------------------------------ | -------------- | --------------- | -------- |
| 1   | Generic SaaS boilerplate in billing        | 7              | 4               | High     |
| 2   | Fear-based retention/loss framing          | 4              | 3               | High     |
| 3   | Redundant subtitles restating headings     | 5              | 4               | Medium   |
| 4   | Vague "suggested" framing without evidence | 5              | 3               | High     |
| 5   | Theatrical AI loading indicators           | 3              | 2               | Medium   |
| 6   | Passive/vague error messages               | 4              | 3               | Medium   |
| 7   | Marketing-first empty states               | 3              | 2               | Medium   |
