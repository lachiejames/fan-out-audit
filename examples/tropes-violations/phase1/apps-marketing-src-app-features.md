# Trope Audit: apps/marketing/src/app/features/

**Slice:** 143
**Files audited:** 2

- `features/layout.tsx`
- `features/page.tsx`

---

## Summary

Several violations found in `features/page.tsx`. The layout file contains only metadata and no prose violations.

---

## Violations

### 1. "Ready to get started?" CTA heading

**File:** `features/page.tsx`
**Location:** CTA section, line ~320
**Text:** "Ready to get started?"

**Problem:** This is the single most overused CTA heading in SaaS marketing. Generic, passive, adds no information. Every visitor already decided whether they want to start; the heading does nothing to help them.

**Suggested fix:** Replace with something that connects to the page's thesis, e.g., "Connect your tools. Track the improvement." or "See your AI's report card after the first week."

---

### 2. "Start free, connect your tools, and watch your metrics improve over the first few weeks."

**File:** `features/page.tsx`
**Location:** CTA section subtext, line ~323
**Text:** "Start free, connect your tools, and watch your metrics improve over the first few weeks."

**Problem:** "Watch your metrics improve" is passive and vague in a CTA context. The features page already explains metrics concretely; the CTA subtext retreats to generic language. "Over the first few weeks" is an unanchored hedge.

**Suggested fix:** Ground it in the specific promise: "Connect your tools. After one week, open analytics and see whether the AI is getting better at drafting in your voice."

---

### 3. "Get Started Free" button label (repeated twice)

**File:** `features/page.tsx`
**Location:** CTA section buttons
**Text:** "Get Started Free"

**Problem:** Stock SaaS button copy. "Get Started" is meaningless action language. Not egregious on its own but combined with "Ready to get started?" heading, the entire CTA section reads as boilerplate.

**Note:** This is a shared UI pattern and may appear in many places. Flag for systematic review rather than one-off fix.

---

### 4. Metadata description passive construction

**File:** `features/layout.tsx`
**Location:** Metadata `description` field
**Text:** "Provable AI learning, unified inbox, smart triage, meeting prep, and voice input. See every feature SlopWeaver offers to manage your work tools."

**Problem:** "See every feature SlopWeaver offers to manage your work tools" is generic feature-index language. "Smart triage" uses a vague adjective ("smart") rather than describing what the triage actually does.

**Suggested fix:** Lead with the differentiator: "Track your AI's acceptance rate, confidence scores, and time saved. All 17 integrations, one operating view."

---

### 5. "Behavioral fingerprint" phrasing

**File:** `features/page.tsx`
**Location:** Cross-Platform Intelligence section intro, line ~250
**Text:** "SlopWeaver builds a behavioral fingerprint from all of them..."

**Problem:** "Behavioral fingerprint" is slightly creepy-sounding AI jargon. The word "fingerprint" implies permanent tracking rather than a helpful work model. The intent is clear but the metaphor may create unease in privacy-conscious users.

**Suggested fix:** "SlopWeaver builds a work profile from all of them..." or the already-used term "behavioral profile" which appears elsewhere on the same page.

---

### Notes on what is working

The features page is substantially better than typical AI product marketing. Specific callouts worth preserving:

- "Your AI has a report card." (Pillar 1) -- concrete, memorable, differentiated
- "Writing patterns lose 5% confidence daily without reinforcement" -- specific mechanism, not vague
- "Morning briefs ban greetings, filler, and invented urgency" -- direct, credible
- "VIP contacts get extended thinking. Newsletter triage gets the fast model." -- concrete model routing description
- The metric mockup (82% acceptance rate, 14.2h saved, 47 patterns) is effective specificity

The main issue is the CTA section undoes the page's credibility by reverting to stock SaaS language after strong specific content throughout.
