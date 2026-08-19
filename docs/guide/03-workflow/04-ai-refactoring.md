# Module 11 — AI Refactoring

*[← Previous: AI Debugging](./03-ai-debugging.md) | [Next: AI Code Review →](./05-ai-code-review.md)*

## Why this matters

"Refactor this" is one of the most dangerous two-word prompts you can send. Without scope, AI treats refactoring as an invitation to rewrite: it renames exported props, changes a callback's signature, collapses two components that were deliberately separate, or "simplifies" a hook in a way that quietly drops a memoization your app depends on for a slow list. None of that shows up as a red squiggly line — it shows up two sprints later as a bug report from a consumer of that component who has no idea their code broke because a file they never touched got "cleaned up." Compare that to a scoped request that first inventories the actual problems, lets you pick a direction, and then implements it under an explicit constraint to preserve behavior and the public API — the diff is small, reviewable, and doesn't become someone else's incident.

## Objective

Use AI to improve existing code, not just generate new code. Refactoring is a distinct skill from building: the goal is a smaller, cleaner diff that changes *how* the code works without changing *what* it does for anyone calling it.

---

## 11.1 Analyze Before Refactoring

The instinct to say "clean this up" and let AI figure out what "clean" means is the same instinct Module 8 warned you about with "build X" — you're skipping the step where problems get named before solutions get proposed. A component can be bad in several unrelated ways at once (duplicated logic, unnecessary state, poor boundaries), and if you don't ask AI to separate them, it will pick whichever one is easiest to fix first and rewrite around that, possibly making a different problem worse. Asking for a diagnosis with "do not refactor yet" as an explicit instruction keeps AI from jumping to code the moment it spots something it doesn't like — the same anchoring problem you already saw with "do not write code" in the Analyze step of Module 8.

The curriculum's prompt:

```text
Analyze this component.

Identify:
- duplicated logic
- unnecessary state
- unnecessary renders
- coupling
- poor component boundaries
- difficult-to-test code

Do not refactor yet.
```

What makes this list useful is that each item points at a different root cause, and mixing them up leads to the wrong fix. "Unnecessary state" and "unnecessary renders" are related but not the same thing — you can have state that's necessary but poorly scoped, causing a parent to re-render every child on every keystroke. "Coupling" and "poor component boundaries" are also related but distinct: two components can be loosely coupled and still have the wrong boundary (a `UserAvatar` that reaches into `OrderHistory`'s props because nobody drew the line in the right place). Naming these separately means whatever comes back is a list of specific findings, not a vague "this could be cleaner" the way an unscoped prompt tends to produce.

**Worked example — a checkout flow.** Say you own `CheckoutSummary`, a component that has grown over a year of quick fixes: it renders the cart line items, computes tax and shipping, shows a promo-code input, and disables the "Place Order" button while a payment intent is being created. Nobody has looked at it end to end in months, and every new discount rule gets bolted onto whatever `if` chain already exists. Before touching anything, you run:

```text
Analyze this component.

Component: CheckoutSummary.tsx

Identify:
- duplicated logic
- unnecessary state
- unnecessary renders
- coupling
- poor component boundaries
- difficult-to-test code

Do not refactor yet.
```

A useful answer comes back naming concrete things: tax calculation is duplicated in both `CheckoutSummary` and `OrderConfirmation` with a subtle rounding difference between the two copies; `promoCode`, `promoError`, and `isApplyingPromo` are three separate `useState` calls that are only ever set together and could be one reducer; the component re-renders the entire line-item list on every keystroke in the promo input because the input's state lives in the parent instead of being isolated; and the "Place Order" button's disabled logic is coupled to five different booleans (`isApplyingPromo`, `isCreatingIntent`, `hasValidAddress`, `cartIsEmpty`, `isSubmitting`) scattered across the file, which is also why nobody can write a clean unit test for "when is the button disabled" without mocking all five. That's a diagnosis you can act on — and notice that none of it required AI to guess; it's all observable in the existing code, which is exactly why it belongs in this step rather than folded into a rewrite request.

