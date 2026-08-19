# Playbook: From Docs to Code

*[← Previous: Updating Existing Code](./04-updating-existing-code.md) | [Next: Using a Skill →](./06-using-a-skill.md)*

## Situation

You're starting from a spec, RFC, design doc, or ticket description rather than a live conversation — a written artifact is the source of truth for what needs to be built. Unlike [Starting a New Feature](./01-starting-a-new-feature.md), where you build the contract from a requirement sentence, here the contract is supposed to already be written down. The question is whether it actually is, and how completely.

## Steps

1. **Treat the doc as the contract.** The same way Module 4 treats a requirement sentence as something to convert into types before coding, treat the spec as the artifact you implement against — not as inspiration for a contract you derive yourself.
2. **Ask AI to extract the contract and explicitly list what's ambiguous or underspecified, before generating any code.** A doc written for humans routinely leaves gaps that read as obvious in prose and are not obvious as a type — optional vs. required fields, what happens on an empty list, what a status enum's full set of values actually is. Surfacing these as an explicit list is the whole point of this step; skip it and AI fills the gaps silently instead of flagging them.
3. **Resolve ambiguity against the doc owner, not by letting AI guess.** For each item on that list, get an answer from whoever owns the spec before coding against it. A plausible-sounding default is still a guess, and it's the kind of guess that only surfaces as a bug once someone notices the implementation doesn't match what the doc's author actually meant.
4. **Keep a link back from the implementation to the doc.** A comment, a PR description line, a reference in the contract file — something that lets a future reader (or a future debugging session) catch drift between what's implemented and what the doc says, before it becomes a mystery.

## Quick example

Given an RFC that says "users can bulk-archive orders from the list view," extract the contract and the gaps before writing anything:

```text
Extract the implementation contract from this RFC.

RFC: [paste the relevant section]

Report:
- the request/response shape implied by the RFC
- anything ambiguous or left unspecified (e.g. is there a
  limit on how many orders can be archived at once? what
  happens if one of the selected orders was already archived
  by someone else? is this reversible?)

Do not generate code.
```

A useful answer surfaces exactly the kind of gap an RFC usually leaves open — no stated limit on batch size, no stated behavior for a stale selection, no mention of whether archiving is reversible. Those three questions go to the RFC's author before any implementation starts, not into a set of assumptions AI picks on its own.

## Watch-outs

* **A stale or wrong doc is worse than no doc.** A missing spec at least signals "nobody has decided this yet." A wrong one actively misleads, with the appearance of authority.
* **If implementation reveals the doc itself was incorrect, that's the brownfield case from the debugging playbook** — see [Debugging a Bug](./03-debugging-a-bug.md) and [AI Debugging §10.6](../03-workflow/03-ai-debugging.md). Fix the doc first, then implement against the corrected version. Don't just quietly diverge from what's written and let the two disagree.

## Related reading

* [Problem Decomposition](../02-thinking-tools/01-problem-decomposition.md) — the contract-first discipline this playbook applies to a written spec instead of a requirement sentence.
* [AI Debugging](../03-workflow/03-ai-debugging.md) §10.6 — the brownfield/greenfield ordering for when reality and an existing doc disagree, which is exactly the situation a wrong spec creates once you start implementing against it.

## Try it

Next time you start from a spec or ticket, don't open your editor first — ask AI to extract the contract and list every ambiguity it finds in the doc. Take that list to whoever owns the spec before writing any code, and note how many of those ambiguities you would have otherwise resolved yourself without realizing you were guessing.

---

*[← Previous: Updating Existing Code](./04-updating-existing-code.md) | [Next: Using a Skill →](./06-using-a-skill.md)*
