# Playbook: Managing a Session

*[← Previous: Dividing Work with Sub-Agents](./07-dividing-work-with-subagents.md) | [Next: Prompt Quick Reference →](./09-prompt-quick-reference.md)*

## Situation

You're mid-session, and it's time to decide whether to keep going, compact, or clear. This is the recipe version of [Controlling Your Session](../01-foundations/03-controlling-your-session.md) — read it in full for the mechanics and worked examples behind each move below.

## Steps

1. **Start fresh** when the task category changes, you're about to paste in a materially different set of files, or you can no longer confidently recall everything you've told the session so far.
2. **Continue** when you're iterating on the same piece of work, or mid-way through a multi-step implementation where each step depends on the last one being visible.
3. **Compact** to keep going on the *same* task with more room. It's lossy — it keeps the gist, not the specifics — so use it when you're running long on one coherent task, not as a way to keep an already-drifted session alive.
4. **Clear** when the next task shares nothing with the current one. This is the equivalent of a blank slate — nothing carries over, which is exactly what you want once the current session's content is no longer relevant.
5. **Before any risky or high-stakes step, restate the one or two load-bearing constraints that would be expensive to lose.** Recency bias means something said once, early in a long session, loses influence even though it hasn't been erased — re-asserting it right before the step where it matters costs a sentence and closes off the most likely failure mode.

## Quick example

You've been in one session for an hour: debugged a race condition in a mutation, shipped the fix, and now the next request is "build the notification preferences tab." Same feature area, same directory — but a different task category. That's a clear-and-restart moment, not a continue-or-compact one: the debugging trail (ruled-out theories, rejected fixes) is noise for a build task.

Compare that to being forty minutes into a six-step checkout implementation, on step 4 of 6, with the window filling up but everything in it still relevant — the contract, the component decisions from steps 1–2, the edge case reasoning from step 3. That's when compacting earns its keep: same task, more room, not a different task in disguise.

Right before step 4's implementation prompt, if step 2 established "no `any` in this codebase, ever," restate it in that prompt rather than assuming it's still weighted after two steps of unrelated detail:

```text
Implement step 4: the mark-as-read mutation.

No `any` — if the response shape is a union, use a type guard.
```

## Watch-outs

* **Using compaction to paper over a session that's actually drifted across unrelated tasks.** Compacting a session that's already juggling three different things doesn't restore focus — it just produces a shorter version of the same undifferentiated noise, making stale context cheaper to keep around instead of getting rid of it. If the session has drifted across task categories, the right move is clearing (or starting fresh), not compacting.

## Related reading

* [Controlling Your Session](../01-foundations/03-controlling-your-session.md) — the full reasoning behind start-fresh vs. continue, compacting vs. clearing, splitting work by task not time, and restating load-bearing constraints.
* [How AI Processes Your Request](../01-foundations/02-how-ai-works.md) — *why* this matters: context rot and recency bias, the mechanics underneath every decision in this playbook.

## Try it

Look at the session you have open right now. Name its current task category out loud, and check whether anything still in the window belongs to a different one. If it does, that's your signal for whether to compact, clear, or keep going as-is.

---

*[← Previous: Dividing Work with Sub-Agents](./07-dividing-work-with-subagents.md) | [Next: Prompt Quick Reference →](./09-prompt-quick-reference.md)*
