# Module 0 — AI Mindset

*[← Back to guide overview](../README.md) | [Next: How AI Processes Your Request →](./02-how-ai-works.md)*

## Why this matters

Picture the PR: an engineer asks AI to "add a notifications panel to the dashboard," gets 200 lines of plausible-looking React back, skims it, and ships it. Two days later QA reports that notifications don't clear when a user logs out, the unread count double-counts on fast refresh, and the panel fetches on every parent re-render instead of once. Nothing here was a hallucination — the code compiled, the component rendered, the happy path worked in the demo. The problem is that the engineer treated AI's output as a finished decision instead of a proposal that still needed their judgment. The team that avoids this failure mode isn't the one with the best prompts; it's the one where every engineer holds a clear mental model of what AI is actually reliable at, where it quietly fills gaps with guesses, and who is accountable when those guesses are wrong.

## Objective

Shift your default assumption from:

> "AI writes code for me."

to:

> **"AI is an engineering tool that I am responsible for directing."**

Everything else in this guide — decomposition, context, prompting, planning, review — is a technique for acting on that second sentence. This module is where you install it as a habit before you touch any of the techniques.

---

## 0.1 What Is AI Good At?

AI performs best on tasks that are **well-scoped and locally-defined** — where the input is a piece of code or text in front of it, the transformation is well-understood, and success can be checked by reading the output. It performs worst on tasks that require knowledge that lives outside the prompt: your team's conventions, the business reason a rule exists, or a decision nobody has made yet. Recognizing which category a task falls into, before you ask, is the single highest-leverage skill in this whole guide — it tells you whether to hand the task over directly or to decompose and add context first (Modules 4–5).

The tasks below work well precisely because their scope and inputs are relatively clear — AI is transforming or generating something local, not inventing your architecture:

* Code generation
* Code transformation
* Code explanation
* Pattern discovery
* Refactoring
* Test generation
* Documentation generation
* Debugging assistance
* Comparing approaches
* Boilerplate generation

Examples from the curriculum:

```text
Convert this Vue Options API component
to Composition API.
```

```text
Generate unit tests for this function
based on the existing testing pattern.
```

Here's another example, on a different kind of task — explaining and transforming existing code rather than inventing new behavior:

```text
This `useCheckoutTotals` hook recalculates tax and
shipping on every keystroke in the promo code field.
Explain why, and then rewrite it to only recalculate
when the debounced promo code or cart items change.
Keep the existing return shape.
```

This works well for the same reason the Vue and test-generation examples do: the input (the hook, its current behavior, the trigger) is fully visible in the code, the desired change is a known pattern (debounce + dependency narrowing), and you can verify correctness by reading the diff and running the existing tests. AI doesn't need to know anything about your business beyond what's on the screen.

---

## 0.2 Where Is AI Weak?

AI struggles exactly where the previous section's strengths run out: anywhere the correct answer depends on information that isn't in the prompt or the visible code. It cannot ask a product manager what "recently active" means, cannot check with your security team whether a field is safe to log, and cannot infer that your team standardized on Zustand for client state three sprints ago unless you tell it. When context is missing, AI does not stop and wait — it fills the gap with the most statistically plausible default, stated with the same confident tone as everything else it writes. That confidence is what makes this failure mode dangerous: a wrong guess reads exactly like a right answer.

AI can struggle when:

* Business requirements are unclear.
* Context is incomplete.
* Project conventions are unknown.
* Architecture is implicit.
* Multiple modules are tightly coupled.
* Trade-offs have not been decided.
* Real production behavior needs to be understood.

The curriculum's example:

```text
Build the user management page.
```

AI does not automatically know:

* Where should state live?
* Which API should be used?
* Which Design System should be used?
* What is the pagination pattern?
* How should permissions work?
* How should errors be handled?
* Should filters sync with the URL?
* Should this be server state or client state?
* Is SSR involved?

Here's the same failure mode on a different feature, so you can see it's not specific to "user management":

