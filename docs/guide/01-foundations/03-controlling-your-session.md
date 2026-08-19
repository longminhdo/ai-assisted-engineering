# Module 2 — Controlling Your Session

*[← Previous: How AI Processes Your Request](./02-how-ai-works.md) | [Next: Choosing Your Model & Effort →](./04-choosing-model-and-effort.md)*

## Why this matters

Picture an engineer who opens one AI session on Monday morning and just... keeps it. By Wednesday, that single thread has fixed a login bug, built a settings toggle, refactored a hook, debated three different state-management approaches (settling on one, reverting to another, settling again), and pasted in something like twenty files along the way. On Wednesday afternoon, a straightforward "add a loading spinner here" request comes back with a spinner that doesn't match the design system, ignores a convention that was clearly established Monday, and vaguely references a component that got renamed on Tuesday. The engineer's first instinct is to blame the model. The actual cause is upstream of the model entirely: nobody ever decided when this session should have ended. Module 1 explained *why* that happens — context rot, recency bias, accumulated noise. This module is about what to actually *do* about it: the concrete habits of starting, splitting, and ending sessions on purpose instead of by accident.

## Objective

Turn "when should I start a new session" from a vague feeling into a decision you make deliberately, using the mechanics from Module 1 as your reasoning — plus the handful of concrete tools (clearing, compacting, restating constraints) that let you extend a session's useful life when starting over isn't the right call.

---

## 2.1 When to Start Fresh vs. Keep Going

The instinct to treat one long-running session as more "convenient" than several short ones gets the trade-off backwards. A fresh session costs you a few seconds of re-establishing context; a stale session costs you silently degraded output that you may not notice until it's already in a PR. The question isn't "am I in the middle of something" — it's "does this next request belong to the same task as what's currently in the window."

Start a new session when:

* **The task category changes**, not just the file. Moving from "debug this race condition" to "build this new form" is a new task even if it touches the same component — carrying the debugging back-and-forth into the build task adds noise without adding signal (Module 5's context pollution, arriving gradually instead of all at once).
* **You're about to paste in a materially different set of files.** If the next request's minimum sufficient context (Module 5) barely overlaps with what's already in the window, you're not extending the session — you're diluting it.
* **A prior approach was tried and abandoned.** If you spent ten turns exploring an approach that turned out wrong, that entire exploration is still sitting in the window, and it can bias the model back toward the abandoned approach through simple proximity. A clean session starting from the *decision*, not the exploration, is often sharper than continuing in place.
* **The session has been running long enough that you can't confidently recall everything you've told it.** If you can't, the model's ability to weight it correctly (Module 1.3) is even shakier than yours.

Keep going in the same session when:

* You're iterating on the *same* piece of work — reviewing a diff, asking for a revision, working through the steps of a plan you already approved (Module 8). This is exactly the case a session is good at: the shared context is still relevant on every turn.
* You're in the middle of a multi-step implementation (Module 8's incremental implementation) where each step genuinely depends on the last one being visible.

