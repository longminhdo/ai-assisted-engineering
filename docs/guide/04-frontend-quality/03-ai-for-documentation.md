# Module 16 — AI for Documentation & Knowledge

*[← Previous: AI for Testing](./02-ai-for-testing.md) | [Next: AI Anti-Patterns →](./04-ai-anti-patterns.md)*

## Why this matters

Picture a new hire dropped into a codebase with a component called `LegacyOrderTable` — 900 lines, no comments, three different naming conventions depending on which era of the team wrote which section, and a `// TODO: figure out why this is here` from someone who left two years ago. They spend two full days clicking through it, adding `console.log` statements, and asking around on Slack, before they can safely touch it. An experienced engineer who already holds the mental model in their head could have explained it in ten minutes — but that engineer is in back-to-back meetings, and writing it down formally feels like a task that never makes it to the top of anyone's backlog. This is the same reason your README says "TODO: document setup" from eighteen months ago, and the same reason nobody can confidently explain what the `checkout` module actually does anymore without reading all of it first. Documentation work gets skipped not because it isn't valuable, but because writing it by hand competes with shipping features and always loses.

## Objective

AI is not only for coding — it's a knowledge tool. Explaining, documenting, and summarizing what already exists in your codebase is one of the highest-leverage, lowest-risk things you can hand to AI, because the source of truth (the code itself) is right there for it to read.

---

## 16.1 Explaining Unfamiliar or Legacy Code

The core reasoning here is simple: AI can read an entire file, trace every call site, and hold all of it in context at once, in seconds — something a human has to do serially, one function at a time, while also trying to remember what the last function did. This makes AI exceptionally good at the specific failure mode of "I can see the code but I don't know what it's *for*," which is most of what makes legacy code intimidating. It's also low-risk in a way generation tasks aren't: you're not asking AI to change anything, so a wrong or incomplete explanation costs you a few minutes of re-reading, not a bug in production. That asymmetry — high potential time savings, low cost when it's imperfect — is exactly why this is one of the safest places to lean on AI heavily, even if you're still building trust in it for other tasks. The one thing to watch for is treating the explanation as ground truth rather than a hypothesis: AI can misread intent from code that happens to look like something else, so for anything you're about to modify, verify the explanation against actual behavior (run it, add a log, check a test) before you act on it.

**Worked example — the legacy order table.** You inherit `LegacyOrderTable.tsx`, 900 lines, no documentation, referenced from four other components you also don't fully understand.

```text
Explain this component to someone who has never seen it.

Cover:
- what it renders and when
- what props/state actually affect its behavior vs. what looks unused
- any non-obvious business logic (discount rules, sorting, edge cases)
- what would break if I deleted the `useEffect` on line 340
- anything that looks like a workaround for a bug elsewhere
```

The last two lines matter more than they look. Asking "what would break if I deleted this" forces AI to trace forward from a specific piece of code to its consequences, instead of giving you a generic paraphrase of what the code does — which is closer to what you actually need before touching it. Asking it to flag workarounds surfaces the kind of tribal knowledge ("this `setTimeout` is here because the parent re-renders twice on mount for reasons nobody fixed") that would otherwise only live in one person's head.

**Second example — an undocumented pricing utility.** Say you find a function `applyRegionalAdjustment(price, region, flags)` used across the checkout flow, with no comments and a name that doesn't tell you what "adjustment" means.

```text
Explain what applyRegionalAdjustment does, including:
- every distinct code path through it (list the conditions that select each one)
- what "flags" controls, inferred from how it's used at each call site
- whether any call site passes an unexpected combination of arguments
- what this function assumes about its caller that isn't enforced by its types
```

That last question is doing real work: it's asking AI to find the implicit contract, the assumptions a function makes that aren't captured anywhere in its signature. In legacy code, that's usually exactly the thing that causes the next bug when someone calls the function slightly differently than everyone before them did.

---

## 16.2 Generating READMEs and API Documentation

