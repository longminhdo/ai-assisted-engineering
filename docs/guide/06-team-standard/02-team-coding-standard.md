# Module 19 — Team AI Coding Standard

*[← Previous: Standard AI Workflow for the Team](./01-standard-ai-workflow.md) | [Next: Advanced Track Overview →](../07-advanced-track/README.md)*

## Why this matters

Every other module in this guide takes a paragraph, a diagram, or a worked example to teach one habit. This module compresses all of them into ten one-line rules — short enough that a lead can paste one into a PR comment instead of re-explaining, for the fifth time, why "AI suggested it" isn't a justification for a new state management library. That's the actual use case: not a document you read once, but a shared vocabulary the team points to mid-review so disagreements about AI-assisted code resolve in one sentence instead of a debate. If you only memorize one page from this entire guide, it should be this one.

## Objective

Give the team ten rules, each traceable back to a module you've already worked through, that the team holds each other to in code review — so "that's not how we use AI here" has a specific, citable reason behind it every time.

## The 10 Rules

### Rule 1

> **AI should not make architectural decisions without human review.**

Violated: a component gets a new global store, a new fetching layer, or a new folder structure because AI proposed it in the first response and nobody stopped to ask why. Followed: AI can propose an architecture, but a human evaluates it against the codebase's existing conventions and trade-offs before it gets implemented — the Pilot vs. Copilot split from Module 0, where AI proposes and the engineer decides. This is the same failure Module 0 names directly: treating a suggestion as a decision because it arrived with confident, well-formatted code attached.

*Example PR comment: "This introduces a new caching layer — was this reviewed as an architecture decision, or did it just come out of the first AI response?"*

### Rule 2

> **Non-trivial tasks should be planned before implementation.**

Violated: someone prompts "build the entire user management feature" and starts reviewing whatever comes back line by line, discovering the state-ownership problem only after 400 lines already exist. Followed: the task goes through Module 8's `Analyze → Plan → Review → Implement` sequence first, so component structure, state ownership, and files touched are agreed on while they're still cheap to change — a sentence of feedback instead of a rewritten diff.

*Example PR comment: "This touches five files and adds two new hooks — where's the plan comment or thread that got reviewed before this was implemented?"*

### Rule 3

> **Use existing project patterns whenever possible.**

Violated: a PR adds a hand-rolled modal component when `Modal.tsx` already exists in the design system, or a new error-toast pattern that duplicates one three components over — because AI wasn't shown either and had no way to know they existed. Followed: before generating anything, you find the closest existing implementation and tell AI to follow it, per Module 5's repository pattern learning — "use `OrderFilterModal.tsx` as the reference implementation" instead of describing your conventions in prose.

*Example PR comment: "We already have `useApplyGiftCard` for this exact shape of mutation — this should follow that pattern, not invent a new one."*

### Rule 4

> **Prefer minimum sufficient context.**

Violated: someone pastes the entire `api/` folder, every component in the feature area, and all the CSS into a single prompt "just in case," and AI comes back having picked the wrong pattern out of the noise or invented something unnecessary. Followed: the context is deliberately scoped to what Module 5's context pyramid says the task actually needs — the relevant contract, the closest existing pattern, and nothing else. More context is not automatically better context; past a point it actively increases hallucination risk and wrong-pattern selection.

*Example PR comment: "Did you scope the context for this prompt, or hand it the whole module? The naming here doesn't match anything we actually use."*

### Rule 5

> **Keep AI-generated changes small and reviewable.**

Violated: a single PR contains a new hook, a rewritten component, a schema change, and an unrelated refactor, all generated in one pass, so the reviewer either rubber-stamps it or spends an hour reconstructing what changed and why. Followed: implementation happens in Module 8's incremental loop — `Step 1 → Review → Step 2 → Review` — so each diff is small enough that a human can actually hold the whole thing in their head and a wrong turn gets caught before it compounds into the next step.

*Example PR comment: "Can you split this into the hook change and the UI change? As one diff I can't tell which part introduced the extra re-render."*

### Rule 6

> **Never trust generated code without verification.**

