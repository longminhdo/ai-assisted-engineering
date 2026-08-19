# Module 18 — Standard AI Workflow for the Team

*[← Previous: Prompt Quick Reference](../05-playbooks/09-prompt-quick-reference.md) | [Next: Team AI Coding Standard →](./02-team-coding-standard.md)*

## Why this matters

Every module up to this point taught you one move in isolation: how to decompose a requirement, how to hand AI the right context, how to make it show you a plan before it touches a file, how to verify what it produced, how to make it argue against its own code. Individually, each move is straightforward. The problem is what happens on a real team: eight engineers who've each internalized a different subset of these moves, in a different order, with a different sense of which ones are optional. One person plans everything up front; another jumps straight to implementation and only verifies at the end; a third skips human review of the plan because "the diff will catch it." All eight are technically "using AI well" by some definition, and none of them can hand off a PR to each other without re-explaining their process. This module is the synthesis — it stitches Modules 0 through 17 into one workflow diagram the whole team can point to, so the answer to "how are we supposed to use AI here" is one obvious loop instead of eight personal styles.

## Objective

Give the team a single, standardized end-to-end loop — requirement in, merged PR out — that names every stage explicitly, maps each stage back to the module that teaches it in depth, and makes it obvious which stages nobody is allowed to silently skip.

---

## 18.1 The Loop

This is the same diagram from the curriculum. Nothing about it is new content — it is every stage you've already practiced in Modules 0–13, arranged into the order you run them in on a real ticket:

```text
                REQUIREMENT
                     │
                     ▼
                 UNDERSTAND
                     │
                     ▼
                 DECOMPOSE
                     │
                     ▼
                   CONTEXT
                     │
                     ▼
                    PLAN
                     │
                     ▼
              HUMAN REVIEW PLAN
                     │
                     ▼
                 IMPLEMENT
                     │
                     ▼
                  VERIFY
                     │
              ┌──────┼──────┐
              ▼      ▼      ▼
             Type   Test   Runtime
              │      │      │
              └──────┼──────┘
                     ▼
                  AI REVIEW
                     │
                     ▼
                HUMAN REVIEW
                     │
                     ▼
                    MERGE
```

Two things worth noticing about the shape of this loop before you walk through it stage by stage. First, there are exactly two named "HUMAN REVIEW" gates, and they are not interchangeable — one reviews the *plan*, before any code exists, and one reviews the *implementation*, after it has already passed automated verification. A team that only has the second gate has already let AI make every architectural decision unsupervised; a team that only has the first has no guarantee the implementation actually matched the plan. Second, VERIFY is drawn as three parallel branches (Type, Test, Runtime) that all feed back into one AI REVIEW step — none of the three substitutes for the others, and none of them substitutes for AI REVIEW or HUMAN REVIEW either. Skipping a branch doesn't make the loop faster; it makes the loop incomplete, and incomplete loops are where AI-assisted work turns into a production incident.

---

## 18.2 Understand & Decompose — see Module 4

The loop doesn't start at IMPLEMENT, and it doesn't even start at PLAN. It starts with turning a raw requirement into something concrete enough to hand to anyone — human or AI — without them having to guess. Module 4 taught this as the chain `Requirement → Contract → Data → State → Architecture → Implementation → Verification`. UNDERSTAND is you reading the requirement and identifying what's actually being asked (and what's ambiguous about it); DECOMPOSE is turning that understanding into the concrete artifacts Module 4 covers — a data contract, a state-ownership decision, a component boundary, a list of edge cases. If you skip this stage and go straight to prompting, you're not saving time — you're just asking AI to decompose the problem for you, silently, inside its first response, where you have no chance to catch a wrong assumption before code gets written on top of it.

## 18.3 Context — see Module 5

Once you know what you're building, CONTEXT is deciding what AI actually needs to see to build it well. Module 5's context pyramid applies directly here: start with the task and the relevant contract, add existing patterns and related modules only as needed, and stop before you reach "full repository." This is also where Module 5's core habit — show AI a reference implementation instead of describing your conventions in prose — pays off, because a well-chosen reference file often does more work than three paragraphs of explanation.

## 18.4 Plan + Human Review Plan — see Module 8