READMEs and API docs go stale for a predictable reason: writing them by hand means someone has to describe the code in prose, which takes real effort and produces no visible feature progress, so it's the first thing cut under deadline pressure. AI removes the effort part of that equation — it can read a module's exports, types, and usage sites and produce a first draft in the time it takes you to write the prompt. That doesn't make the docs correct by default; AI documents what the code *appears* to do, including any bugs or inconsistencies baked into it, so a generated README needs the same skepticism you'd apply to generated code. But "correct a good first draft" is a much smaller task than "write documentation from a blank page," and closing that gap is usually what determines whether documentation gets written at all.

**Worked example — a shared component library package.** Your team maintains an internal `@company/ui-notifications` package with a handful of exported components and hooks, and it has never had a README.

```text
Generate a README for this package based on its actual exports and usage.

Include:
- what the package is for, in one paragraph
- installation/setup
- a usage example for each exported component and hook, using real prop
  names and types from the source
- known limitations or gotchas visible in the code (e.g. required
  provider wrappers, SSR caveats)
- do not describe props or behavior that isn't actually present in the code
```

The last line is worth keeping in every documentation prompt you write. Without it, models tend to round up to "what a package like this usually supports" rather than "what this package actually supports" — describing a `variant` prop that doesn't exist because similar libraries usually have one.

**Second example — API documentation for an internal hooks module.** You have a `useOrderStatus(orderId)` hook used across five screens, with return types that have grown organically and no documented contract.

```text
Generate API documentation for useOrderStatus.

Include:
- parameters, with types and whether each is required
- every field on the returned object, with type and a one-line description
  of when it changes
- loading/error states this hook can be in, and how to distinguish them
- a minimal usage snippet
- explicitly call out any field that is deprecated or unused by current callers
```

That final bullet is a documentation task with a debugging side effect: generating accurate API docs for a hook that's grown over time is often how you first notice a field three callers reference but nothing actually sets anymore.

---

## 16.3 Summarizing Architecture

A single file or function is small enough to read in full; a module, feature area, or subsystem usually isn't — by the time you've read every file, you've forgotten what the first one did. Architecture summaries exist to compress that: instead of holding twelve files in your head at once, you hold one page that tells you how they relate. AI is well suited to producing this compression because the "how do these files relate" question is really a graph-traversal problem (what imports what, what calls what, what state flows where), and that's mechanical work a model can do exhaustively across an entire directory rather than by sampling a few files and guessing at the rest. Use this whenever you're about to work in an area you don't own, before a design discussion about changing it, or when you want a written artifact to check your own understanding against.

**Worked example (from the curriculum).**

```text
Analyze this module and generate a developer-oriented
architecture overview.

Include:
- responsibilities
- dependencies
- data flow
- important abstractions
- known risks
```

The "known risks" line is what separates a genuinely useful architecture summary from a table of contents. It pushes AI to make a judgment call — this coupling looks fragile, this abstraction is leaking, this piece of state is mutated from three unrelated places — rather than just neutrally listing what exists. Read that section with the same skepticism as any AI claim, but treat it as a prioritized list of places to look closer, not a verdict.

**Second example — a cross-cutting notifications system.** Suppose "notifications" in your app touches a WebSocket connection, a Zustand store, a badge count in the header, a dropdown panel, and a toast queue — five files, no single one of which tells the whole story.

```text
Summarize the architecture of the notifications system across these
files: socket.ts, notificationStore.ts, NotificationBadge.tsx,
NotificationPanel.tsx, ToastQueue.tsx.

Include:
- how a single notification event flows from the socket to something
  the user sees, end to end
- which pieces of state are the source of truth vs. derived
- what happens if the socket disconnects and reconnects
- any place two of these files can get out of sync with each other
```

Asking for the disconnect/reconnect behavior and the out-of-sync case specifically is what turns this into something you can act on — a generic "here's what each file does" summary won't surface that the badge count and the panel's unread list are independently derived and can drift apart, which is exactly the kind of cross-file bug that's invisible when you review each file in isolation.

---

## 16.4 Migration and Onboarding Guides

