# Module 8 — Plan Before Code

*[← Previous: AI Communication Framework](../02-thinking-tools/04-ai-communication-framework.md) | [Next: AI as a Coding Partner →](./02-ai-coding-partner.md)*

## Why this matters

Picture the more common version of the failure from Module 0: an engineer asks AI to "add a notifications panel to the dashboard," AI comes back with 800 lines spanning a new global store, three components, and a polling hook — and only *after* the PR is up does anyone notice the unread count is duplicated in both Zustand and the API response, silently drifting out of sync under load. Untangling that after the fact means unpicking state that's already threaded through five files. Compare that to catching the same mistake in a two-line plan review, before a single component exists — "the notification list should be server state, not a Zustand store" — which costs one sentence and zero rework. The difference between those two outcomes isn't the model, the prompt wording, or luck; it's whether an architecture decision got reviewed while it was still just a paragraph of text.

## Objective

Teach the workflow:

> **Analyze → Plan → Review → Implement**

This is the module where Modules 4, 5, and 7 stop being separate skills and become a sequence you run, in order, before AI writes a single line of implementation code for anything non-trivial.

---

## 8.1 Analyze

Jumping straight to "build X" skips a step that costs almost nothing and catches a lot: understanding what already exists before deciding what to add. If you ask AI to generate code before it has looked at your components, your state patterns, and your data flow, it's not being lazy — it genuinely doesn't know they're relevant unless the analysis step forces it to look. Asking for analysis first, and explicitly forbidding code in the same turn, keeps AI from collapsing "understand the problem" and "write the solution" into one pass where the first one gets skipped.

The curriculum's example:

```text
Analyze this requirement and the existing code.

Identify:
1. Relevant components.
2. Existing patterns.
3. State ownership.
4. Data dependencies.
5. Potential edge cases.

Do not write code.
```

The "do not write code" line is doing real work here — without it, most models will treat "analyze" as a warm-up and start generating anyway, and once code exists in the conversation it anchors everything that follows, plan included.

**Worked example — notifications panel.** Say the requirement is: add a notifications panel to the dashboard header. A bell icon shows an unread count; clicking it opens a panel of recent notifications; users can mark one or all as read; the list loads more as they scroll. Before asking for a plan, you run:

```text
Analyze this requirement and the existing code.

Requirement: add a notifications panel to the dashboard header —
bell icon with unread badge, opens a panel of recent notifications,
mark one or all as read, loads more on scroll.

Identify:
1. Relevant components.
2. Existing patterns.
3. State ownership.
4. Data dependencies.
5. Potential edge cases.

Do not write code.
```

A useful analysis comes back naming things like: the existing `Popover` component used by the user menu, the `useInfiniteQuery` pattern already used in the activity log, the fact that unread count and notification list both come from the same `/notifications` endpoint, and edge cases such as a new notification arriving while the panel is open, marking-as-read racing with the next page load, and the empty state. None of that required guessing — it's all visible in the codebase, which is exactly why it's worth asking for before anyone commits to an approach.

---

## 8.2 Plan

Analysis tells you what exists; a plan is where AI commits to a specific shape for the solution — and a proposal on paper is far cheaper to correct than a proposal already turned into 15 files of code. Asking for the plan as a distinct step, separate from analysis and separate from implementation, gives you a single artifact small enough to actually read and disagree with. This is also where the "context is more important than a clever prompt" lesson from Module 5 pays off: the plan is only as good as the patterns AI was shown in the Analyze step.

```text
Propose an implementation plan.

Include:
- component structure
- state ownership
- data flow
- files to modify
- files to create
- potential risks
```

**Worked example, continued.** For the notifications panel, the plan that comes back might look like this:

```text
Component structure:
  NotificationsBell (existing header, add badge)
  NotificationsPanel (new, Popover-based)
  NotificationItem (new)

State ownership:
  - unreadCount and notifications list: new Zustand store,
    `useNotificationsStore`, populated by a 30s poll.
  - panel open/closed: local state in NotificationsBell.

Data flow:
  poll → store → NotificationsBell (badge) + NotificationsPanel (list)

Files to create:
  NotificationsPanel.tsx, NotificationItem.tsx,
  useNotificationsStore.ts

Files to modify:
  DashboardHeader.tsx (mount NotificationsBell)

Risks:
  - polling interval vs. real-time expectations
  - pagination while marking as read
```

That's a plausible-looking plan. It's also wrong in a way that won't show up until production: it invents a second source of truth for data that already has one.

