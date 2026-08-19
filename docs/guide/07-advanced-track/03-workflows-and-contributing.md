# Advanced Track — Workflows & Contributing a Skill

*[← Previous: Skills vs. Rules](./02-skills-and-rules.md) | [Next: Back to guide overview →](../README.md)*

## Why this matters

An engineer who's just read Module 1's anatomy of a Claude project and Module 2's skills-vs-rules trade-off is now dangerous in a specific, predictable way: everything starts looking like it deserves an elaborate multi-agent workflow. A three-file bug fix gets planned as five parallel subagents with an integration step. A changelog entry gets a full orchestration pipeline. This is the same over-engineering instinct Module 0 warns about in a different guise — just as AI will happily produce an architecture you didn't ask it to justify, a team newly excited about shared tooling will happily build orchestration a task never needed. The other failure is the mirror image: an engineer who correctly identifies a repeated pattern in their own work, and then never does anything about it beyond privately being annoyed that they keep retyping the same prompt. Both failures are avoidable with the same discipline this whole guide has been teaching from Module 0 onward — match the structure to the actual scale of the problem, and then actually act on what you've noticed instead of just noticing it.

This page closes the guide with both halves: when heavier orchestration is genuinely worth it, and the practical, unglamorous loop for turning something you keep doing by hand into a skill the whole team benefits from.

## Objective

Know when a multi-agent workflow is the right amount of structure for a problem versus over-engineering a task that a sequential session or a single skill already handles well — and walk through the concrete steps of taking a pattern from "I keep doing this by hand" to "the team has a skill for it."

---

## 3.1 Multi-Agent Workflows: What They're For

A multi-agent workflow is scripted orchestration across many subagents — not one subagent delegated to once, but a structured pipeline where multiple agents work in parallel or in a defined sequence, each with its own scoped context, and their outputs get integrated at defined points. This is Module 4.6's Divide & Conquer principle, taken past the scale where one engineer coordinating a handful of parallel sessions by hand is still practical.

The tasks that actually justify this are ones where the *scale* of independent, parallelizable work exceeds what one sequential session — or even one engineer manually juggling a few subagents — can hold without becoming its own coordination burden:

* **A large-scale migration** — porting forty components from one API to another, where each component's migration is independent of the others but there are too many of them to review one sequential diff at a time.
* **A broad audit** — checking every route in a large app for a specific accessibility regression, where each route can be checked independently and the value is in coverage, not depth on any single one.
* **A wide research sweep** — surveying how a pattern is used across an entire monorepo before deciding whether to standardize it, where "how is this done across fifty packages" is the actual question, not "how is this done in the one package I'm looking at."

What these share is a dependency graph (Module 4.6) with many independent branches at once, at a volume where manually opening and tracking each one yourself stops being the efficient choice. The orchestration's job is exactly what Module 1.4's subagent containment already does at a larger scale: each branch works in its own scoped context, and only integrated results come back to the level a human is actually reviewing at.

## 3.2 When Plain Sequential Prompting Is Already Enough

Most day-to-day frontend work is not this. A single bug fix, a new form, a refactor of one hook, a changelog entry, a dependency bump on one package — these are exactly the tasks Modules 8 and 9 already cover well with a single, well-scoped session, or Module 2 (this track) covers well with a single well-scoped skill. Reaching for orchestration on tasks like these adds coordination overhead — integration points to review, parallel branches to reconcile — without a corresponding volume of independent work to justify it. You'd be paying the cost of Module 4.6's "explicit integration step, reviewed as carefully as any other task" for branches that had no real reason to be separate in the first place.

The question to ask before reaching for a multi-agent workflow is the same one Module 0 asks before reaching for AI at all: **does the structure I'm about to add match the actual scale of the problem in front of me?** A large migration across forty components has a real dependency graph with real independent branches — orchestration matches its scale. A single settings-tab feature does not suddenly gain that shape just because the tooling to orchestrate it exists. Module 17's *Over-Automation* anti-pattern applies here in a new form: not every task should be delegated to AI at all, and among the tasks that should, not every one should be delegated through the heaviest available structure. More orchestration is not a more advanced way to work a normal-sized task — it's the wrong tool for it.

A practical filter: if you can't name at least a handful of genuinely independent branches of work, each large enough to be worth its own scoped agent, you don't have a multi-agent workflow problem — you have a Module 8 (Plan Before Code) problem, and the fix is a better plan in one session, not more agents.

---

## 3.3 The Contribution Loop

Everything in Modules 1 and 2 (this track) described the mechanisms and the trade-offs. None of it happens on its own — someone has to actually notice a pattern, write it up, and get it in front of the team. Here's what that looks like in practice, end to end.

**1. Notice a pattern repeating across your own sessions.** This is the raw material, and it's usually more obvious in hindsight than in the moment — a prompt you've retyped with minor variations three times this month, a set of steps you keep re-explaining to AI because the same procedure comes up again ("how we do a dependency upgrade," "how we write a changelog entry," "how we add a new feature flag"). The signal isn't "this was hard once" — it's "I've done this same shape of thing more than once and re-explained it more than once."

