# Module 5 — Context Engineering

*[← Previous: Problem Decomposition](./01-problem-decomposition.md) | [Next: Visual-First Design Analysis →](./03-visual-first-design-analysis.md)*

## Why this matters

Picture a PR where the AI-generated modal doesn't call the shared toast helper for errors, rolls its own `useState`-based fetch instead of the team's query hook, and names its props nothing like the rest of the codebase. Nobody told the AI those things existed — it just wrote what it knows works "in general," and in review someone has to redo half of it. Now picture the opposite: an engineer pastes in the team's existing `OrderFilterModal.tsx` and says "build the new one like this," and the diff comes back looking like it was written by someone who's been on the team for a year. Same model, same task type, wildly different outcome. The difference was never the prompt's wording — it was what the AI was shown before it started typing.

## Objective

Internalize a single idea before anything else in this guide: **good context is more important than a clever prompt.** You can phrase a request beautifully and still get generic, off-pattern code if the AI never saw your types, your components, or your conventions. Most of what looks like "prompting skill" is actually context selection.

---

## 5.1. What Does AI Need to Know?

An AI model has no persistent memory of your codebase between sessions (and even within a long session, it only knows what's been shown or read). Every time you start a task, you are handing it a blank slate with strong generic instincts — it knows React and TypeScript idioms in general, but it does not know *your* React and TypeScript idioms. Your job is to close that gap before it starts generating, not after.

For a typical frontend task, the useful context tends to stack up like this:

```text
Requirement
+
Types
+
API Contract
+
Existing Components
+
Existing Hooks
+
Design System
+
Relevant Business Rules
+
Constraints
```

Notice what's absent from that list: "the entire repository." You do not need to, and should not, hand over everything you have. You need the specific pieces that shape *this* task's decisions — the type that defines the data shape, the hook that already does data fetching your way, the component that already solved a similar UI problem, the business rule that changes the logic in a non-obvious way. Everything else is noise the AI has to wade through to find the signal.

**Worked example — a checkout flow.** Say the task is "add a promo code field to checkout." The requirement alone tells the AI almost nothing useful. What it actually needs is: the `CheckoutFormValues` type (so it knows the shape it's extending), the `applyPromoCode` API contract (so it doesn't invent a different one), the existing `CheckoutSummary` component (so the new field lands in the right visual and data slot), the `useCheckoutMutation` hook (so promo application goes through the same request/error path as the rest of checkout), and the business rule that promo codes can't stack with an existing employee discount. Hand over those five things and the requirement sentence, and the AI is now working from the same mental model you have — not guessing at one.

---

## 5.2. Context Pyramid

Not all context is equally important, and treating it that way is how sessions go sideways. It helps to think of it as a pyramid, where the task itself sits at the tip — the thing actually being decided right now — and each layer below is progressively less specific to that decision and more expensive to include.

```text
                    Full Repository
                         ▲
                         │
                  Related Modules
                         ▲
                         │
                 Existing Patterns
                         ▲
                         │
                    Contracts
                         ▲
                         │
                       Task
```

Read the pyramid from the top down as "reach for this only if the layer below wasn't enough." Start at **Task** — what, precisely, are you asking for. Then **Contracts** — the types and API shapes that constrain the answer. Then **Existing Patterns** — the one or two components/hooks that show how this kind of thing is normally built here. Then **Related Modules** — other files that interact with what you're building, needed only if the task actually touches them. **Full Repository** sits at the base because it is the least targeted and most expensive form of context — reach for it only when you are doing broad exploration ("where else is this pattern used?"), never as your default starting point for an implementation task.

**Worked example — a settings page.** Task: "add a two-factor authentication toggle to the account settings page." Applying the pyramid: start with the task statement itself. Add the `UserSecuritySettings` contract (the shape of the data being toggled). Add the existing `NotificationsToggleRow` pattern, since it's the closest existing example of a labeled toggle row wired to a mutation with optimistic UI. You probably don't need "related modules" here at all — 2FA toggle doesn't touch billing or the dashboard. And you almost certainly don't need the full repository. If you found yourself reaching for the whole codebase for a single toggle, that's a signal you've skipped past cheaper, more targeted layers instead of climbing the pyramid in order.

---

## 5.3. Context Pollution

There's an intuitive but wrong belief that more context is always safer — "if I give the AI everything, it can't possibly miss something important." In practice the opposite happens. Every additional file you include is something the model has to weigh against the task, and irrelevant files don't sit passively in the background — they compete for attention with the files that actually matter, and they can visibly pull the output toward the wrong pattern.

Bad example, still seen constantly in real sessions:

```text
20 files
+
entire API folder
+
all components
+
all CSS
+
all hooks
```

The AI now has to process a large amount of irrelevant information, and that has real consequences, not just a slower response:

* Loss of focus
* Wrong pattern selection
* More assumptions
* Unnecessary code
* Increased hallucination risk

Concretely: "wrong pattern selection" means if you dump 12 different modal components into context and only one is the right reference, the AI may blend conventions from several of them, or pick whichever one it saw most recently rather than the one that's actually canonical. "More assumptions" means when the actually-relevant file is buried among 19 irrelevant ones, the model may not even weight it correctly and will fill the gap with a generic guess instead. This is not a hypothetical failure mode — it's the most common reason "I gave it the whole context and it still got it wrong" sessions happen.

**Worked example — a notifications panel.** An engineer asks for a new "mark all as read" button in the notifications panel and, to "be safe," pastes in the entire `src/features/notifications/` directory (18 files: the panel, three different notification-item variants for three different notification types, the whole notifications API client, a Storybook file, and two now-unused legacy components left over from a redesign). The AI's response mixes styling from the wrong item variant, half-heartedly tries to update the unused legacy files too, and reimplements a "mark as read" API call that already existed in the API client, just named slightly differently, because the actual call was one function among sixty and got lost in the noise. The fix isn't a better prompt — it's handing over three files instead of eighteen: the panel component, the specific `useNotifications` hook, and the one API function that matters.

---

## 5.4. Minimum Sufficient Context

The operating principle that ties 5.1–5.3 together: **give AI everything it needs, and nothing it doesn't.** This is deliberately symmetric. Under-context produces generic, assumption-riddled output. Over-context produces distracted, off-pattern output. The skill being trained here is the same one you'd use briefing a new contractor who has one day on the project — you wouldn't hand them the whole repo and say "read this," and you wouldn't hand them one sentence and walk away. You'd hand them exactly the handful of files and facts that let them do this one task correctly.

For a Filter Modal, the curriculum's own example, the relevant context might be:

```text
UserFilter type
Filter API
Existing FilterModal
Existing Form
Existing Select
Validation schema
One similar feature
```

Not the entire repository. Seven items, each one earning its place because it directly shapes a decision the AI has to make — the type shapes the fields, the API shapes the request/response handling, the existing modal and form and select shape the structure and styling, the validation schema shapes the rules, and one similar feature (already-built and working) shapes everything by example rather than by description.

**Worked example — a dashboard widget.** Task: "add a 'revenue this month' widget to the dashboard, matching the existing widgets." Minimum sufficient context here is short: the `DashboardWidget` shared wrapper component (defines the card shell, loading state, and error state every widget already uses), the `useRevenueMetrics` query hook if it exists — or, if it doesn't, the sibling `useActiveUsersMetrics` hook as the pattern to copy — and one existing widget, say `ActiveUsersWidget.tsx`, as the direct structural reference. That's three files. You do not need the routing layer, the auth context, or the other eleven widgets that don't share this data shape. If you're unsure whether something belongs in the set, ask: "does this file change a decision the AI will make in this task?" If the answer is no, leave it out.

---

## 5.5. Repository Pattern Learning

There is a specific trap worth calling out on its own: describing your conventions in prose instead of showing them. It feels thorough — "let me explain how our project does things" — but prose descriptions of a pattern are lossy and get outdated. Text like this is common and much weaker than it feels:

```text
Our project uses a custom query hook.
Our project uses a custom modal.
Our project handles errors with toast.
```

Each of those sentences describes an outcome but not the *shape* of the code that produces it — naming conventions, file structure, exact error-handling call sites, how loading and empty states are rendered, how tests are structured around it. An AI reading only the sentence has to reconstruct all of that from generic training knowledge, which is exactly the gap that produces off-pattern code.

The stronger move is to provide the actual implementations instead:

```text
OrderFilterModal.tsx
useOrderQuery.ts
OrderFilter.test.tsx
```

Then tell the AI explicitly what to do with them:

```text
Use these files as reference implementations.

Follow their:
- component structure
- naming conventions
- state management
- error handling
- testing patterns
```

This works because you're no longer asking the AI to infer your conventions from a description — you're asking it to pattern-match against a concrete, working example, which is a task it's very good at.

**Worked example — a settings page, again.** Instead of telling the AI "our settings pages use a two-column layout with a sticky save bar, validate on blur, and show a success toast on save," point it at the actual `NotificationSettingsPage.tsx`, its `useNotificationSettingsForm.ts` hook, and its test file, and say: "Build the new `PrivacySettingsPage` the same way — same layout, same save/validate/toast behavior, same test structure." The AI now has the sticky-save-bar CSS classes, the exact toast call, the exact validation timing, and the exact test setup in front of it, instead of a paraphrase of those things that it has to reverse-engineer.

Core principle:

> **Show AI the pattern instead of describing the pattern.**

---

## Key Takeaways

- Good context beats a clever prompt — most "prompting skill" is actually context selection.
- Climb the Context Pyramid from the task outward (Task → Contracts → Existing Patterns → Related Modules → Full Repository); don't start at the base.
- More context is not safer. Irrelevant files cause loss of focus, wrong pattern selection, and more assumptions — this is context pollution, not thoroughness.
- Aim for minimum sufficient context: the smallest set of types, contracts, and reference files that actually shapes the decisions the AI needs to make.
- Show, don't describe. Point the AI at real files (a component, a hook, a test) rather than summarizing your conventions in prose — patterns are lossy in words and exact in code.

## Try It Yourself

1. Before your next AI session, write down the minimum sufficient context for the task you're about to hand off — list the specific types, contracts, and 2-4 reference files it actually needs, the way the worked examples above did. Then open the session and give AI only that list instead of describing the codebase or pasting in a whole directory. Compare the output quality to your last session where you over-shared or under-shared context.
2. Pick one recent AI-assisted PR (yours or a teammate's) that had to be reworked in review. Identify whether the root cause was under-context (AI guessed at conventions it was never shown) or context pollution (AI had access to too much and picked the wrong pattern). Write one sentence naming which failure mode it was — that diagnosis is the first step to fixing your context habits going forward.

---

*[← Previous: Problem Decomposition](./01-problem-decomposition.md) | [Next: Visual-First Design Analysis →](./03-visual-first-design-analysis.md)*
