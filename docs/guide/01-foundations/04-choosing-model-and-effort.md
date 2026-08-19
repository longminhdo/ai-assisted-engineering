# Module 3 — Choosing Your Model & Effort

*[← Previous: Controlling Your Session](./03-controlling-your-session.md) | [Next: Problem Decomposition →](../02-thinking-tools/01-problem-decomposition.md)*

## Why this matters

Picture two engineers on the same team. One runs every single request — renaming a variable, writing a one-line type, and designing a new caching strategy for the dashboard — through the same top-tier, maximum-reasoning-effort model, because "more powerful is always better." Their teammate does the opposite: everything runs on the fastest, cheapest setting, including a genuinely gnarly race-condition bug that gets three shallow, wrong guesses in a row before anyone thinks to slow down. Both engineers are making the same mistake in opposite directions — treating model and effort choice as a fixed setting instead of a decision that should change with the task in front of them. The first wastes time and money getting a slow, over-elaborate answer to a question that had an obvious one-line fix. The second burns more time re-prompting a fast model through a problem it was never resourced to solve than a single well-resourced attempt would have taken.

## Objective

Treat "which model, and how much reasoning effort" as a deliberate resourcing decision made per task — the same way you'd decide whether a bug deserves five minutes or a half-day investigation — rather than a fixed default you never revisit.

---

## 3.1 Not All Tasks Deserve the Same Model