**2. Draft it as a skill with a clear name and description.** Apply Module 2's discovery rules from the start: name it for the action, describe the concrete trigger situation, and write out the actual procedure — the same steps you'd otherwise retype. Resist the urge to make it more general than the pattern you actually observed; a skill scoped to exactly the repeated situation is more discoverable and more correct than one vaguely scoped to "helps with releases" in the hope it covers more ground.

**3. Test it on a real task, not a toy one.** Run the draft skill the next time the actual situation comes up — not a contrived example built to make the skill look good. A toy test only proves the skill works when nothing about the task pushes back; a real task will surface the edge case the draft didn't account for (the changelog entry for a PR that touches three unrelated things at once, the dependency upgrade where the changelog itself has no entry for the version in between). Treat a rough edge found here as the draft doing its job, not as a reason to abandon it.

**4. Get a teammate to try it and give feedback.** You wrote the skill around your own mental model of the task, which means you're the worst-positioned person to notice where its description or steps assume context only you have. A teammate running it cold — reading only the name and description, the way Claude Code itself will match against them — will tell you within one use whether it's actually discoverable and actually correct, not just correct to the person who wrote it.

**5. Land it for the team, at the same review bar as any other shared code.** A skill that other engineers' sessions will run against is exactly as load-bearing as a shared utility function, and it should go through the same scrutiny: does it match existing conventions, does it avoid inventing behavior nobody agreed to, is it reviewed by someone other than the author. Module 19's Rule 1 ("AI should not make architectural decisions without human review") applies to the skill's authorship too — a skill you personally use is a personal tool; a skill the team runs is a piece of shared infrastructure, and it earns that status through review, not through one person deciding it's ready.

This loop is deliberately not a one-time capstone exercise — it's the ordinary maintenance loop for a team's shared AI tooling, the same way code review is the ordinary maintenance loop for the codebase itself. The team that does this well isn't the one that ran it once; it's the one where noticing a repeated pattern and eventually landing a skill for it becomes as unremarkable as noticing duplicated code and eventually extracting a shared function.

## 3.4 Closing: The Same Five Principles, One Level Up

The curriculum this guide is built on states five core principles as the foundation for using AI on any single task:

> 1. AI doesn't know your codebase unless you show it.
> 2. Don't ask AI to solve a problem you haven't decomposed.
> 3. Plan before coding when the task is non-trivial.
> 4. Give AI patterns, not just instructions.
> 5. Never trust generated code without verification.

This entire track is those same five principles, aimed at the team's shared tooling instead of at one task. `CLAUDE.md` and skills are how you show AI your codebase's conventions *before* anyone has to explain them by hand (Principle 1) — for every session, not just the one you're in. Deciding which mechanism a piece of knowledge belongs in, and whether a workflow's scale actually justifies orchestration, is decomposition applied to the team's tooling instead of to a feature (Principle 2). The contribution loop's draft-test-review sequence is planning before landing something non-trivial (Principle 3). A well-scoped skill or a hook enforcing a real convention is, quite literally, giving AI a pattern to follow instead of hoping every future session remembers an instruction (Principle 4). And landing a skill at the same review bar as any other shared code is verification applied to the thing that will shape every future session's output, not just this one's (Principle 5).

Modules 0–19 taught you to be good at directing AI on your own work. This track taught you to make that goodness compound — to leave the shared structure a little better than you found it, so the next engineer's first session on a new task starts from a little more of what the team has already learned, instead of from zero.

---

## Key Takeaways

* Multi-agent workflows earn their complexity when a task has a real dependency graph with many independent branches at a volume beyond what one engineer coordinating a few subagents by hand can hold — a large migration, a broad audit, a wide research sweep. Most day-to-day work is not this.
* Reaching for orchestration on a normal-sized task is Module 17's Over-Automation anti-pattern in a new form: more structure than the problem's actual scale justifies.
* The contribution loop is: notice a repeated pattern in your own sessions → draft it as a skill with a clear name and description → test it on a real task → get a teammate's cold-read feedback → land it at the team's normal review bar.
* A skill that other engineers' sessions will run against is shared infrastructure, not a personal tool — it earns that status through review, the same way any shared code does.
* This whole track is the guide's five core principles applied one level up: not to a single task, but to the shared instructions, skills, and guardrails every future task on the team will run against.

## Try It Yourself

1. Think back over your last month of AI sessions. Name one thing you've re-explained or retyped more than once — not hypothetically, an actual repeated instance. Draft it as a skill following 3.3's steps 1–2, then actually run it against the next real occurrence of that task rather than a toy example.
2. Before your next task that feels like it might need heavy orchestration, apply 3.2's filter out loud: name the independent branches, and count them. If you can't name more than one or two, treat that as your answer — plan it as a single well-scoped session instead, and save the orchestration for when the count is actually large.

---

*[← Previous: Skills vs. Rules](./02-skills-and-rules.md) | [Next: Back to guide overview →](../README.md)*