---

## 11.2 Compare Approaches

A diagnosis tells you what's wrong; it doesn't tell you which fix is worth making. Every non-trivial refactor has more than one reasonable shape, and the "right" one depends on trade-offs only you can weigh — how much this code is expected to change in the next quarter, how much risk the team can absorb this week, whether performance actually matters here or is a hypothetical concern. Asking AI to write the refactor before those trade-offs are on the table means it picks for you, silently, based on whatever it judged "best practice" in isolation — and best practice in isolation is not the same as best practice for your deadline, your team's familiarity with the pattern, or your test coverage.

```text
Propose two refactoring approaches.

For each:
- complexity
- maintainability
- performance
- risk
```

The engineer chooses the approach.

That last line is not filler — it is the actual point of this step. Asking for two approaches and stopping there, without a human picking one, just delays the same problem: AI still ends up choosing, only now it's choosing between two options it also generated, which isn't meaningfully different from letting it choose unprompted. The value comes from a human weighing complexity against risk against the calendar, using context AI doesn't have.

**Worked example, continued.** For `CheckoutSummary`, you take two of the findings — the duplicated tax logic and the five-boolean disabled state — and ask:

```text
Propose two refactoring approaches for CheckoutSummary's
tax calculation duplication and disabled-button logic.

For each:
- complexity
- maintainability
- performance
- risk
```

A useful comparison looks something like this:

```text
Approach 1: Extract shared calculateTax() utility,
consolidate button-disabled logic into a single
useCheckoutReadiness() hook returning { canSubmit, reason }.

  Complexity: low — pure extraction, no new abstractions.
  Maintainability: high — one place to fix tax bugs,
    one hook to test disabled logic against.
  Performance: neutral, no change to render behavior.
  Risk: low — behavior-preserving by construction.

Approach 2: Introduce a CheckoutContext that owns cart,
tax, promo, and submission state as a single reducer,
consumed by CheckoutSummary and OrderConfirmation alike.

  Complexity: medium-high — new context, new reducer,
    both consumers need to migrate off local state.
  Maintainability: high long-term — single source of
    truth eliminates the class of bug that caused the
    tax duplication in the first place.
  Performance: requires care — a naive context will
    re-render both consumers on every field change;
    needs selector-based reads or context splitting.
  Risk: medium — touches two components' state model,
    not just one function.
```

Approach 1 is the safer pick if you need this shipped this week and the duplication is the only symptom you actually care about right now. Approach 2 is the better pick if this is the third time this quarter a tax or promo bug has come from these two components drifting apart, and you're willing to spend a sprint fixing the root cause instead of the symptom. Neither answer is generically "correct" — that's a call about your codebase's trajectory, not a technical fact AI can determine on its own, which is exactly why this step exists as a checkpoint rather than getting folded into the first prompt.

---

## 11.3 Refactor Safely

Choosing an approach is not the same as giving AI a blank check to implement it however it sees fit. Left unconstrained, an implementation pass will often "improve" things you didn't ask about along the way — renaming a prop because the new name reads better, adding a dependency because it made a utility function nicer to write, touching a neighboring file because it was already open in context. Every one of those is a plausible, well-intentioned change, and every one of them turns a refactor that should be behavior-preserving into one that isn't, without anyone deciding that on purpose. The fix is to state the constraints as explicitly as you stated the requirements in Module 7 — preserving behavior and the public API aren't defaults you can assume AI will infer; they're instructions you have to give.

```text
Implement approach 1.

Constraints:
- Preserve behavior.
- Do not change the public API.
- Do not introduce dependencies.
- Keep the diff minimal.
```