> **In Claude Code, specifically.** Analyze → Plan is a literal mode, not just a prompting discipline: press `Shift+Tab` until the status bar shows `⏸ plan mode on` (or start with `claude --permission-mode plan`), and Claude reads files and proposes without touching disk until you approve — the "do not write code" instruction above is enforced by the tool, not just requested. `Ctrl+G` opens the proposed plan in your text editor so you can edit it directly instead of only replying in chat. Per the [official best-practices guide](https://code.claude.com/docs/en/best-practices#explore-first-then-plan-then-code): *"planning is most useful when you're uncertain about the approach, when the change modifies multiple files, or when you're unfamiliar with the code being modified. If you could describe the diff in one sentence, skip the plan."*

---

## 8.3 Human Review

This is the step that actually earns the "plan before code" name — a plan you don't review is just code generation with extra latency. Your job here is not to check whether the plan is internally consistent (AI is good at that); it's to check whether it's consistent with *decisions AI can't see*: your project's conventions, your team's state-ownership rules, and trade-offs nobody wrote down anywhere AI could read them. This is where Module 4's state-ownership categories (local, server, global, URL) stop being theory and become the checklist you run the plan against.

```text
Is the architecture correct?
Is the state in the right place?
Does this follow our project conventions?
Are there unnecessary changes?
```

**Worked example, continued.** Running that checklist against the notifications plan above surfaces the problem immediately: notifications and unread count are fetched from an API — they're server state — but the plan puts them in a new Zustand store, which means the app now has to keep that store in sync with the server by hand (polling, invalidation, race conditions on mark-as-read) instead of letting React Query do it. That's not a style preference; it's the same category of mistake the curriculum flags for the user-filter example, just on a different feature. You send the correction back as its own instruction, not folded into a implement request:

```text
Revise the plan.

Notifications and unread count are server state, not
component state. Use React Query (useNotificationsQuery)
with invalidation on the mark-as-read mutation, matching
the pattern in useOrdersQuery.ts. Only the panel's
open/closed state stays local to NotificationsBell.
```

The curriculum's original example is the same move on a different mistake:

```text
Revise the plan.

The filter state should be URL-owned,
not component-owned.
```

Both corrections cost one message. Neither would have cost one message if caught after implementation — they'd have cost a rewrite of the data layer.

---

## 8.4 Implement

Only after the plan is approved does AI touch code — and even then, "approved" means "approved for the step you're about to ask for," not a blank check to build the whole plan in one shot. Scoping the implement request to one step keeps the diff reviewable and keeps AI from making follow-on decisions on steps 2 through 6 that you never actually signed off on.

```text
Implement step 1 only.

Do not modify unrelated files.
```

**Worked example, continued.** For the corrected notifications plan, step 1 is the query hook, not the UI:

```text
Implement step 1 only: the useNotificationsQuery hook.

Follow the pattern in useOrdersQuery.ts.
Return { notifications, unreadCount, isLoading, error }.
Do not build NotificationsPanel or NotificationsBell yet.
Do not modify DashboardHeader.tsx.
```

Notice what this prompt does *not* ask for: it doesn't say "and then wire it up to the header," even though that's the obvious next step. That's deliberate — the next step gets its own turn, and its own review, in Section 8.5.

---

## 8.5 Incremental Implementation

The instinct to ask for everything at once is understandable — it feels faster to get one big answer than five small ones. It isn't, once you account for what happens when something in that big answer is wrong: a 1,000-line diff hides its assumptions inside other correct-looking code, and finding the one wrong assumption means reading all 1,000 lines, not just the ones that changed. A sequence of small steps, each reviewed before the next begins, surfaces a wrong assumption at the size of a single file, while it's still cheap to redirect.

Avoid:

```text
Generate 1,000 lines of code.
```

Prefer:

```text
Step 1
→ Review
→ Step 2
→ Review
→ Step 3
→ Review
```

**Worked example, continued.** For the notifications panel, that sequence looks like:

```text
Step 1: useNotificationsQuery hook
→ review: does it match useOrdersQuery's error/loading shape?
Step 2: NotificationsBell badge, wired to the hook
→ review: does the badge handle 0 and 99+ correctly?
Step 3: NotificationsPanel shell + NotificationItem
→ review: empty state, keyboard focus when opened
Step 4: mark-as-read mutation + optimistic update
→ review: does it race with the next poll/page fetch?
Step 5: infinite scroll for older notifications
→ review: does scroll position survive a mark-as-read?
Step 6: tests
```

Compare that to asking for "the full notifications feature" in one prompt: if step 4's optimistic update is wrong, you'd otherwise have to find that bug inside a diff that also touches the bell, the panel, the scroll logic, and the tests, all generated in the same pass and all now suspect.

Benefits:

* Smaller diffs
* Easier debugging
* Easier rollback
* Easier detection of assumptions
* Better architectural control

---

## Key Takeaways

* Run Analyze → Plan → Review → Implement in order for any non-trivial task — skipping straight to "build X" means architecture decisions get made inside generated code instead of in a paragraph you can correct for free.
* Analyze with "do not write code" as an explicit instruction, or AI will treat analysis as a warm-up and start generating anyway.
* A plan is only worth writing if you actually review it against things AI can't see — your project's state-ownership rules and conventions — not just whether it reads coherently.
* When you correct a plan, state the correction as its own instruction ("Revise the plan: X should be Y"), the same way you'd state any other constraint from Module 7.
* Scope every implement prompt to one step and forbid changes to unrelated files — this is what keeps the diff small enough to actually review.
* Incremental delivery isn't slower in practice: a wrong assumption caught at step 2 costs a re-prompt, while the same assumption buried in a 1,000-line generation costs a full re-read to even find it.

## Try It Yourself

1. On your next non-trivial ticket, force yourself through Analyze → Plan → Review before letting AI write a single line. Write down, in one sentence, what the review step caught — a wrong state ownership call, an unnecessary file, a convention violation, or "nothing, the plan was right." If it's the last one, that's useful data too.
2. Take a feature you'd normally ask AI to build in one shot, and instead break its plan into the same kind of step list used for the notifications panel above (one step per file or concern, with a review in between each). After you finish, note whether any step's review changed something you would have missed in a single big diff.

---

*[← Previous: AI Communication Framework](../02-thinking-tools/04-ai-communication-framework.md) | [Next: AI as a Coding Partner →](./02-ai-coding-partner.md)*