**Worked example.** You've spent thirty minutes debugging why a modal doesn't close after a mutation (Module 10's hypothesis-driven loop), and you've finally confirmed the cause and shipped the fix. The next request is "now build the notification preferences tab." Even though it's the same feature area and even the same file directory, this is a fresh-session moment: the debugging trail (three ruled-out hypotheses, two rejected fixes, several rounds of "try this instead") is now pure noise for a build task that shares almost no actual context need with it. Starting clean and pasting in only what the new task needs — the settings page pattern, the relevant contract — will outperform continuing in the debugging-saturated thread.

---

## 2.2 Compacting and Clearing

Most AI coding tools give you at least two levers short of a full restart, and knowing what each one actually does (versus what it feels like it does) matters.

**Clearing** empties the context window outright — the equivalent of Module 1.1's "blank slate." Nothing carries over: not files, not decisions, not corrections. This is the right move when the *next* task shares essentially nothing with the current session's content — the "task category changes" case from 2.1.

**Compacting** (where available) asks the model to summarize the conversation so far and continue from that summary instead of the full history, freeing up token budget (Module 1.2) while trying to preserve the gist. This is not equivalent to a human skimming their own notes — it's lossy in a specific way: broad decisions tend to survive ("we decided state should be URL-owned"), but specific details often don't ("the exact reason we rejected the `useReducer` approach was X, evidenced by Y"). Compacting is a reasonable move mid-task when you're running long on *one* thing and want to keep going without losing the thread entirely — it is a poor substitute for actually finishing and starting fresh when the task itself has changed.

The practical rule: **compact to continue the same task with more room; clear (or start a new session) to begin a different one.** Using compaction to paper over a session that's actually drifted across five unrelated tasks just produces a shorter version of the same undifferentiated noise from Module 1.4 — it doesn't restore focus, it just makes the noise cheaper to keep around.

> **In Claude Code, specifically.** `/clear` is the literal blank-slate command. `/compact` is the literal compaction command, and it takes optional instructions — `/compact focus on the API changes` — to steer what survives the summary instead of leaving it to a generic pass. `Esc` interrupts Claude mid-action without losing context, so you can redirect immediately instead of waiting out a wrong turn. Double-tap `Esc` (or `/rewind`) opens a checkpoint menu that can restore the conversation, the code, or both to any earlier point — including "summarize from here" or "summarize up to here," a more targeted version of compacting that only condenses part of the conversation. Anthropic's own guidance is blunt about the failure mode this section is warning about: *"if you've corrected Claude more than twice on the same issue in one session, the context is cluttered with failed approaches. Run `/clear` and start fresh."* See the [official sessions guide](https://code.claude.com/docs/en/sessions#manage-context-within-a-session) for the full command reference.

**Worked example.** You're forty minutes into implementing the checkout flow from Module 4's worked example, following its six-task breakdown one step at a time. You're on task 4 of 6, the window is getting full, and everything in it is still relevant — the contract, the component structure decided in tasks 1–2, the reasoning behind a validation edge case from task 3. This is exactly when compacting earns its keep: you're not changing tasks, you're running long on one coherent task and want the historical decisions preserved in condensed form rather than dropped. Contrast this with reaching for compaction after task 6 is shipped and tested, and you're about to start an unrelated bug fix — at that point, compacting just carries forward a summary of work that's now irrelevant; clearing is the right move.

---

## 2.3 Splitting Work Across Sessions by Task, Not by Time

A common but unhelpful habit is treating session boundaries as roughly "one per day" or "one per sitting." The unit that actually matters is the task, not the clock. Module 4's task decomposition already teaches you to break a feature into small, independently reviewable steps (contract → form → query → table → pagination → tests); session boundaries should track that same decomposition, not fight it.

This doesn't mean *every* task needs its own session — a tightly-linked sequence (Module 4's checkout example, Module 8's incremental implementation) is meant to stay in one session precisely because each step depends on the last one being visible. It means the boundary should be a deliberate match to the *shape* of the work, not an accident of "well, I was already in this window." Two practical patterns:

* **One session per feature slice**, when the slice's steps are tightly sequential and each depends on the previous step's output (a single component's build-out, one bug's full debug-and-fix loop).
* **One session per independent task**, when Module 4.6's Divide & Conquer applies — several decomposed pieces that don't actually depend on each other's implementation details are exactly the case for running them in separate sessions (or separate parallel agents), rather than sequentially bloating one window with all of them.

**Worked example.** The checkout feature's six tasks (contract, form, query, table, pagination, tests) are sequential and interdependent — one session, run through step by step, is correct. Compare that to a sprint with three unrelated tickets: fix a flaky test, add a tooltip to an existing button, and update a README. These share no context need with each other at all. Running all three through one session because it's "the same afternoon" is the same mistake as 2.1's task-category case — three fresh, minimal sessions (or three parallel sub-agents, Module 4.6) will each outperform one session asked to juggle all three.

> **In Claude Code, specifically.** Sessions are saved automatically, so splitting work across sessions doesn't cost you the ability to return to any of them: `claude --continue` resumes the most recent session in the current directory, `claude --resume` opens a picker across every saved session, and `/resume` does the same from inside a running session. Name a session with `claude -n <name>` at startup or `/rename` mid-session (e.g. `checkout-flow`, `flaky-test-fix`) so the split is findable later instead of a wall of untitled transcripts. For genuinely parallel work — two independent branches of the dependency graph, run at the same time instead of one after another — `claude --worktree <name>` gives each session its own git worktree so simultaneous edits can't collide; see Module 4.6 and the [Dividing Work with Sub-Agents playbook](../05-playbooks/07-dividing-work-with-subagents.md) for when that's the right call.

---

## 2.4 Restating Load-Bearing Constraints

Module 1.3 established that recency bias is real: something said once, early, loses influence as a session goes on — not because it's erased, but because it's no longer near the front of the model's attention. The habit that counteracts this is simple and costs almost nothing: **restate a load-bearing constraint at the point where it matters, instead of trusting that it's still "in mind."**

This isn't about repeating everything — that would just add more noise. It's about identifying the handful of constraints in a session that, if silently dropped, would produce a wrong or unacceptable result, and re-asserting exactly those at the moment of highest risk (right before a new implementation step, right before a refactor, right after switching tasks within the same session).

**Worked example.** Early in a long session you establish: "this codebase has zero tolerance for `any` — every PR with one gets bounced in review." Two hours and several tasks later, you're about to ask for a fairly mechanical data-transformation utility — exactly the kind of task where a generic AI instinct might reach for `any` to sidestep a messy union type. Rather than assuming the hour-old constraint is still fully weighted, restate it in the same message: "Implement this transform. No `any` — if the union is awkward, use a type guard." That one line costs a sentence and removes the single most likely failure mode for this specific request, instead of hoping recency bias doesn't bite you this time.

---

## Key Takeaways

* Session boundaries should track the *task*, not the clock or your convenience. A category change, a materially different context need, or an abandoned approach still lingering in the window are all signals to start fresh.
* Clearing and compacting are different tools: compact to keep going on the *same* task with more room; clear (or restart) when the next task doesn't share the current session's context.
* Split multi-task work along the same lines Module 4 already teaches for decomposition — sequential, dependent steps stay in one session; independent pieces belong in separate sessions or parallel agents (Module 4.6), not stacked into one.
* Recency bias (Module 1.3) means constraints fade with distance, not deletion. Restate the handful that are actually load-bearing at the point of highest risk — don't rely on "I said this already."
* None of this is about session length in the abstract. A long session on one coherent task can stay sharp the whole way through; a short session juggling three unrelated things can go stale fast.

## Try It Yourself

1. Look back at your last three AI sessions. For each one, identify every distinct *task category* it covered (not files — tasks). If any session covered more than one unrelated category, that's a session that should have been split — note where the split point should have been.
2. In your next long implementation session, deliberately restate one load-bearing constraint (a "never do X" or "always do Y" rule) right before the step where it's most at risk of being dropped, instead of assuming it's still in force. Compare the result to a recent case where you didn't do this and the model dropped a constraint from earlier in the same session.

---

*[← Previous: How AI Processes Your Request](./02-how-ai-works.md) | [Next: Choosing Your Model & Effort →](./04-choosing-model-and-effort.md)*