Violated: the code compiles, the happy path works in a five-second manual click-through, and it gets merged — until it turns out the loading state was never wired up and a slow network lets a user submit a form three times. Followed: Module 13's full verification pyramid runs before merge — type-check, lint, tests that assert actual behavior, and runtime verification of loading, empty, error, and duplicate-click states. Compilation has never once proven a checkout button correctly disables itself.

*Example PR comment: "This passes type-check but I don't see a test that actually asserts the discount math — `toBeDefined()` isn't verification."*

### Rule 7

> **Do not introduce dependencies without justification.**

Violated: AI reaches for a new date library, a new state manager, or a new form library because it's a common choice in its training data, not because the codebase needs it or lacks an equivalent already. Followed: Module 11's refactoring constraints apply just as much to new code as to refactors — "do not introduce dependencies" is an explicit constraint you state up front, and any new dependency has to be argued for the same way a human-authored one would be, tying back to the anti-pattern Module 17 calls AI-Driven Architecture: AI can suggest a library, but the team evaluates the trade-off, not the other way around.

*Example PR comment: "Why did this need a new date-formatting library when we already use `date-fns` everywhere else in the app?"*

### Rule 8

> **Business requirements are the source of truth, not AI output.**

Violated: a requirement is genuinely ambiguous about how a discount should apply, AI fills the gap with a plausible-sounding assumption, and the assumption ships as if it were a product decision because nobody flagged it. Followed: the ambiguity gets surfaced and resolved against the actual requirement — or against whoever owns the requirement — before implementation, exactly the discipline Module 0 describes when it lists what AI cannot know on its own: pagination rules, permission logic, and business rules are not things AI infers correctly by default, they're things you have to supply.

### Rule 9

> **Existing code is a better reference than a textual description.**

Violated: someone writes three paragraphs describing "how we handle mutation errors in this codebase" instead of pointing at the file that already does it, and AI's implementation drifts from the actual pattern in ways the prose didn't capture. Followed: Module 5's core habit — show the pattern, don't describe the pattern — is applied by default: point AI at `OrderFilter.test.tsx` or `useOrderQuery.ts` and tell it to follow that file's structure, naming, and error handling, because a real file communicates conventions a description always leaves gaps in.

*Example PR comment: "Instead of explaining our modal conventions again, just point it at `UserFilterModal.tsx` next time — that's what this should have matched."*

### Rule 10

> **The engineer owns the final result.**

Violated: a bug ships and the explanation is "AI generated that part" — as if authorship transfers responsibility away from whoever opened the PR. Followed: every rule above exists in service of this one. Module 0's Pilot vs. Copilot framing means AI can propose, generate, transform, and review, but the name on the commit is the name accountable for it in production next month, regardless of how clean the diff looked or how confidently AI presented it.

## Key Takeaways

* These ten rules aren't new content — they're the enforcement layer for everything Modules 0–18 already taught, condensed to a size a reviewer can actually hold in their head during a PR.
* Rules 1, 2, 5, and 6 exist to keep architecture and risk decisions human-gated at the cheapest possible point, before a large diff makes reversing course expensive.
* Rules 3, 4, and 9 exist to keep AI's output aligned with the codebase that already exists, not the average codebase in its training data.
* Rule 7 and Rule 8 exist to stop AI's default assumptions — a familiar dependency, a plausible interpretation of an ambiguous ticket — from quietly becoming team decisions.
* Rule 10 is the one that makes the other nine enforceable: without explicit ownership, "AI wrote it" becomes an excuse instead of a fact.
* A rule the team can't cite by number in a review comment is a rule the team doesn't actually have — if you can't name which of the ten a PR violates, figure that out before merging it anyway.

## Try It Yourself

1. Pick the one rule from this list you are personally most likely to skip when a deadline is close — be honest, it's usually Rule 2 or Rule 5 under pressure. Name one specific task on your board this sprint where you'll deliberately apply it anyway, and note afterward whether it caught anything.
2. Look back at the last three PRs you merged with AI's help. For each one, find the rule it's the weakest match for. If you can't find a weak match in any of the three, look harder — or start applying this checklist to the next one before you merge it, not after.

---

*[← Previous: Standard AI Workflow for the Team](./01-standard-ai-workflow.md) | [Next: Advanced Track Overview →](../07-advanced-track/README.md)*
