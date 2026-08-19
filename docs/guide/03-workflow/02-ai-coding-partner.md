# Module 9 — AI as a Coding Partner

*[← Previous: Plan Before Code](./01-plan-before-code.md) | [Next: AI Debugging →](./03-ai-debugging.md)*

## Why this matters

Once a plan is approved, most of what you'll actually ask AI to do falls into a small number of repeating jobs: write new code, reshape code that already exists, explain code someone else wrote, or find out how the codebase already solves a problem you're about to solve again. Teams that treat all four as "just prompting" get inconsistent results — a "convert this to TypeScript" request that quietly drops a null check, an "explain this" answer that summarizes syntax instead of the actual risk, a new `useDebounce` hook added in a PR when one already existed two folders over. Each of these is a different kind of request with a different failure mode, and each has its own way of asking well.

## Objective

Use AI effectively during implementation — specifically for generating new code, transforming existing code, explaining unfamiliar code, and discovering patterns already established in your codebase, so that implementation moves fast without duplicating logic or losing precision.

---

## 9.1 Generate

### Why generation is AI's easiest job — and where that confidence turns into risk

Generation is what AI is best at and what most engineers reach for first, which is exactly why it's worth being deliberate about when it's the right tool. For self-contained, well-specified units of code, AI can produce a correct first draft faster than you can type it. The risk isn't that generation fails outright — it's that it succeeds *locally* while ignoring everything Module 8's Analyze step would have surfaced: an existing utility that does the same thing, a naming convention, a shape your other hooks already follow. Generation works best as the last step in the workflow, after analysis and planning have already told AI what "correct" looks like in your codebase — not as the first thing you reach for.

Good use cases for generation:

* Boilerplate
* Components
* Hooks
* Types
* Utility functions
* Tests

Each of these shares a property: the unit is small enough to review in one pass, and its correctness doesn't depend on context spread across five other files. The moment a "generate" request actually needs your existing error-handling convention, your existing query pattern, or your existing state-ownership rules to come out right, you're no longer doing pure generation — you're back in Module 8's Analyze → Plan territory, and skipping straight to generation is how you end up with the second Zustand store from Module 8's notifications example.

**Worked example — a settings page toggle group.** Say you're adding a "Notifications preferences" section to a settings page: three toggles (email, push, SMS), each with its own on/off state, all persisted through a single `updatePreferences` mutation. This is a good generation candidate because the shape is self-contained:

```text
Generate a PreferenceToggle component.

Props: label (string), checked (boolean), onChange (checked: boolean) => void, disabled (boolean, optional).
Use the existing Switch primitive from components/ui/Switch.tsx.
Include a data-testid derived from the label.
```

That's a clean generation request — one component, one job, dependencies named explicitly. Compare it to the version most engineers actually write first:

```text
Add toggles for notification preferences to the settings page.
```

The vague version forces AI to invent the component boundary, the prop shape, the styling primitive, and the persistence approach all at once — decisions that generation shouldn't be making on its own. The fix isn't a smarter prompt; it's recognizing that "toggles for the settings page" is actually three separate generate requests (the toggle component, the types for the preferences payload, the hook that wires toggles to the mutation), each scoped the way the `PreferenceToggle` example above is scoped.

---

## 9.2 Transform

### Why transformation is a different skill from generation

Transformation starts from working code, which changes the failure mode entirely: generation can be wrong by omission, but transformation can be wrong by *silent alteration* — behavior that worked before the transform and doesn't after, with a diff that still looks plausible. AI is genuinely strong at transformation because the input already encodes the intent; the job is translating that intent into a different shape, not inventing it. But "different shape" is exactly where subtle behavior changes hide — a JS-to-TS conversion that widens a type until a runtime check becomes dead code, a "convert to a hook" that drops a cleanup function because it wasn't obviously tied to the return value.

AI is particularly useful for transformations like:

```text
Convert this JavaScript function to TypeScript.
```

```text
Convert this component to a custom hook.
```

```text
Replace lodash usage with a native implementation.
```

```text
Migrate this component from API A to API B.
```

Each of these prompts is short because the source code is doing the specifying — but short doesn't mean unconstrained. The one thing worth adding to every transform prompt is what must stay identical: "preserve existing error handling," "keep the same return shape," "don't change the public props." Without that, AI will sometimes take a transform request as license to also "improve" things you didn't ask it to touch, and now your diff contains an intentional change buried inside a mechanical one.

**Worked example — a checkout flow.** Say your checkout page has an old `calculateOrderTotal.js` utility, written before the codebase adopted TypeScript, that computes subtotal, discount, tax, and shipping. It's used by both the checkout page and an admin refund tool, so a silent behavior change here has two blast radii, not one.

```text
Convert calculateOrderTotal.js to TypeScript.

Preserve existing behavior exactly, including the rounding
rule (round half up, not banker's rounding) and the order
of operations (discount before tax, not after).
Do not change the function's exported shape — it is
imported by both CheckoutSummary.tsx and RefundTool.tsx.
```

Naming the rounding rule and the operation order explicitly matters here precisely because those are the two details a generic "convert to TypeScript" pass is most likely to normalize away without telling you — TypeScript conversion often invites "cleanup" as a side effect, and a rounding change that's off by a cent will pass every type check while breaking real refunds.

---

## 9.3 Explain

### Why "what does this code do?" is the wrong question

