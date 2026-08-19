# Playbook: Updating Existing Code

*[← Previous: Debugging a Bug](./03-debugging-a-bug.md) | [Next: From Docs to Code →](./05-from-docs-to-code.md)*

## Situation

You're changing code that already works, for a reason other than a bug — a small feature tweak, a behavior change requested by product, an added edge case. This is distinct from [Starting a New Feature](./01-starting-a-new-feature.md) (nothing exists yet) and from [Migrating Code](./02-migrating-code.md) (a wholesale pattern swap). Here, most of the code and its structure are correct and should stay that way — you're changing one thing, not rebuilding the surrounding area.

## Steps

1. **Ask AI to find the existing pattern or convention for this kind of change first.** Before generating anything, send it looking: "Find the pattern used by this repository for handling [this kind of case]. Do not create a new abstraction." This is Module 9's discovery step, applied at the moment of a small update instead of a new build. See [AI as a Coding Partner](../03-workflow/02-ai-coding-partner.md) §9.4.
2. **Define the smallest diff that satisfies the change.** Once you know what the existing pattern looks like, scope the update to the minimum change that fits inside it — not a broader rewrite of the surrounding code "while you're in there."
3. **Explicitly scope out unrelated cleanup.** State "do not refactor unrelated code" as a constraint in the same prompt as the change itself — don't leave it implied.

## Quick example

Requirement: "Add a 'suspended' status to the user table's status filter, in addition to active/inactive." Instead of jumping to the filter component:

```text
Find the pattern used by this repository for adding a new
value to an existing status enum used in filters (see
UserStatus in the user list).

Check where UserStatus is defined, where the filter UI reads
its options from, and whether adding a value requires touching
more than one place.

Do not create a new abstraction — report what exists.
```

A useful answer names the single source of truth — say, `UserStatus` is defined once and the filter dropdown maps over its values automatically — which turns the actual change into a one-line addition to the enum, not a new dropdown component. The follow-up prompt then stays scoped:

```text
Add "suspended" to the UserStatus enum and its two existing
label maps (UserStatusLabel, UserStatusBadgeColor).

Do not refactor the filter component. Do not touch any other
status-related file.
```

## Watch-outs

* **An update request quietly turning into a drive-by refactor.** AI notices something else nearby it doesn't like — a naming inconsistency, a component it thinks is too long — and "improves" it alongside the actual change you asked for. Now your diff contains two unrelated changes bundled together, and the one you actually meant to make is harder to review because it's buried in the one you didn't ask for.
* **Two inconsistent patterns for the same problem.** If you skip step 1 and let AI generate a fresh implementation instead of finding the existing one, you can end up with a second, slightly different way of doing something the codebase already does consistently — and now every future engineer has to guess which one is "the" way.
* **If a bigger cleanup really is warranted,** don't let it ride along inside this update. Split it into its own explicit task and run it through [AI Refactoring](../03-workflow/04-ai-refactoring.md)'s Analyze → Compare → Refactor sequence, reviewed on its own terms.

## Related reading

* [AI as a Coding Partner](../03-workflow/02-ai-coding-partner.md) §9.4 "Discover Existing Patterns" — the full reasoning for asking AI to find conventions before generating.
* [AI Refactoring](../03-workflow/04-ai-refactoring.md) — for when a bigger cleanup really is warranted, as its own explicit task.

## Try it

Next time you're about to update a small piece of working code, open with a discovery prompt — "find the existing pattern for this, do not create a new abstraction" — before asking for the change itself, and add "do not refactor unrelated code" to the actual update prompt. Compare the diff you get to what a plain "make this change" would likely have touched.

---

*[← Previous: Debugging a Bug](./03-debugging-a-bug.md) | [Next: From Docs to Code →](./05-from-docs-to-code.md)*
