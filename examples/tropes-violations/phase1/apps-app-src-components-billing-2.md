# Tropes Audit: apps/app/src/components/billing (batch 2)

Files audited:

- `cancel-subscription-modal.tsx`
- `downgrade-modal.tsx`
- `invoice-history.tsx`
- `invoice-status-badge.tsx`
- `payment-failed-banner-wrapper.tsx`
- `payment-failed-banner.tsx`
- `payment-failed-modal.tsx`
- `payment-method-card.tsx`

---

## Findings

### `cancel-subscription-modal.tsx`, line 128

**Trope: false vulnerability / sentimental heading**

Text: `"Sorry to see you go"`

This is a stock cancellation-flow cliche. The phrase performs emotion on behalf of a software product, which is a form of false vulnerability. It is used verbatim (or near-verbatim) across Spotify, Notion, Linear, and dozens of other SaaS products. A factual heading like `"Cancel subscription"` or `"Confirm cancellation"` is cleaner and does not reach for sentiment the product has not earned.

Severity: minor.

---

### `cancel-subscription-modal.tsx`, line 147

**Trope: "you'll lose access to" fear-based framing**

Text: `"You'll lose access to:"`

Loss-framing lists are a dark-pattern staple, and the phrase itself is boilerplate across virtually every SaaS cancellation flow. The intent is legitimate (show impact), but the wording is the industry's most overused formulation. A neutral alternative: `"Features included in your current plan:"` or simply removing the heading and letting the feature list stand on its own.

Severity: minor.

---

### `downgrade-modal.tsx`, line 130

**Trope: "you'll lose access to" fear-based framing (duplicate)**

Text: `"You'll lose access to:"`

Same as above in `cancel-subscription-modal.tsx`. Both modals use the identical loss-framing phrase.

Severity: minor.

---

### `invoice-history.tsx`, line 52

**Trope: redundant subtitle / filler description**

Text: `"Download past invoices and receipts"`

The section heading is already `"Invoices & Receipts"`. The subtitle adds no new information, it just restates the heading as an imperative. This is a mild case of one-point dilution. Dropping the subtitle entirely tightens the UI.

Severity: minor.

---

### `payment-failed-banner.tsx`, line 31

**Trope: compound imperative that over-explains**

Text: `"Your payment failed. Update your payment method to continue."`

The first sentence is functional. The second sentence ("...to continue") adds a consequence clause that is self-evident. If payment failed, of course updating is required to continue. The consequence clause is the kind of reasoning-out-loud that AI-generated microcopy typically adds. A tighter alternative: `"Payment failed. Update your payment method."` or just `"Update your payment method to restore access."` (single sentence, action-first).

Severity: minor.

---

### `payment-failed-modal.tsx`, line 74-75

**Trope: passive/indirect framing for a direct error**

Text: `"Payment couldn't be processed. Please update the payment method to continue using SlopWeaver."`

"Couldn't be processed" is passive and vague compared to "Payment failed." The phrase "to continue using SlopWeaver" is a consequence clause again (same pattern as the banner). "Please" in UI error text is unnecessary hedging. A direct rewrite: `"Payment failed. Update your payment method to restore access."`

Severity: minor.

---

## No findings

- `invoice-status-badge.tsx` - No prose, just `status.toLowerCase()` label rendering.
- `payment-failed-banner-wrapper.tsx` - No user-facing text, pure conditional rendering logic.
- `payment-method-card.tsx` - Text labels ("Subscription Provider", "Manage Subscription", "Restore Purchases", "Choose Plan", "Managed by your device's app store") are functional labels with no tropes.
