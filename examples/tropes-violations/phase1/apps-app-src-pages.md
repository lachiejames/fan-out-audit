# AI Writing Tropes Audit — apps/app/src/pages

**Files audited** (2):

- `inbox-messages-list.tsx`
- `page.tsx`

---

## inbox-messages-list.tsx

**Lines 47-48:**

```tsx
<div className="mx-auto mb-3 h-6 w-6 animate-pulse rounded-full bg-muted" />
<p className="text-muted-foreground text-sm">Loading messages...</p>
```

**Trope: "Loading messages..." as a generic loading label.**
The trailing ellipsis on a loading state is a common AI-era pattern that implies ongoing narration. "Loading messages" (no ellipsis) or a more specific label like "Fetching your inbox" is cleaner. Minor — flagged for awareness.

---

## page.tsx

**Lines 57-60:**

```tsx
<div className="h-2.5 w-2.5 animate-pulse rounded-full bg-muted" />
<p className="text-muted-foreground text-sm">Loading inbox…</p>
```

**Trope: "Loading inbox…" with an ellipsis as a loading label.**
Same pattern as `inbox-messages-list.tsx`. The ellipsis reads as AI-narrated suspense. "Loading inbox" without trailing punctuation is cleaner.

**Lines 68-71:**

```tsx
<div className="h-2.5 w-2.5 animate-pulse rounded-full bg-muted" />
<p className="text-muted-foreground text-sm">Redirecting to setup…</p>
```

**Trope: "Redirecting to setup…" as narrated navigation.**
Narrating a redirect to the user is an AI UX pattern. If the redirect is fast, this text will flash briefly and be confusing. If it is slow, "Setting up your account" is more informative. The ellipsis again adds a sense of automated narration.

No other violations found. The rest of `page.tsx` is structural JSX with no user-visible copy.
