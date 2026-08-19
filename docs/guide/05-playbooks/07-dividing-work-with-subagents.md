# Playbook: Dividing Work with Sub-Agents

*[← Previous: Using a Skill](./06-using-a-skill.md) | [Next: Managing a Session →](./08-managing-a-session.md)*

## Situation

You already have a decomposed task list — Module 4's output, not something you're deciding now — and you're figuring out which of those tasks can run in parallel instead of one after another in a single session. This is the recipe version of Module 4 §4.6's Divide & Conquer; read that section for the full reasoning if any step here feels too compressed.

## Steps

1. **Draw the dependency graph between tasks.** For every pair of tasks, ask: does one need the other's actual output to start? A task that produces a contract, a component, or a decision another task consumes is a real dependency. A task that could be hand-checked in isolation, knowing nothing about the other, is independent.
2. **Tasks with no edge between them are candidates for separate sessions or parallel sub-agents.** Each one gets scoped to its own minimum sufficient context (Module 5) — not the whole task list's worth of files stacked into one window, just what that one task actually needs.
3. **Tasks that share a mental model or build on each other stay sequential, in one session.** If holding a single evolving decision in your head across steps is what the task requires, splitting it doesn't help — it just duplicates the mental model across threads that now have to be reconciled.
4. **Always plan an explicit integration step, and review it as carefully as any other task.** Parallel work doesn't merge itself. Someone has to verify that what came out of two independently-run branches actually fits together.

## Quick example

Task list for a settings-page redesign: contracts (Task 1), then five independent tabs (Tasks 2–6, one per tab), then integration (Task 7). The dependency graph:

```text
Task 1 (shared tab-container contract)
   │
   ├──▶ Task 2 (Profile tab)     ─┐
   ├──▶ Task 3 (Notifications)   ─┤
   ├──▶ Task 4 (Security)        ─┼──▶ Task 7 (integrate + review)
   ├──▶ Task 5 (Billing)         ─┤
   └──▶ Task 6 (Connected apps)  ─┘
```

Task 1 has to run first and alone. Tasks 2–6 don't depend on each other at all — each is dispatched with only its own tab's contract and one reference tab pattern, not all five tabs' context stacked into one session. Task 7 is the integration step: check that all five tabs actually consume the Task 1 contract the same way, not just that each one renders correctly in isolation.

> **In Claude Code, specifically.** The literal phrasing for delegating a task is `"use a subagent to investigate/implement X"` — a subagent runs in its own context window and reports back a summary, so exploring five tabs' worth of files doesn't fill your main session's context with all five. For tasks you want running genuinely at the same time rather than one dispatched after another, `claude --worktree <name>` gives each one its own isolated git checkout so simultaneous edits can't collide — see the [official worktrees guide](https://code.claude.com/docs/en/worktrees). Before treating parallel work as done, a fresh subagent reviewing the integrated diff against the shared contract (not the session that built it) catches the disagreement this section warns about more reliably than self-review, because it isn't anchored to the reasoning that produced the code.

## Watch-outs

* **Two independently-built pieces silently disagreeing about a contract that was ambiguous in one direction.** This is the most common failure in parallel work — each branch resolves an underspecified part of the shared contract slightly differently, both diffs look correct on their own, and the mismatch only shows up at integration. Verify the two pieces actually agree with each other, not just that each one compiles alone.

## Related reading

* [Problem Decomposition](../02-thinking-tools/01-problem-decomposition.md) §4.6 "Divide & Conquer: Parallelizing with Sub-Agents" — the full worked examples, including the checkout-flow task graph and the five-tab settings-page case this playbook compresses.
* [Controlling Your Session](../01-foundations/03-controlling-your-session.md) §2.3 "Splitting Work Across Sessions by Task, Not by Time" — the session-boundary reasoning that mirrors this same dependency logic.

## Try it

Take your current task list for an in-progress feature and draw the dependency graph. Find one pair of tasks with no real edge between them, run them as two separate sessions or sub-agents instead of one sequential thread, and treat the point where their outputs meet as its own reviewed integration step.

---

*[← Previous: Using a Skill](./06-using-a-skill.md) | [Next: Managing a Session →](./08-managing-a-session.md)*
