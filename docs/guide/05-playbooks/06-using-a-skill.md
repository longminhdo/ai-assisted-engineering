# Playbook: Using a Skill

*[← Previous: From Docs to Code](./05-from-docs-to-code.md) | [Next: Dividing Work with Sub-Agents →](./07-dividing-work-with-subagents.md)*

## Situation

The team, or the tool itself, has packaged some recurring task as an invokable skill — a named, pre-written set of instructions you trigger on demand (a slash command, a saved prompt template) — rather than something you write from scratch in a prompt every time. You're facing a task and it's not immediately clear whether one of these already covers it.

## Steps

1. **Notice a skill exists and is relevant.** This is mostly a discovery habit: check whether the task in front of you matches something already named and packaged — by convention, teams tend to name skills after the recurring task they cover, so a task that feels routine ("write a PR description," "generate tests for this phase," "review this diff") is worth a quick check before you draft a prompt from scratch.
2. **Understand what invoking a skill actually buys you.** A skill front-loads instructions and conventions you'd otherwise have to restate every session — the exact payoff [Controlling Your Session](../01-foundations/03-controlling-your-session.md) describes for restating load-bearing constraints, except here the restating is done once, by whoever authored the skill, instead of by you every time. Invoking it is not "the same as prompting, but shorter" — it's retrieving a pre-agreed set of conventions instead of re-deriving them.
3. **Know when to fall back to plain prompting instead.** A skill is built for the shape of task its author had in mind. If your actual situation doesn't fit that shape — a genuinely different edge case, a constraint the skill doesn't account for — don't force the task into it just to avoid typing a prompt from scratch. Using a skill that's a poor match produces output confidently shaped like the wrong task.

## Quick example

You're about to write a from-scratch prompt asking AI to "review this PR for common issues." Before drafting it, check whether the team already has a packaged review skill — say, `/code-review` — and what it actually covers:

```text
Before running /code-review, skim what it checks for
(correctness bugs, reuse/simplification) and at what
effort level. If this PR's risk is mostly around a security-
sensitive auth change that the skill's description doesn't
mention, that's a signal to also ask a direct, scoped question
about that specific risk rather than relying on the skill alone.
```

Contrast that with reaching for the skill on a PR that's an unusual one-off — say, a generated-code dump from an external tool — where the skill's assumptions (a normal hand-written diff) don't fit, and a plain, situation-specific prompt will serve you better than forcing the mismatch.

## Watch-outs

* **Treating a skill as a black box.** A skill that writes a commit message is low-stakes to trust blindly. A skill that touches deployment config, security review, or generates code you're about to ship is not — read what it actually does before trusting its output on anything consequential, the same way you wouldn't merge a generated diff you hadn't looked at.

> **In Claude Code, specifically.** A skill is typically a `SKILL.md` file with a name and description in its frontmatter, invoked either by name — `/skill-name` — or automatically, when Claude Code matches your request against the description on its own. That's why step 1's discovery habit matters: a well-named skill you don't know exists is no different, in practice, than one that was never written.

## A note on scope

This playbook covers the *consumer* side of skills — using ones that already exist. Building and contributing new skills for the team is a different skill set, covered in the [Advanced Track](../07-advanced-track/README.md).

## Related reading

* [Controlling Your Session](../01-foundations/03-controlling-your-session.md) — the session-control payoff a skill gives you for free by front-loading conventions you'd otherwise restate.

## Try it

Next time you're about to draft a from-scratch prompt for a task, pause and check whether a skill already covers it. If one does, read through what it actually instructs before running it, and compare the result to what you'd have gotten from your own prompt.

---

*[← Previous: From Docs to Code](./05-from-docs-to-code.md) | [Next: Dividing Work with Sub-Agents →](./07-dividing-work-with-subagents.md)*