PLAN and HUMAN REVIEW PLAN together are Module 8's `Analyze → Plan → Review → Implement` sequence, split at the point where a human has to sign off. You ask AI to analyze the requirement and existing code without writing any code yet, then to propose an implementation plan — component structure, state ownership, files touched, risks. HUMAN REVIEW PLAN is you actually reading that plan and asking Module 8's three questions: is the architecture correct, is the state in the right place, does this match our conventions. This is the cheapest place in the entire loop to catch a wrong direction, because at this point "catching it" costs you a sentence of feedback ("the filter state should be URL-owned, not component-owned") instead of a rewritten diff. Teams that skip straight from CONTEXT to IMPLEMENT are the ones who discover the architecture problem during HUMAN REVIEW near the end of the loop, when fixing it means throwing away finished work.

## 18.5 Implement — see Modules 7 & 9

IMPLEMENT is where Module 7's communication framework (Context → Goal → Constraints → References → Output) and Module 9's coding-partner patterns (generate, transform, explain, discover) actually get used, now that PLAN has already fixed the architecture. This is also where Module 8's incremental-implementation principle matters most: you don't ask AI to build the whole approved plan in one shot. You implement one step, look at it, then move to the next — `Step 1 → Review → Step 2 → Review → Step 3 → Review` — because a plan being correct doesn't guarantee every line of its implementation will be, and small diffs are the only way to catch a wrong turn before it compounds into the next step.

## 18.6 Verify — see Module 13 (and Module 14 for frontend-specific checks)

VERIFY is Module 13's pyramid, and the diagram's three branches — Type, Test, Runtime — are three different altitudes of that pyramid running in whatever order is cheapest: type-checking and lint catch the free mistakes first, tests isolate logic errors, and runtime verification is where you actually click through loading, empty, error, slow-network, and duplicate-click states, because compiling has never once proven a checkout button correctly disables itself. If the feature touches React state, data fetching, performance, accessibility, or responsive layout, Module 14's frontend-specific checklist is the version of "Runtime" and "Test" you apply — cache behavior, render counts, keyboard navigation, and the rest are frontend-specific instances of the same verification principle, not a separate step you can skip because the generic pyramid technically passed.

## 18.7 AI Review — see Module 12

Before a human spends review time on the diff, AI REVIEW puts AI in an adversarial role against its own (or your) output — Module 12's "review this as a critical senior engineer, try to prove it's wrong" pattern, ideally run one dimension at a time (state ownership, race conditions, accessibility) rather than as one vague "is this good?" pass. This stage exists precisely because the same tool that's good at generating plausible code is also good at spotting where plausible code is actually wrong, if you ask it to look for that specifically instead of asking it to confirm what it already wrote. It is cheap, and it catches a meaningful fraction of what would otherwise land directly on a human reviewer's plate.

## 18.8 Human Review — final accountability per Module 0

HUMAN REVIEW, the second gate, is not a formality that rubber-stamps whatever passed VERIFY and AI REVIEW. It's where Module 0's Pilot vs. Copilot principle cashes out: AI can propose, generate, transform, and even review adversarially, but the engineer decides, and the engineer is accountable for what merges. A green pipeline and a clean AI self-review both raise your confidence; neither one transfers ownership. This is the stage where you ask the questions no automated layer can ask — does this actually solve the business problem, is this the right trade-off for this codebase, would I be comfortable being the name on this commit if it breaks in production next month.

## 18.9 Merge

MERGE is not "AI is done, ship it" — it's "a human, having run every stage above, is willing to put their name on this." Everything before this point exists to make that final decision well-informed instead of a leap of faith.

---

## 18.10 Worked Example — "Apply Promo Code" at Checkout

Here's the full loop applied to one concrete ticket: *"Users should be able to enter a promo code at checkout and see the discount applied before paying."*

**REQUIREMENT.** The ticket as written above — intentionally underspecified, like most tickets are.

**UNDERSTAND.** You read it and surface what's actually ambiguous: what happens on an invalid code, can a user apply more than one code, does the discount apply to the subtotal or the subtotal plus shipping, is there a minimum order value. You don't ask AI yet — you either answer these from existing product rules or flag them to whoever wrote the ticket.

**DECOMPOSE (Module 4).** You write the contract before anything else:

```ts
type PromoCodeResult = {
  valid: boolean;
  discountAmount: number;
  reason?: "expired" | "not_found" | "minimum_not_met";
};
```

