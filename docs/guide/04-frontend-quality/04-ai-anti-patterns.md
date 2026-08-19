# Module 17 — AI Anti-Patterns

*[← Previous: AI for Documentation & Knowledge](./03-ai-for-documentation.md) | [Next: Playbooks Overview →](../05-playbooks/README.md)*

## Why this matters

This module is a mirror, not a new technique. Every pattern below is the exact opposite of something Modules 0 through 16 already taught you — vague prompting undoes Module 7, premature coding undoes Module 8, blind acceptance undoes Module 13 — and yet almost every engineer who reads this guide will still catch themselves doing at least one of these things the next time a deadline is tight and a conversation with AI is already open. The point of naming them explicitly isn't to shame anyone; it's so that the next time you're mid-sprint and about to type "just build the whole thing," some part of your brain recognizes the shape of what you're doing before you hit enter.

## Objective

Learn to recognize these seven failure modes in your own work and in code review — not as abstract mistakes other people make, but as the specific, reasonable-sounding shortcuts you are most likely to take under pressure. Every one of them quietly undoes a principle from an earlier module, so catching them in the moment is what makes the rest of this guide actually hold up in practice.

---

## The Seven Anti-Patterns

### 17.1 Vague Prompting

```text
Make this better.
```

No context, criteria, or expected output.

This happens for an almost sympathetic reason: you know exactly what "better" means in your own head — faster, more readable, fewer re-renders, whatever you were just staring at — and it doesn't occur to you that AI doesn't share that context. It reads "better" as an invitation to change whatever it notices first, which is rarely the axis you actually cared about. The cost isn't just a wasted response; it's a full review cycle spent figuring out that the diff optimized for the wrong thing, followed by a second, more specific prompt that you could have written the first time.

What to do instead: apply Module 7's Context → Goal → Constraints → References → Output structure. Name the dimension you want improved and what "better" is measured against.

Here's the same request on a different feature, done both ways:

```text
Make this better.
```

versus

```text
This ProductGallery component re-renders all 20 thumbnails whenever
the active image changes. Optimize it so only the active thumbnail's
selected state re-renders.

Constraints:
- No new dependencies.
- Preserve the existing prop API.
```

The second version gives AI a specific target (re-render count), a specific mechanism to look at (the active-image state), and a boundary on what not to touch. There's nothing left to guess.

---

### 17.2 Premature Coding

```text
Build the entire feature.
```

before architecture has been determined.

This is tempting precisely because AI is fast — generating a full feature in one shot *feels* like progress, and skipping the planning step feels like the efficient move under a deadline. What actually happens is that the architecture gets decided anyway, just implicitly, as a side effect of whatever component structure and state ownership AI's first draft happens to land on. By the time someone notices the state lives in the wrong place, it's already threaded through four files and three tests, and "fixing the architecture" now means a rewrite instead of a redirect.

What to do instead: run Module 8's Analyze → Plan → Review → Implement loop before any code gets written. A plan is cheap to redirect; a finished implementation is not.

A second example, on a different feature: asking AI to "build the entire admin permissions matrix" produces a working-looking screen — but it has also silently decided, without anyone reviewing it, whether permission checks live in a hook, a context provider, or route guards. If your team already has a convention for that (and most teams do), premature coding is how a fourth, inconsistent pattern gets introduced.

---

### 17.3 Context Dumping

Providing the entire repository without identifying what is relevant.

This usually comes from a defensible instinct: when you're not sure what AI needs, giving it everything feels safer than guessing wrong and leaving something out. But as Module 5 covers, more context is not free — it actively works against you. AI has to process irrelevant files alongside relevant ones, which increases the odds it picks up the wrong pattern, makes more assumptions to fill perceived gaps, and generates code that doesn't match the one existing implementation you actually wanted it to follow.

What to do instead: use Module 5's Context Pyramid and Minimum Sufficient Context principle. Start with the task and the smallest set of contracts, patterns, and reference files that solve it — not the folder tree.

A concrete instance: asking for a small fix to a date-formatting utility and attaching the entire `src/` directory — including unrelated API clients, style files, and test fixtures — "just in case." AI now has to sift through hundreds of irrelevant lines to find the one function you meant, and it's just as likely to anchor on a formatting convention from an unrelated file as the one you actually wanted followed.

---

### 17.4 Blind Acceptance

```text
AI generated it.
It compiles.
Done.
```

Compilation does not prove correctness.

This one happens because the checks that exist are visible and satisfying: green type-check, green lint, a diff that reads fine on a skim. Under time pressure, "it compiles" starts to feel like "it's correct," because stopping there is so much cheaper than continuing. The cost, as Module 13 lays out in detail, is that compilation and lint only catch a narrow band of mistakes — they say nothing about whether the double-click on a submit button fires two API calls, or whether an `any`-typed payload lets a typo'd field silently through.

What to do instead: work Module 13's verification pyramid bottom-up. Type-check and lint are the floor you clear in seconds, not the finish line — runtime behavior, tests that assert real values, and a human walkthrough of loading/empty/error states are what actually verify the code.

---

### 17.5 AI-Driven Architecture

```text
AI suggested Zustand.
Therefore, we should use Zustand.
```

Wrong mindset.

AI can suggest.

Engineers evaluate the trade-offs.

This slips in because AI states suggestions with total confidence, in the same tone it uses for everything else — a library recommendation reads exactly as authoritative as a correct type fix, so it's easy to treat it as a decision that's already been made rather than one option worth evaluating. It's also just easier: deferring to a suggestion takes less energy than working through the trade-offs yourself, especially when you're mid-task and the suggestion is sitting right there. The cost is architectural drift nobody actually signed off on — a dependency or pattern enters the codebase because it was convenient in one conversation, not because the team decided it was the right tool.

