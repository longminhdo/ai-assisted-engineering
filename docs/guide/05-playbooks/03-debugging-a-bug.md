# Playbook: Debugging a Bug

*[← Previous: Migrating Code](./02-migrating-code.md) | [Next: Updating Existing Code →](./04-updating-existing-code.md)*

## Situation

Something is broken and you're about to ask AI about it. This playbook is intentionally thin — the real depth lives in [AI Debugging](../03-workflow/03-ai-debugging.md), and if any step below feels too compressed to act on, that's where the reasoning and worked examples are.

## Steps

1. **Give full debugging context, not a one-liner.** Expected behavior, actual behavior, error, stack trace, relevant code, recent changes, environment, and what's already been tried. Skip "why doesn't this work?" — it has no falsifiable claim in it, so AI has nothing to reason against except guesses.
2. **Ask for hypotheses with evidence and a way to verify each, not an immediate patch.** "List the top 3 possible causes. For each: explain why, identify evidence, suggest how to verify it." Add "do not modify code yet."
3. **Confirm the cause yourself before accepting a fix.** A plausible-sounding hypothesis is not a confirmed one — check it with a log line, a breakpoint, or a quick reproduction before anything gets patched.
4. **Write the short root-cause note.** What was actually happening, what assumption was wrong, why the fix addresses the cause and not the symptom, and what else in the codebase might share the same wrong assumption. See [AI Debugging §10.5](../03-workflow/03-ai-debugging.md).
5. **Run the brownfield/greenfield check.** Does an existing doc, comment, or convention make a claim your bug just disproved? If yes, fix that documented understanding first, then implement against the corrected version. If there's no prior claim to correct, implement, verify, then document what turned out to be true. See [AI Debugging §10.6](../03-workflow/03-ai-debugging.md).

## Quick example

Instead of:

```text
The date picker shows the wrong date sometimes, can you fix it?
```

send:

```text
Expected:
Selecting a date in the picker shows that same date in the
input field.

Actual:
For dates near a DST transition, the input field sometimes
shows the day before.

Error:
None thrown.

Relevant code:
DatePicker.tsx, formatSelectedDate() in dateUtils.ts

Recent changes:
None in this area recently.

Environment:
Reproduces locally, only for dates in late March / early
November.

What's been tried:
Confirmed the picker's internal selection state is correct;
the mismatch appears when formatting for display.

Analyze possible causes. Do not modify code yet.
```

That's enough for AI to reason about timezone conversion during formatting, instead of guessing at the picker's UI logic — which is exactly the kind of misdirected first attempt a vague report invites.

## Watch-outs

* **The "error → AI randomly changes code → maybe it works" loop.** Each iteration in this loop is independent of the last — nothing ties attempt #2 to evidence about why attempt #1 didn't work — so it can feel productive while never actually converging on the real cause. If you notice you're three "fixes" in with no new fact learned about *why* the bug happens, you're in this loop; stop and go back to step 2.

## Related reading

* [AI Debugging](../03-workflow/03-ai-debugging.md) — the full module this playbook summarizes: the debugging context checklist, hypothesis-driven debugging, the debugging loop, root-cause notes, and the brownfield/greenfield ordering. Read this directly if any step above needs more than a one-line reminder — this playbook is a fast-reference summary of it, not a substitute for it.

## Try it

Take a bug you're currently facing and run it through steps 1–3 above before touching any code: write the full context, get three hypotheses with evidence, and verify one yourself. Notice how much of the "fix" is now obvious once the cause is actually confirmed, versus how it would have gone if you'd opened with "why doesn't this work?"

---

*[← Previous: Migrating Code](./02-migrating-code.md) | [Next: Updating Existing Code →](./04-updating-existing-code.md)*