Migrations and onboarding are both "explain the path from A to B" problems, just at different scales — one is code moving from an old pattern to a new one, the other is a person moving from knowing nothing about a codebase to being productive in it. Both suffer from the same neglect as READMEs: writing a migration guide takes time away from doing the migration, and writing onboarding docs takes time away from whatever the senior engineer who understands the system was actually hired to build. AI can produce a first draft of either by reading the current state (the deprecated API's usage sites, or the feature area a new hire will land in) and describing the path out of it, which is often enough to unblock someone even before a human reviews and corrects it.

**Worked example — migrating off a deprecated API.** Your team is moving off a homegrown `useLegacyFetch` hook toward React Query, and twenty components still use the old one.

```text
Generate a migration guide for moving a component from useLegacyFetch
to React Query's useQuery.

Include:
- a side-by-side before/after example using one real call site from
  this codebase
- how loading, error, and retry behavior differ between the two and
  what callers need to change to preserve current behavior
- anything the old hook does that useQuery doesn't handle automatically
  (e.g. custom caching, manual refetch triggers) and how to replicate it
- a checklist to verify a migrated component still behaves correctly
```

Asking for the behavior differences, not just the syntax differences, is what makes this useful for twenty components instead of one. A migration guide that only shows "here's the new syntax" will get every call site compiling; a migration guide that also flags "the old hook retried 3 times silently and useQuery defaults to 3 retries but surfaces errors differently" is what prevents a silent behavior regression from shipping across all twenty at once.

**Second example — onboarding documentation for a confusing settings page.** A `UserSettingsPage` has accumulated five years of feature flags, three different form libraries from different eras, and one section that's disabled by a flag nobody remembers the purpose of.

```text
Generate an onboarding explainer for this settings page aimed at an
engineer joining the team who has never seen it.

Include:
- a map of which section owns which piece of state, and which form
  library each section uses
- what each feature flag in this file currently does, based on how
  it's actually used in the code
- what looks safe to modify vs. what has non-obvious dependencies
  elsewhere in the app
- three questions a new engineer should ask a teammate before changing
  anything here, based on what the code alone can't tell you
```

That last bullet is deliberately part of the prompt, not an afterthought. A good onboarding document from AI should end by telling you what it *couldn't* determine from the code alone — the intent behind a flag, the history behind a workaround — because that's the boundary between what reading the code can teach you and what still requires asking a person.

---

## Key Takeaways

* AI's advantage in documentation work is reading speed and exhaustiveness, not judgment — it can trace every call site in seconds, but you still verify anything you're about to act on.
* Generated docs describe what the code *appears* to do, bugs and inconsistencies included — treat a generated README or API doc as a first draft to correct, not a finished artifact.
* Always tell AI not to invent behavior the code doesn't have ("don't describe props that aren't in the source") — otherwise it rounds up to what's typical for that kind of code, not what's actually there.
* The best documentation prompts ask for a judgment, not just a description: "known risks," "what would break if I deleted this," "where can these get out of sync" — these surface the things a plain summary would hide.
* Migration and onboarding guides both benefit from explicitly asking what AI *couldn't* determine from the code — that boundary tells you exactly what still needs a human conversation.
* This is one of the lowest-risk uses of AI in your workflow: a wrong explanation costs you re-reading time, not a production bug, which makes it a good place to build trust in the tool.

## Try It Yourself

1. Pick the single most confusing file in your current codebase — the one you'd least want to be assigned a bug in. Prompt AI for a full explanation using the structure in 16.1 (what it does, what would break if you deleted a specific suspicious-looking piece, what looks like a workaround). Then find a teammate who actually wrote or last touched it and compare notes — where did AI's explanation match their mental model, and where did it miss context only they had?
2. Take one internal package, hook, or module in your codebase that has no README or stale documentation, and generate one using the prompt structure in 16.2, with the explicit "don't describe what isn't there" instruction. Read the result against the actual source line by line and correct anything that's rounded up or invented — then decide if it's worth committing as the real README.

---

*[← Previous: AI for Testing](./02-ai-for-testing.md) | [Next: AI Anti-Patterns →](./04-ai-anti-patterns.md)*