Each constraint here is closing a specific failure mode, not just adding boilerplate caution. "Preserve behavior" blocks the drive-by bug fix that sneaks into a refactor diff and now can't be reviewed as a bug fix on its own. "Do not change the public API" blocks the renamed prop or reshaped return value that breaks every caller silently. "Do not introduce dependencies" blocks solving an internal problem by reaching for a library, which trades a maintenance cost you didn't agree to for a convenience you didn't ask for. "Keep the diff minimal" blocks the unrelated tidy-ups — reordering imports, reformatting a file you're not even fixing — that make the actual change harder to find in review.

**Worked example, continued.** Having picked Approach 1 for `CheckoutSummary` — the lower-risk option, since you don't have a sprint to spend on Approach 2 this week — the implement prompt looks like:

```text
Implement approach 1: extract calculateTax() and
useCheckoutReadiness().

Constraints:
- Preserve behavior. CheckoutSummary and OrderConfirmation
  must compute identical totals for all existing test fixtures.
- Do not change the public API. CheckoutSummary's props
  and OrderConfirmation's props must not change.
- Do not introduce dependencies.
- Keep the diff minimal — do not touch promo-code UI
  or line-item rendering, they are out of scope.
```

Notice the constraints are specific to this component, not just the generic template — "identical totals for all existing test fixtures" is a concrete, checkable bar, and naming the two files that must keep their props stable removes any ambiguity about what "public API" means here. Because the tax bug was a *rounding difference* between two copies of similar logic, "preserve behavior" is actually doing something interesting: it should preserve the *existing, still-buggy* behavior of each caller individually, while the extraction is what makes the two calculations consistent with each other going forward. If you wanted the rounding itself fixed, that's a separate, explicit instruction — mixing "extract this" with "and also fix the rounding while you're in there" is exactly the kind of scope creep this step exists to prevent. Run one change, review it, then decide if the rounding fix is worth its own pass.

Once the diff comes back, review it the way you'd review any other change: does it actually touch only what was asked, do the existing tests for `CheckoutSummary` and `OrderConfirmation` still pass unmodified, and is the "minimal diff" claim actually true or did a stray formatting pass sneak in. A refactor prompt with constraints doesn't guarantee a constrained result — it just gives you clear grounds to reject the diff when it isn't.

---

## Key Takeaways

* Never send a bare "refactor this." Run Analyze → Compare → Refactor as three separate steps, the same discipline Module 8 taught for building new code.
* Analyze with "do not refactor yet," and ask for specific categories (duplication, unnecessary state, unnecessary renders, coupling, boundaries, testability) — a vague "what's wrong with this" produces a vague answer.
* Always ask for at least two approaches with trade-offs, and make the choice yourself. Two AI-generated options with no human decision in between is just a slower version of letting AI decide unprompted.
* State refactor constraints explicitly — preserve behavior, don't change the public API, don't add dependencies, keep the diff minimal — because none of these are defaults AI will infer on its own.
* "Preserve behavior" can mean preserving existing bugs on purpose. If you want a bug fixed too, say so as a separate, explicit instruction rather than letting it ride along inside the refactor.
* Review the resulting diff against the constraints you gave, not just against whether the code looks cleaner — a clean-looking diff that touched the public API anyway is still a rejected diff.

## Try It Yourself

1. Pick a component in your own codebase that you've been meaning to clean up — the one everyone avoids touching. Run the 11.1 analysis prompt against it and write down which of the six categories (duplication, unnecessary state, unnecessary renders, coupling, boundaries, testability) actually apply. Don't refactor yet — just see whether the diagnosis matches what you already suspected, or surfaces something you'd missed.
2. Take one finding from that analysis and run it through all three steps end to end: get two approaches, pick one and write down why in one sentence, then implement it with explicit preserve-behavior and minimal-diff constraints. Check the resulting diff against your existing tests before you decide whether the constraints actually held.

---

*[← Previous: AI Debugging](./03-ai-debugging.md) | [Next: AI Code Review →](./05-ai-code-review.md)*