Asking AI to understand unfamiliar code is one of the highest-leverage things you can do with it — reading someone else's component cold is slow, and a good explanation can save you the twenty minutes of tracing state by hand. But the value of that explanation depends entirely on what you ask it to focus on. An unscoped "explain this" request produces a line-by-line narration of syntax, which is the least useful kind of explanation for an engineer who can already read the language — you don't need to be told that `useEffect` runs after render. What you need is the thing that isn't obvious from reading top to bottom: *why* the state updates in that order, *when* the side effect actually fires, *what* would break the assumptions this component is making.

Use AI to understand unfamiliar code by focusing the request on a specific dimension:

```text
Explain this component.

Focus on:
- state flow
- side effects
- rendering conditions
- data dependencies
```

Rather than:

```text
What does this code do?
```

Ask AI to focus on a specific dimension. The difference isn't stylistic — a scoped request forces AI to trace the thing you actually care about instead of summarizing everything at equal weight, which is how a genuinely tricky race condition ends up as one clause in a twelve-bullet answer nobody reads closely enough to notice.

**Worked example — a dashboard widget.** Say you've inherited a `RevenueTrendWidget` that renders a chart, but it also silently recomputes a rolling average, debounces a resize handler, and memoizes a formatter — and nobody on the team wrote it. A vague request:

```text
What does this code do?
```

comes back as a paragraph confirming it renders a chart, which you already knew from looking at it. A focused request gets you the part that actually matters before you touch it:

```text
Explain RevenueTrendWidget.tsx.

Focus on:
- when the rolling average recalculates, and whether it can
  run against stale data if the date range prop changes
  mid-fetch
- what the resize debounce is protecting against
- which values are memoized and what would invalidate that
  memoization incorrectly
```

That version surfaces exactly the kind of bug you'd otherwise find by breaking it: for example, that the rolling average recalculates off the *previous* date range if the range prop changes while a fetch is in flight, because the memo key doesn't include the fetch's own request id. That's not something "what does this do?" was ever going to surface, because it requires AI to actually reason about the failure mode instead of describing the happy path.

---

## 9.4 Discover Existing Patterns

### Why discovery is a codebase question, not a code-generation question

The instinct when you need to handle a new case — a new mutation, a new error state, a new loading pattern — is to ask AI to build it. But if your codebase already has a convention for that case, the right first move is to ask AI to *find* it, not invent an alternative next to it. This is the same principle from Module 5's repository pattern learning applied at the moment of implementation rather than during setup: AI doesn't know your conventions exist unless you send it looking for them, and if you don't, it will happily generate a second, slightly different way of doing something you already do consistently elsewhere. Two inconsistent patterns for the same problem is worse than one imperfect pattern, because now every future engineer has to guess which one is "the" way.

```text
Find the pattern used by this repository
for handling mutation errors.

Do not create a new abstraction.
```

The "do not create a new abstraction" instruction matters for the same reason "do not write code" mattered in Module 8's Analyze step: without it, AI treats discovery as a warm-up and drifts into proposing something new the moment it notices the existing pattern isn't perfect. The point of this step is not to get AI's opinion on your error-handling convention — it's to get an accurate report of what that convention already is, so the next thing you build matches it.

**Worked example — a search box on a dashboard list.** Say you're adding a search input to a dashboard's order list, and it needs to debounce keystrokes before firing a query. Before writing a `useDebounce` hook, or reaching for a library, ask what already exists:

```text
Find the pattern used by this repository for debouncing
user input before triggering a query or mutation.

Check hooks/, components/search/, and any existing filter
or search inputs already in the codebase.
Do not create a new abstraction — report what exists
and where it's used.
```

A useful answer here would name something concrete: a `useDebouncedValue` hook already used by the customer-list filter, its default delay (300ms), and the one place it deviates (an admin search that uses 500ms because its query is expensive). That answer tells you to reuse `useDebouncedValue` with the standard delay — and it also tells you *why* the admin search is the exception, so you don't "fix" it into consistency by accident later. Generation should only start after this step confirms there's genuinely nothing to reuse — and even then, the new code should follow the shape discovery just showed you, not a shape you invented from scratch.

AI should be used to **discover the codebase**, not only generate code.

---

## Key Takeaways

* Generation is AI's strongest and riskiest default — strong for small, self-contained units (components, hooks, types, utilities, tests), risky the moment correctness depends on context spread across other files, which is when you're actually back in Analyze → Plan territory.
* Transformation preserves intent from working code, so the failure mode is silent alteration, not omission — always state what must stay identical (behavior, rounding rules, operation order, public shape) as part of the prompt, not as an assumption.
* "Explain this" and "what does this do?" invite a syntax narration; naming the dimension you actually need (state flow, timing, invalidation, race conditions) is what turns an explanation into something worth reading.
* Discovery should come before generation whenever a new case might already have a convention in your codebase — ask AI to find the existing pattern, and explicitly forbid it from proposing a new abstraction while looking.
* Two slightly different solutions to the same recurring problem are a worse outcome than one imperfect solution used consistently — discovery is what prevents that split.
* All four of these are still governed by Module 7's prompt discipline: scope the request, name the constraints, and don't bundle "generate" with "transform" or "explain" in the same turn.

## Try It Yourself

1. The next time you're about to write a new hook, utility, or component from scratch, stop and run a discovery prompt first: "Find the pattern used by this repository for [the thing you're about to build]. Do not create a new abstraction." Note whether it surfaces something you would have duplicated.
2. Take one piece of code in your current task that you inherited and don't fully trust — a reducer, a memoized calculation, a data-fetching hook. Write a focused "explain" prompt naming the specific dimension you're worried about (timing, invalidation, edge cases), run it, and compare the answer to what a plain "what does this do?" would have told you.

---

*[← Previous: Plan Before Code](./01-plan-before-code.md) | [Next: AI Debugging →](./03-ai-debugging.md)*
