# Tropes Audit: packages/ui/src/molecules/KnowledgeSparkles

Files reviewed:

- `packages/ui/src/molecules/KnowledgeSparkles/KnowledgeSparklesPanel.tsx`
- `packages/ui/src/molecules/KnowledgeSparkles/KnowledgeFactItem.tsx`
- `packages/ui/src/molecules/KnowledgeSparkles/KnowledgeSparklesPanel.stories.tsx`

---

## KnowledgeSparklesPanel.tsx

**Header title (line 133):**

> `<h3 className="font-semibold">Insights</h3>`

The word "Insights" is used as the panel title visible to users. This is a common AI-startup noun — vague, abstract, implying intelligence without specifying what is actually shown. The panel contains extracted facts about the user (email address, AWS usage, billing alerts, communication patterns). "Insights" is meaningfully vaguer than "What we've learned" or "Facts we've found" which would be more concrete.

Verdict: **Violation.** "Insights" as a panel heading is abstract AI-startup language. A more specific label tied to what the panel actually shows would be clearer.

**Total count text (lines 136-142):**

```tsx
{
  totalCount > 0 ? (
    `${totalCount} facts discovered`
  ) : isExtracting ? (
    <Skeleton width={100} />
  ) : (
    "Learning from your data..."
  );
}
```

**"`${totalCount} facts discovered`"** — The word "discovered" frames routine data extraction as a magical revelation. Factual alternative: `"${totalCount} facts found"` or `"${totalCount} facts extracted"`.

**`"Learning from your data..."`** — Classic AI-startup vague process copy. This string appears when there are no facts yet and extraction is not active. It implies continuous learning but gives no actionable information. Better: `"Connect an integration to start"` or `"No facts yet"`.

Verdict: **Two violations.**

**"Reviewing:" source message label (line 149):**

> `Reviewing: <span className="font-medium text-foreground">{sourceMessage}</span>`

The word "Reviewing" is reasonably concrete — it names the message being processed. No trope language. No findings.

**Empty state copy (lines 172-175):**

```tsx
<p className="font-medium text-sm">Processing messages</p>
<p className="mt-1 text-xs">Insights appear as more data comes in</p>
```

**`"Processing messages"`** — Functional. No findings.

**`"Insights appear as more data comes in"`** — Uses "Insights" again (same vagueness issue as panel header). Also "as more data comes in" is passive and vague. Better: `"Facts appear as messages are processed"` or `"No facts yet. Sync a platform to start."`.

Verdict: **Violation** (repeated use of "Insights" as noun).

**Component description comment (lines 4-9):**

> "KnowledgeSparklesPanel - Panel for displaying extracted insights"
> "Shows AI-extracted facts with animations as they're discovered."
> "The 'wow moment' of onboarding - making the invisible visible."

Internal developer documentation. The phrase `"making the invisible visible"` is marketing copy in a code comment. Not user-facing, but a signal that the component's design intent is wrapped in AI-startup framing.

---

## KnowledgeFactItem.tsx

**KnowledgeFactItem JSDoc example (lines 120-124):**

```tsx
<KnowledgeFactItem fact={{ category: "skill", content: "User uses AWS", confidence: 0.92 }} isNew={true} />
```

Example data. No trope language. No findings.

**Fact content rendering (line 140):**

> `<span>"{fact.content}"</span>`

Fact content is displayed in quotation marks. The content itself comes from the API, not from this component. No findings in the component itself.

---

## KnowledgeSparklesPanel.stories.tsx

**SkeletonLoading story `sourceMessage` (line 97):**

> `sourceMessage: "Analyzing your messages..."`

The phrase "Analyzing your messages..." is a loading state string used in stories. If this is representative of what real source messages look like, "Analyzing" is process-language that implies intelligence. However, in production the `sourceMessage` is the actual subject line (e.g., "AWS Billing Alert"), not a generic phrase — this appears to be story-specific data only. No production violation.

**AnimatedDemo story description (line 194):**

> `"Watch as facts animate in with sparkle effects. Each new fact glows briefly."`

Internal story documentation. No findings.

**Mock fact content (lines 25-72):**

> `"User uses AWS for cloud infrastructure"` / `"User has a billing notification from AWS"` / `"User's email is lachie_james@hotmail.com.au"` / `"User receives automated billing alerts"` / `"User has active AWS account since March"` / `"User communicates with AWS support team"`

Mock data for stories. These facts start with "User ..." which is an odd third-person construction if these are meant to be shown to the user about themselves. In production the fact content comes from the API. In stories, the awkward third-person framing is notable but not a production concern. Flag for API content review, not UI component review.

---

## Summary

**Violations found:**

1. **`KnowledgeSparklesPanel.tsx` line 133** — Panel heading `"Insights"` is vague AI-startup language. Replace with a concrete label (e.g., "What we've learned" or "Facts").

2. **`KnowledgeSparklesPanel.tsx` lines 136-138** — `"${totalCount} facts discovered"` uses "discovered" to frame routine extraction as a magical revelation. Replace with `"${totalCount} facts found"`.

3. **`KnowledgeSparklesPanel.tsx` lines 140-142** — `"Learning from your data..."` is classic AI-startup vague process copy with no actionable information. Replace with something specific, e.g., `"Connect a platform to start"` or `"Sync more messages to see facts"`.

4. **`KnowledgeSparklesPanel.tsx` line 175** — `"Insights appear as more data comes in"` repeats the vague "Insights" noun and "as more data comes in" gives no guidance. Replace with concrete alternative.