and settle state ownership: the promo code input is local UI state, the applied discount is derived server state (it depends on a validation call), and the cart subtotal is already owned elsewhere — this feature only adds a discount on top of it, it doesn't move where the subtotal lives.

**CONTEXT (Module 5).** You hand AI the `PromoCodeResult` type, the existing `CheckoutSummary` component, an existing similar mutation hook (`useApplyGiftCard`, if one exists) as a reference pattern, and the specific business rule about how discounts stack with shipping. You do not hand it the entire checkout module or the payment provider integration — neither is relevant to this task.

**PLAN (Module 8).** You prompt: *"Analyze this requirement and propose an implementation plan. Include component structure, state ownership, files to modify, and risks. Do not write code yet."* AI comes back proposing a `PromoCodeInput` component, a `usePromoCode` mutation hook modeled on `useApplyGiftCard`, and a note that the discount needs to recalculate the order total shown in `CheckoutSummary`.

**HUMAN REVIEW PLAN.** You read it and catch one thing: the plan put the applied-discount state in the global cart store. You push back — *"the discount is checkout-page-scoped, not global cart state; the cart itself doesn't know about promo codes, only this checkout flow does"* — and ask for a revised plan before implementation starts.

**IMPLEMENT (Modules 7 & 9).** You implement in steps: first the `usePromoCode` hook and its error handling, review it, then the `PromoCodeInput` component wired to it, review it, then the total-recalculation logic in `CheckoutSummary`. Each step is a small, reviewable diff rather than one large one.

**VERIFY (Module 13, plus Module 14).** Type-check confirms `PromoCodeResult` is used consistently and nothing fell back to `any`. A unit test asserts the actual discount math (`applies 15% off a $200 subtotal → $30 off`, not just `toBeDefined()`). Runtime verification catches what no static check would: submitting the form twice in quick succession fires two validation requests, and the loading state on the "Apply" button was never wired up, so a slow network lets a user click it three times.

**AI REVIEW (Module 12).** You ask AI to review the diff specifically for race conditions and error handling. It flags that an invalid code's error message never clears if the user then types a *different*, valid code before the first response returns — a stale-response race the happy-path testing didn't surface.

**HUMAN REVIEW (Module 0).** You read the full diff, confirm the fixes from both VERIFY and AI REVIEW landed, and separately judge something no tool can: is a flat error message the right UX here, or should an expired code suggest a currently-valid alternative promo — a product judgment call, not a correctness check.

**MERGE.** You merge it, having run every stage rather than having "asked AI to build the promo code feature" and read the diff once.

---

## Key Takeaways

* This module doesn't introduce a new skill — it's the assembly instructions for Modules 0–17, arranged into one loop the whole team runs the same way.
* The loop has two distinct human-review gates, not one: reviewing the *plan* before code exists, and reviewing the *implementation* after it's already been verified. Collapsing them into a single end-of-process review means every architectural mistake gets caught at the most expensive possible point.
* VERIFY is three parallel checks (Type, Test, Runtime), not one — passing type-check is not evidence the tests are meaningful, and passing tests is not evidence the runtime behavior is correct.
* AI REVIEW and HUMAN REVIEW are sequential, not redundant: AI REVIEW is cheap and catches a real subset of bugs before a human's more expensive attention gets spent on the same diff.
* The engineer is accountable at MERGE regardless of how clean every earlier stage looked — a green pipeline and a clean AI self-review raise confidence, they don't transfer ownership.
* If your team can't point to this diagram and say "here's the stage we're skipping and here's why," you don't have a standardized workflow — you have eight personal ones that happen to produce PRs.

## Try It Yourself

1. Take your next ticket and, before you open your AI tool, write down the ten stage names from the diagram on a scratch note. As you work the ticket, mark down which stage you're in every time you go to prompt AI. At the end, look at which stages you never marked — that's the stage you're actually skipping in practice, whatever you'd have said if asked.
2. Pick a PR you or a teammate merged recently with AI's help. Reconstruct which of the ten stages it visibly went through, using nothing but the PR description, commit history, and review comments. For any stage you can't find evidence of, ask: would a bug that stage is specifically good at catching have made it past everything else in this loop?

---

*[← Previous: Prompt Quick Reference](../05-playbooks/09-prompt-quick-reference.md) | [Next: Team AI Coding Standard →](./02-team-coding-standard.md)*
