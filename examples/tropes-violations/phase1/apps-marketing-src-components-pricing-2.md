# Trope Audit: apps/marketing/src/components/pricing/usage-calculator.tsx (Slice 157)

File audited:

- usage-calculator.tsx

---

## Findings

### usage-calculator.tsx

**Line 56** - `reason: "Your usage fits comfortably in the Starter tier"`
Weak recommendation copy. "Fits comfortably" is vague filler. More useful would be a specific statement referencing the buffer percentage already calculated, for example: "You have X% headroom in Starter based on your inputs."

**Line 65** - `reason: "Pro gives you room to grow"`
"Room to grow" is a generic marketing phrase. The calculator already computes a buffer percentage; using it here would be stronger ("Pro gives you X% buffer at your current usage"). The phrase as written could appear on any SaaS pricing page for any tier.

**Line 73** - `reason: "Max tier for heavy users"`
Thin. "Heavy users" is undefined. The calculator knows exactly how many actions the user needs; the reason should reference that ("Your estimated X actions/month fits in Max"). This is the same problem as the Starter reason: the data exists but the copy ignores it.

**Line 109** - `<p className="text-lg text-muted-foreground">Approximate your plan based on daily actions</p>`
"Approximate your plan" is slightly awkward and could be misread as "approximate the plan" (estimate it inaccurately) rather than "find the right plan." Minor but worth clarifying: "Find the right plan based on your daily usage" or similar.

No banned phrases detected. No structural AI tells. No overclaiming. The calculator itself is functionally sound: it uses real contract values, computes actual action costs, and shows a buffer percentage.

---

## Summary

| Issue                                          | Location | Severity |
| ---------------------------------------------- | -------- | -------- |
| Vague recommendation reason "fits comfortably" | Line 56  | Low      |
| Generic "room to grow" phrase                  | Line 65  | Low      |
| Thin "heavy users" descriptor                  | Line 73  | Low      |
| Slightly awkward subheadline                   | Line 109 | Very low |

No hard violations of banned phrases or structural AI tells. All issues are copy quality improvements rather than rule violations. The calculator's use of real contract values (`ACTION_COSTS`, `SUBSCRIPTION_TIER_DISPLAY`) is correct and the recommendation logic is sound.

**Priority fix:** The three `reason` strings (lines 56, 65, 73) should use the already-computed buffer/actions data rather than generic phrases. This is a missed opportunity given the data is available in the same `useMemo` block.