```text
Build a checkout page with a promo code field
and an order summary.
```

This looks concrete — it names two UI pieces — but it hides the same category of unanswered questions as the user management example:

* Does the promo code get validated client-side, server-side, or both?
* What happens to the order summary while promo validation is in flight — disabled state, optimistic update, spinner?
* Is the cart server state (fetched, revalidated) or client state (local reducer)?
* What's the error contract when a promo code is invalid, expired, or the API times out?
* Does this need to survive a page refresh mid-checkout?
* Is there an existing `<OrderSummary>` or `<PriceLine>` component elsewhere in the codebase, or does AI invent a new one?

Faced with a prompt like this, AI **will make assumptions** on every one of these points — usually the most generic, framework-default assumption, not your team's actual pattern. It won't flag most of them as assumptions; they'll just show up baked into the code. If you don't decompose the request or supply the missing context yourself (Modules 4 and 5), you are not getting "a checkout page" — you are getting a stranger's guess at your checkout page, and you'll pay for the gap later, usually in review or in production.

---

## 0.3 Pilot vs. Copilot

The reason this distinction needs its own section, rather than just being implied by 0.1 and 0.2, is that it's about **accountability**, not just capability. AI being weak at architecture doesn't mean AI can't produce architecture — it will happily produce one if you ask, complete with folder structure and state management choices. The question this framework answers is: when that architecture turns out wrong, whose job was it to catch that before it shipped? The answer is always yours. AI has no stake in the codebase, no memory of the incident review, and no name on the commit.

### Engineer — Pilot

The engineer owns:

* Architecture
* Business logic
* Technical decisions
* Security
* Performance
* Correctness
* Final code review

### AI — Copilot

AI assists with:

* Exploration
* Proposals
* Generation
* Transformation
* Review
* Debugging
* Testing

A copilot in an aircraft can fly the plane, run checklists, and flag anomalies — but the pilot in command is the one who decides what to do about a warning light, and it's the pilot's judgment call that goes in the incident report. Treat AI's output the same way: as a competent copilot's input into your decision, not as the decision itself. Concretely, this means you don't ask AI "should this be server state or client state?" and then type whatever it says into the PR description as the rationale — you ask it to lay out the options and trade-offs, and you make the call, because you're the one who'll be debugging it under load six months from now.

Core principle:

> **AI can propose. Engineers decide.**

---

## Key Takeaways

* AI is strong at locally-scoped, well-defined transformations (convert, generate tests, explain, refactor a known pattern) and weak wherever the correct answer depends on context it can't see (business rules, conventions, unmade trade-offs).
* When context is missing, AI doesn't stop to ask — it silently fills gaps with plausible defaults, delivered with full confidence. Vague prompts like "build the checkout page" produce a stranger's guess at your checkout page, not your checkout page.
* You are the pilot: you own architecture, business logic, security, performance, correctness, and the final review. AI is the copilot: it explores, proposes, generates, and assists — it does not decide.
* Before sending any request to AI, ask yourself which category it falls into (0.1 vs 0.2) — that answer tells you whether to hand it over directly or to decompose and add context first (Modules 4–5).
* "AI can propose. Engineers decide." is not a slogan to recite — it's the standard you should be able to point to when explaining why you rewrote or rejected something AI produced.

## Try It Yourself

1. Pick a task you're about to hand to AI this week. Before you write the prompt, classify it: is it closer to the "convert/explain/generate tests" category (0.1) or the "build the X page/flow" category (0.2)? Write down, in one sentence, why.
2. If it falls into the 0.2 category, list the unanswered questions the way this module did for the checkout example (state ownership, API, error handling, existing components to reuse, etc.) *before* you prompt. Then compare: did AI's actual output guess wrong on any of the questions you identified? That gap is exactly what the next two modules teach you to close.

---

*[← Back to guide overview](../README.md) | [Next: How AI Processes Your Request →](./02-how-ai-works.md)*