What to do instead: apply Module 0's Pilot vs Copilot framing. AI proposes, you decide — and "deciding" means actually weighing the trade-off, not rubber-stamping the first plausible-sounding answer.

Here's how this pattern actually spreads in practice: you ask AI for a way to manage one piece of local UI state on a checkout page — say, whether the promo code input is expanded — and it suggests reaching for Zustand since it's already a dependency in the project. That's a reasonable call for that one piece of state. But over the next few prompts, "we're already using Zustand for the promo field" quietly becomes the justification for pushing cart totals and shipping-address form state into the same store too — even though cart totals are properly server state owned by React Query, and the shipping address already belongs to your form library. Nobody made an explicit decision to duplicate server state into a client store; it just accreted, one AI suggestion at a time, because no one stopped to ask whether the earlier precedent actually applied.

---

### 17.6 Endless Conversation

A single conversation keeps accumulating unrelated work:

```text
Task 1
→ Fix
→ Another fix
→ Another feature
→ Refactor
→ Bug
→ Another feature
```

The context becomes polluted.

When the scope changes significantly:

> **Start a new focused context.**

This is the path of least resistance: the conversation is already open, AI already "remembers" your codebase's conventions from the last ten messages, and starting a new thread feels like throwing away useful context. The problem is that everything from the earlier, unrelated tasks stays in the context window and starts leaking into later answers — constraints from Task 1, file references from Task 2, a naming convention someone corrected three tasks ago — none of which apply to what you're actually working on now, but all of which AI will happily keep drawing on.

What to do instead: when the *category* of work changes — not just the specific detail, but the kind of task (bug fix versus new feature versus refactor) — start a new conversation. Carry forward only the specific artifact you actually need (a file, a decision, an agreed plan), not the whole prior thread. A fresh, focused context every time the task changes shape is worth more than a long one that "remembers everything."

A realistic version of how this goes wrong: a conversation starts with "fix the notification bell not clearing its unread count on logout." Once that's resolved, it's tempting to keep going in the same thread — "while we're here, can you also refactor the settings page's tab component" — and then "oh, and add a dark mode toggle to settings." Three unrelated units of work now share one context. By the fourth message, AI's suggestions for an unrelated notifications follow-up start incorporating settings-page tab conventions that have nothing to do with notifications, because both are sitting in the same polluted window. Splitting these into three separate conversations from the start would have kept each one focused and each diff reviewable on its own terms.

---

### 17.7 Over-Automation

Not every task should be delegated to AI.

Important decisions such as:

* Architecture
* Security
* Data model
* Major dependencies
* Breaking changes

should have explicit human ownership.

This creeps in for a simple reason: AI is available for everything, so it's tempting to route every decision through it, including ones that were never really about speed. Generating a component is a speed problem; deciding to change a shared data model or introduce a new dependency is a judgment-and-accountability problem, and those don't get easier just because you can produce an answer to them in five seconds. The cost of over-automating them is that consequential decisions end up with no clear owner — when a schema change turns out to be hard to migrate, or a "small" new dependency becomes load-bearing across the app, there's no one who can explain why it was made that way, because no one actually made it; it was accepted.

What to do instead: treat Module 0's Pilot-owned list — architecture, business logic, technical decisions, security, performance, correctness, final review — as a literal checklist of what should never be fully automated. AI can explore the trade-offs and draft the proposal for any of these; it should never be the one who decides.

A concrete case: AI proposes a breaking change to a shared `Button` component's prop API — say, renaming `variant` to `intent` — because it makes the naming more consistent with a newer component it just wrote. That's a reasonable suggestion. Accepting it wholesale and letting AI apply it across every consumer in the design system, without a human decision on migration strategy, deprecation window, or who needs to be notified, is over-automation: a breaking change with real blast radius just happened because it was easy to say yes to, not because anyone weighed the rollout cost.

---

## Key Takeaways

Use this as a quick self-check before you send a prompt or approve a PR:

* **Vague Prompting** — did you name a specific goal and success criteria, or just say "make it better"?
* **Premature Coding** — did architecture get decided on purpose, or did it just fall out of the first draft?
* **Context Dumping** — did you hand over the minimum sufficient context, or the whole repository "just in case"?
* **Blind Acceptance** — did you verify beyond "it compiles," or is a green type-check standing in for correctness?
* **AI-Driven Architecture** — was a suggestion evaluated on trade-offs, or adopted because AI said it with confidence?
* **Endless Conversation** — does this thread hold one focused task, or has it quietly become three unrelated ones?
* **Over-Automation** — does this change touch architecture, security, data model, dependencies, or a breaking change — and if so, did a human actually decide, not just approve?

---

## Try It Yourself

1. Pull up your last five AI conversations or the last five PRs where AI wrote a meaningful part of the diff. For each one, go through the seven patterns above and mark which ones you can honestly say you fell into, even partially. Most engineers find at least two or three across five conversations — the goal isn't a perfect score, it's noticing which pattern you personally default to under pressure.
2. Pick the one anti-pattern you found most often in exercise 1. Find the specific module section that addresses it (linked throughout this module) and re-read it with that recent example in mind. Then write one sentence describing what you'd do differently next time you feel the same pressure that caused it.

---

*[← Previous: AI for Documentation & Knowledge](./03-ai-for-documentation.md) | [Next: Playbooks Overview →](../05-playbooks/README.md)*