Most AI coding tools today give you a choice along two roughly independent dimensions: which **model** to use (commonly a faster/lighter tier versus a larger/more capable one), and how much **reasoning effort** that model applies to a given request (how long and how thoroughly it "thinks" before answering, when that's configurable). Neither dimension is about raw ability in a way where "more" is free — a larger model or a higher effort setting is slower and more expensive per request, and past a certain point it doesn't make a trivial task's answer any more correct, it just makes you wait longer for the same answer.

The variable that should drive your choice isn't the task's *size* — a 200-line component isn't automatically more deserving of a top-tier model than a 20-line one — it's the task's **ambiguity, risk, and required judgment**. A large but mechanical task (rename this prop across 30 files, apply this exact pattern to 5 similar components) is often *lower* effort than a small but high-stakes one (design the caching strategy for a shared data layer three features depend on), even though it touches far more code.

**Worked example.** "Rename `userId` to `accountId` across this feature's 12 files, including the API contract and tests" is large in scope but requires almost no judgment — the transformation is unambiguous and mechanically verifiable (does it compile, do tests pass). "Decide how the notification panel should reconcile a websocket-pushed update with an optimistic local mutation the user just made" is small in scope but genuinely ambiguous — there's a real design decision buried in it, with several plausible answers and a wrong one that could cause hard-to-reproduce bugs. Running the rename through a slow, maximum-effort configuration doesn't make it more correct; running the reconciliation question through a fast, minimal-effort one risks getting a confident, plausible-sounding answer to a question that actually deserved careful reasoning.

---

## 3.2 What "Reasoning Effort" Actually Buys You

Higher reasoning effort generally means the model spends more of its process exploring the problem before committing to an answer — considering alternative approaches, checking its own intermediate steps, catching an inconsistency before it reaches the final response — rather than pattern-matching to the first plausible-looking answer. This has real value on tasks where the *first* plausible answer is often subtly wrong: multi-step architectural trade-offs, debugging a cause that isn't obvious from a quick read, reviewing code for problems that only show up under specific interaction sequences.

It has close to zero marginal value on tasks where the correct answer is already unambiguous once the model reads the input: converting a class component to a function component using a pattern you've shown it, generating a test for a pure function, writing a type from an API response you've pasted in. Extra "thinking" on an unambiguous task doesn't produce a better answer — the ceiling was already reached — it just costs more time and, on metered usage, more money for a wait that ends at the same destination.

The practical failure mode to watch for isn't just "used too much effort on an easy task" (mostly a waste, rarely wrong) — it's the reverse: **using too little effort on a task with real ambiguity**, where a fast, shallow pass produces something that reads as confident and complete (Module 0's "confidently wrong" failure mode) but never actually grappled with the hard part of the problem. That's the expensive mistake, because it doesn't look like a shortcut was taken — it looks like a finished answer.

**Worked example.** Asked at low effort to "review this data-fetching hook for problems," a fast pass might catch the obvious stuff — a missing dependency in a `useEffect` array, an unhandled error case — and stop there, because those are the first plausible findings and nothing prompted it to keep looking. Asked the same question at higher effort, the model is more likely to actually trace what happens when the component unmounts mid-fetch, or when two instances of the hook race against each other with the same cache key — the kind of finding that only surfaces if something keeps exploring past the first obvious answer. If the hook in question is shared across a dozen features, that's exactly the case where the extra effort is worth the extra time; if it's a one-off hook used in a single low-traffic settings page, the fast pass's findings were probably good enough.

---

## 3.3 A Practical Decision Framework

Rather than deciding per-task from scratch, calibrate against three questions:

1. **How reversible is a wrong answer?** A misnamed variable is a two-second fix in review. A wrong state-ownership decision (Module 4.3) that six other components get built on top of is expensive to unwind later. Low reversibility pushes toward more effort up front.
2. **How much does the correct answer depend on judgment versus mechanical pattern-matching?** "Apply this exact refactor pattern to these five files" is pattern-matching. "Should this be optimistic or pessimistic UI, given these failure modes" is judgment. Judgment-heavy tasks benefit far more from higher effort than mechanical ones do.
3. **What's actually at stake if it's subtly wrong?** A styling tweak that's slightly off is caught on sight in a visual review. A race condition, an auth check, or a data-integrity assumption that's subtly wrong can ship silently and surface as a production incident weeks later. Higher stakes justify more effort even when the task looks small.

A simple working default: reach for a faster tier and lower effort for mechanical generation, transformation, and boilerplate (Module 9.1's "generate" category) — reach for a more capable tier and higher effort for architecture decisions, adversarial code review (Module 12), hypothesis-driven debugging on a non-obvious bug (Module 10), and anything where a wrong answer is expensive to catch later.

**Worked example — a single PR, two settings.** A PR adds a new settings tab: it needs (a) a fairly boilerplate form component following an existing pattern, and (b) a decision about whether the tab's "unsaved changes" state should be tracked per-field or as a single dirty flag, given that a sibling tab already made this decision differently and the two are inconsistent. Task (a) is a good candidate for a fast, low-effort pass — the pattern is already established, and there's little ambiguity to reason through. Task (b) is exactly the kind of small-but-judgment-heavy decision from 3.1 that deserves a slower, higher-effort pass — ideally one where you also feed in *why* the sibling tab made its choice, so the model is reasoning about a real inconsistency rather than picking an option in a vacuum.

---

## 3.4 The Cost of Getting the Calibration Wrong

Over-provisioning (top effort on a trivial task) mostly costs time and money — real costs on a team, but forgiving ones, because the output is still correct, just slower to arrive than necessary. Under-provisioning is the direction that actually damages the codebase, because it fails silently: a fast, shallow pass on a task that needed real reasoning doesn't come back with "I'm not confident about this" — it comes back with a normal-looking, confidently-stated answer that simply never engaged with the hard part of the problem (the same overconfidence failure mode from Module 0.2, but caused by resourcing rather than missing context).

The practical implication is asymmetric: when you're genuinely unsure which way to calibrate a task, err toward *more* effort rather than less, and treat the cost as the price of not having a wrong architectural decision baked into three weeks of downstream code. Reserve the fast, low-effort setting for the cases where you're confident the task is mechanical — not as a default you fall back to because it's faster to type.

---

## Key Takeaways

* Model and effort choice is a resourcing decision that should track a task's ambiguity, risk, and required judgment — not its size, and not a fixed personal default.
* Higher reasoning effort buys more exploration before committing to an answer — valuable when the first plausible answer is often subtly wrong, wasteful when the correct answer is already unambiguous.
* The dangerous direction to miscalibrate is under-provisioning judgment-heavy tasks: a fast, shallow answer to a hard question looks just as confident and finished as a well-reasoned one, which is exactly what makes it dangerous.
* Calibrate against reversibility, how much judgment (versus pattern-matching) the task actually requires, and what's at stake if the answer is subtly wrong — not against how many lines of code are involved.
* When unsure which way to round, round up. The cost of over-provisioning is time and money; the cost of under-provisioning on the wrong task is a confidently-wrong decision that ships.

## Try It Yourself

1. Take your last five AI-assisted tasks. For each, classify it against the three questions in Section 3.3 (reversibility, judgment vs. pattern-matching, stakes if subtly wrong), and compare that classification to the model/effort you actually used. Note any mismatches — especially any judgment-heavy, low-reversibility task that got a fast, low-effort pass.
2. Next time you have a task with real ambiguity in it (a state-ownership call, a trade-off between two valid approaches), deliberately run it once at a fast/low-effort setting and once at a slower/higher-effort one, without changing the prompt. Compare not just the final answer but whether the higher-effort version surfaced a consideration the faster one never mentioned at all.

---

*[← Previous: Controlling Your Session](./03-controlling-your-session.md) | [Next: Problem Decomposition →](../02-thinking-tools/01-problem-decomposition.md)*
