# Playbook: Starting a New Feature

*[← Previous: Playbooks Overview](./README.md) | [Next: Migrating Code →](./02-migrating-code.md)*

## Situation

A new requirement lands — a ticket, a sentence from product, a Slack message — and nothing for it exists in the codebase yet. There's no prior pattern to match, no existing behavior to preserve, no doc to reconcile against. You're about to open your AI tool and you're deciding what to give it first.

## Steps

1. **Turn the requirement into a contract before anything else.** A ticket is a sentence; a contract is a type. Write down the shapes involved — the request, the response, the state — the way Module 4 walks through for a filter or a checkout flow. Do this on paper or in a scratch file before you type a single prompt. See [Problem Decomposition](../02-thinking-tools/01-problem-decomposition.md).
2. **Check whether the contract already exists somewhere in the system.** Before committing to the shapes you just wrote, ask AI to search for an existing type, API response, or convention that already covers this. If one exists, hand it over instead of letting AI (or you) re-derive a slightly different version.
3. **Gather the minimum sufficient context, not the whole repo.** Once the contract is settled, identify the smallest set of reference files AI actually needs — one similar feature, the relevant hook pattern, the contract itself. See [Context Engineering](../02-thinking-tools/02-context-engineering.md).
4. **Run Analyze → Plan.** Ask AI to analyze the requirement against the existing codebase (relevant components, existing patterns, state ownership, data dependencies, edge cases) with "do not write code" as an explicit instruction, then ask for a plan (component structure, state ownership, data flow, files touched, risks).
5. **Human Review the plan before any implementation.** Check it against decisions AI can't see — your team's conventions, your actual state-ownership rules. This is a one-sentence correction now, or a rewrite later. See [Plan Before Code](../03-workflow/01-plan-before-code.md).
6. **Implement incrementally, one reviewable step at a time.** Scope each implement prompt to a single step from the plan, review the diff, then move to the next step. Don't ask for the whole feature in one shot.

## Quick example

Requirement: "Let users save a search as a favorite, with a name they choose." Before any prompt:

```ts
type SavedSearch = {
  id: string;
  name: string;
  query: UserFilter; // reuse the existing filter contract, don't redefine it
  createdAt: string;
};
```

Then the analyze prompt:

```text
Analyze this requirement and the existing code.

Requirement: users can save a search as a favorite with a
chosen name, using this contract:

type SavedSearch = { id, name, query: UserFilter, createdAt }

Identify relevant components, existing patterns for saved/
named resources, state ownership, and edge cases (duplicate
names, empty query, list ordering).

Do not write code.
```

Only after that analysis and the resulting plan are reviewed does step 1 of implementation start — the favorites list component, not the whole feature.

## Watch-outs

* **Coding before the contract exists.** The moment you skip straight to "build this," the contract gets decided implicitly, as a side effect of whatever shape AI's first draft happens to land on — and it's now buried inside working code instead of sitting in a paragraph you could have corrected for free.
* **Letting AI invent a contract that already exists elsewhere.** If `UserFilter` or `OrderSummary` (or whatever this requirement's equivalent is) is already defined somewhere in the system, AI re-deriving its own near-match version is worse than reusing the real one — it creates two slightly different shapes for the same concept, and nobody decided that on purpose.

## Related reading

* [Problem Decomposition](../02-thinking-tools/01-problem-decomposition.md) — the full Requirement → Contract → Data → State → Architecture sequence.
* [Context Engineering](../02-thinking-tools/02-context-engineering.md) — minimum sufficient context, and why more isn't safer.
* [Plan Before Code](../03-workflow/01-plan-before-code.md) — Analyze → Plan → Review → Implement in full, including why "do not write code" matters.
* [AI Communication Framework](../02-thinking-tools/04-ai-communication-framework.md) — Context → Goal → Constraints → References → Output, for structuring each prompt along the way.

## Try it

Take the next new-feature ticket in your queue and stop before opening your AI tool. Write the contract — just the types — on paper or in a scratch file first, then run Analyze → Plan against it and see whether the plan review catches anything you'd have otherwise found two steps into implementation.

---

*[← Previous: Playbooks Overview](./README.md) | [Next: Migrating Code →](./02-migrating-code.md)*
