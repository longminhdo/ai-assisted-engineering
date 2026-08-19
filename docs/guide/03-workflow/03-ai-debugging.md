# Module 10 — AI Debugging

*[← Previous: AI as a Coding Partner](./02-ai-coding-partner.md) | [Next: AI Refactoring →](./04-ai-refactoring.md)*

## Why this matters

Picture a bug report that says "the checkout button spins forever sometimes" and nothing else. You paste that into AI, it changes the loading state's initial value, you try again, it still happens. You paste the same one-liner again, it wraps the mutation in a `try/catch` that was already there, you try again, still happens. Three "fixes" later, the bug is still live in production and you've burned twenty minutes without learning a single new fact about *why* it happens — because nobody ever gave AI a real repro, and without one it's not debugging, it's guessing with syntax highlighting. The fix for that isn't a smarter model; it's treating debugging as an investigation with evidence, not a vending machine you shake until code falls out.

## Objective

Turn AI into a debugging partner: a tool for narrowing down causes through evidence, not a black box you feed a vague complaint and hope it hallucinates the right patch.

---

## 10.1 Debugging Context

AI cannot see your browser, your network tab, your terminal, or the ticket someone filed against your feature. Everything it knows about the bug is whatever you type, which means the quality of its diagnosis is capped by the quality of your report — the same way it would be capped for a human colleague you paged at 2am with "it's broken, please look." Handing over rich context up front costs you two extra minutes of writing and saves several rounds of AI proposing fixes for bugs that don't exist.

A good debugging request includes:

```text
Expected behavior
Actual behavior
Error
Stack trace
Relevant code
Recent changes
Environment
What has already been tried
```

Each line earns its place. "Expected vs. actual" turns a vague complaint into a falsifiable claim AI can reason about. "Error" and "stack trace" are ground truth — they tell AI exactly which line threw, instead of making it guess across an entire component. "Recent changes" matters because most regressions are caused by something that changed recently, and if you don't mention the dependency bump from yesterday, AI will spend its first three guesses on code that hasn't moved in months. "Environment" catches the class of bug that only reproduces in Safari, or only in production behind a CDN, or only for users on a stale service worker. And "what has already been tried" stops AI from re-suggesting the fix you already ruled out five minutes ago.

**Worked example — checkout confirmation spinner.** Say support reports that some users are stuck on a spinner after paying: the payment succeeds (Stripe shows the charge), but the UI never advances to the confirmation screen. Filling in the full context checklist looks like this:

```text
Expected:
After POST /checkout/confirm resolves 200, the app navigates
to /order/:id/confirmation and the spinner on the Place Order
button clears.

Actual:
For some users, the spinner spins indefinitely even though
the payment succeeded (confirmed in the Stripe dashboard and
in server logs). No visible error in the UI.

Error:
No error is thrown or logged client-side. No error boundary
triggered.

Stack trace:
None — this is a silent hang, not a crash.

Relevant code:
useCheckoutMutation.ts (the mutation + onSuccess handler)
CheckoutPage.tsx (calls the mutation, owns isSubmitting state)

Recent changes:
Yesterday we added an analytics call inside onSuccess
(trackPurchase(orderId)) as part of PR #482.

Environment:
Reported only in production, not reproducible locally or in
staging. Affects roughly 2% of checkout attempts per the
dashboards, not tied to one browser.

What has already been tried:
Confirmed the API call itself succeeds (verified in server
logs and Stripe). Confirmed it's not a network timeout —
the response comes back within 400ms.

Analyze possible causes.

Do not modify code yet.
```

Notice this report doesn't ask for a fix — it ends with "analyze possible causes" and an explicit instruction not to touch code yet, which is exactly the discipline the next two sections build on.

---

## 10.2 Avoid

The single most common way engineers under-use AI for debugging is typing the same sentence they'd type into a Slack message to a teammate who already has full context on the codebase: "why doesn't this work?" A teammate can walk over and look at your screen. AI cannot. That question forces AI to either ask a dozen clarifying questions (which most people skip past) or guess, and guessing is where the random-change loop from Section 10.4 starts.

Avoid:

```text
Why doesn't this work?
```

Prefer a structured debugging prompt — expected, actual, and the relevant code, with an explicit instruction to analyze rather than fix:

```text
Expected:
The modal closes after a successful mutation.

Actual:
The API succeeds but the modal remains open.

Relevant code:
...

Analyze possible causes.

Do not modify code yet.
```

The "do not modify code yet" line matters as much here as it did in Module 8's "do not write code" for planning, and for the same reason: once AI has generated a patch, that patch anchors the conversation. You end up reviewing *its* theory of the bug instead of first agreeing on what the bug even is.

**Worked example, continued.** For the checkout spinner, the "avoid" version of this request is the one-liner: "the checkout button gets stuck spinning, can you fix it?" The "prefer" version is the trimmed core of the full context dump from Section 10.1 — expected, actual, relevant code, and the instruction to analyze first:

```text
Expected:
Spinner clears and the app navigates to the confirmation
page after a successful checkout.

Actual:
Spinner spins indefinitely for ~2% of checkout attempts,
even though the payment succeeded server-side.

Relevant code:
useCheckoutMutation.ts, CheckoutPage.tsx

Analyze possible causes. Do not modify code yet.
```

That's a strictly smaller prompt than the full context dump, and it's still enough for AI to start reasoning productively — because "expected/actual/relevant code" is the floor, not the ceiling, of what a debugging prompt needs.

---

## 10.3 Hypothesis-Driven Debugging

A diagnosis you can't verify is just a guess wearing a lab coat. The value of asking AI for causes instead of a fix is that a cause comes with a *reason to believe it* and a *way to check it* — and both of those are things you, the engineer, can independently confirm before a single line of code changes. This is the same "human review before implementation" principle from Module 8's plan step, applied to root-causing instead of architecture.

Ask AI:

```text
List the top 3 possible causes.

For each cause:
- explain why
- identify evidence
- suggest how to verify it
```

The engineer verifies the hypothesis before applying a fix. That verification step is not optional busywork — it's the entire point. AI can be confidently wrong about which cause is most likely, especially when it can't run your app, so "plausible-sounding" and "actually true" are two different bars, and only you can clear the second one, usually with a breakpoint, a log line, or a quick reproduction.

**Worked example, continued.** For the checkout spinner bug from Section 10.1, a useful response to the hypothesis prompt might look like this:

```text
1. The onSuccess handler throws inside trackPurchase(orderId),
   and the throw happens before navigate() runs.
   Why: PR #482 added trackPurchase inside onSuccess, ahead of
   the navigation call, with no try/catch around it.
   Evidence: the bug started appearing after PR #482 shipped
   (per the "recent changes" note); no client error is logged,
   which fits an analytics call that fails silently (e.g. an
   ad blocker or network error swallowed by a fire-and-forget
   call).
   Verify: temporarily wrap trackPurchase in try/catch with a
   console.error, or move it after navigate(), and see if the
   2% failure rate tracks with ad-blocker usage in analytics.

2. A stale closure captures an old orderId, so navigate()
   fires but to a route that doesn't match the current render,
   and the spinner state never gets torn down for the visible
   component.
   Why: useCheckoutMutation.ts builds the onSuccess callback
   inline in the hook; if it closes over state from an earlier
   render, isSubmitting on the *current* CheckoutPage instance
   never flips back to false.
   Evidence: this would explain "spinner never clears" without
   any error, since nothing crashes — it just updates the wrong
   instance's state.
   Verify: log the orderId inside onSuccess and compare it
   against the orderId the visible component is currently
   rendering for a few reproductions.

3. The confirmation route is behind a feature flag that
   fails open to "false" under a slow flag-fetch, so
   navigate() to /order/:id/confirmation silently redirects
   back to /checkout.
   Why: if the flag check runs on route entry and the flag
   service hasn't resolved yet, some flag clients default to
   off, which would look exactly like "never navigates" from
   the user's side.
   Evidence: would explain why it's intermittent and
   production-only, since the flag service round-trip is the
   kind of thing that behaves differently under production
   latency than in local dev.
   Verify: check flag-evaluation logs for the affected
   orderIds and see if evaluation timed out or defaulted.
```

Only after checking which of these actually holds up — say, confirming in analytics logs that the failures correlate with ad-blocker traffic, ruling out the stale closure and the flag theory — do you go back and ask AI to implement a fix for cause #1 specifically. That's a completely different, much smaller change than the one you'd get from asking "please fix the checkout spinner" cold.

---

## 10.4 Debugging Loop

Hypothesis-driven debugging isn't a single prompt trick — it's a loop, and the loop is what keeps a debugging session converging instead of thrashing. Each pass through the loop should either confirm a cause (move to fix) or rule one out (go back to hypothesis with one theory eliminated), so the search space shrinks every time instead of staying constant.

```text
Problem
 ↓
Hypothesis
 ↓
Evidence
 ↓
Experiment
 ↓
Fix
 ↓
Verify
```

Avoid the alternative, which looks productive because code keeps changing but isn't actually converging on anything:

```text
Error
 ↓
AI randomly changes code
 ↓
Maybe it works
```

The difference between the two loops isn't effort, it's direction. In the random-change loop, every iteration is independent of the last — AI has no way to know if attempt #2 is smarter than attempt #1, because neither one was tied to evidence about what's actually happening. In the hypothesis-driven loop, every iteration either eliminates a theory or confirms one, so even a "failed" experiment moves you forward by shrinking the remaining possibilities.

**Worked example, continued.** Running the checkout spinner bug through the loop:

```text
Problem:
  Checkout spinner never clears for ~2% of attempts,
  production only, started after PR #482.

Hypothesis:
  trackPurchase(orderId) throws inside onSuccess, before
  navigate() runs, so the navigation and spinner-clear
  never execute.

Evidence:
  No client error logged (consistent with a swallowed
  throw); regression timing lines up with PR #482;
  affected users skew toward known ad-blocker extensions
  in the analytics breakdown.

Experiment:
  Add a temporary console.error around trackPurchase and
  ship to a 1% canary; confirm the throw fires for the
  affected cohort and that it happens before navigate().

Fix:
  Move trackPurchase(orderId) after navigate(), and wrap
  it in its own try/catch so a future analytics failure
  can never block checkout again.

Verify:
  Re-run the canary; confirm navigation now succeeds for
  users with the throwing analytics call, and confirm
  trackPurchase still fires (wrapped) so analytics data
  isn't silently lost.
```

Compare that to the random-change version of the same bug: reordering `isSubmitting` state, adding a timeout to force-clear the spinner after five seconds, wrapping the whole `onSuccess` in a blanket `try/catch` that swallows the real error along with everything else. All three of those might make the *symptom* go away in a quick local test, and none of them tell you why it was happening — which means the underlying bug (an unguarded analytics call blocking navigation) is still there, waiting to resurface the next time someone adds another line to `onSuccess` without a try/catch.

---

## 10.5 Debug Through Documentation, Not Silent Patches

Section 10.2's "do not modify code yet" stops AI from anchoring on an unconfirmed theory mid-conversation. This section extends the same discipline one step further: once a cause **is** confirmed, don't let the fix exist only as a diff. Make AI write down, in a short and durable form, what the bug actually was, why it happened, and why the fix addresses the cause and not just the symptom — *before* or alongside implementing it, not as an afterthought nobody reads.

This matters for a reason that goes beyond tidiness. A diff answers "what changed." It does not answer "what did we believe was true that turned out to be false" — and that second question is exactly the one that prevents the same class of bug from recurring somewhere else in the codebase that has the identical wrong assumption baked in. Module 1 also gives this a concrete mechanical reason: if a debugging session spans more than one sitting, or gets handed to a teammate, the reasoning that lived only in a chat transcript is subject to exactly the context rot and recency decay described in Module 1.3–1.4. A short written root-cause note survives outside any single session's context window; a conversation does not.

Ask for this explicitly, the same way you'd ask for a plan before implementation (Module 8):

```text
Before implementing the fix, write a short root-cause note:

- What was actually happening (not the symptom — the mechanism)
- What assumption was wrong, and where it was made
- Why this fix addresses the cause, not just the symptom
- What else in the codebase might share the same wrong assumption

Then implement the fix.
```

**Worked example, continued.** For the checkout spinner bug, the root-cause note is short but does real work:

```text
Root cause: onSuccess assumed trackPurchase() could never fail in a
way that mattered. It called trackPurchase() before navigate(), with
no error handling, so a thrown/rejected analytics call silently
aborted the rest of onSuccess — including navigation and clearing
the spinner.

Wrong assumption: "fire-and-forget side calls inside a mutation's
onSuccess are safe to leave unguarded." True for calls that can't
throw synchronously or reject; false here, since trackPurchase can
throw when blocked by an ad blocker.

Why this fix addresses the cause: moving trackPurchase after
navigate() and wrapping it in its own try/catch means a future
failure in that call can no longer block the primary success path,
regardless of why it fails.

Where else this assumption might live: any other onSuccess/onSettled
handler in the codebase that calls an analytics or logging function
before its main side effect, unguarded. Worth a quick audit.
```

That last line is the payoff this section is really about — a diff alone would never have produced it, because a diff only describes the one instance that got caught. The note turns a single bug fix into a codebase-wide question.

---

## 10.6 Brownfield vs. Greenfield: Which Comes First, the Docs or the Code

Section 10.5 established *that* you should document root causes. This section addresses a question that comes up immediately once you try to apply it: when a bug reveals a wrong assumption, do you fix the documentation of that assumption first and then write the code — or implement the fix first and let the documentation catch up afterward? The answer depends on whether an authoritative description of the current design already exists and disagrees with reality, or whether none exists yet because you're still finding out what's true.

**Brownfield — an existing, documented assumption turned out to be wrong.** If the codebase has a README, an ADR, an inline comment, or even just a well-established convention everyone follows that describes how something is *supposed* to work, and the bug just proved that description is wrong or incomplete, **fix the documented understanding first, then implement against the corrected version.** The reasoning: if you patch the code first, you now have two disagreeing sources of truth — the code (which reflects what you just learned) and the doc (which still reflects the old, wrong belief) — and the next engineer, human or AI, who reads the doc before the code will inherit the same wrong assumption you just spent an hour disproving. Updating the doc first also forces you to state the corrected model precisely enough to implement against, the same way Module 4.1's contract-before-code discipline forces precision before implementation.

```text
Avoid:
  Patch the bug → ship it → docs still describe the old (wrong) behavior.

Prefer (brownfield):
  Confirm the cause (10.4) → update the doc/comment/ADR describing
  the affected assumption → implement the fix against the corrected
  doc → verify.
```

**Greenfield / exploratory — no established doc, or you're in genuinely uncharted territory.** If you're prototyping, spiking, or debugging a brand-new piece of code that has no prior documented design to be wrong about, writing documentation *before* the fix is premature — there's no established belief to correct yet, only a hypothesis you haven't confirmed. Here, the order from Sections 10.3–10.5 already has it right: form the hypothesis, verify it, implement the fix, confirm it works, *then* write the root-cause note and any doc update — because the note should describe what you now know is true, not what you guessed might be true before testing it.

```text
Avoid:
  Write a detailed design doc for how the new module "should" behave,
  based on an unverified hypothesis about the bug, before confirming
  anything.

Prefer (greenfield):
  Confirm the cause (10.4) → implement and verify the fix →
  write the root-cause note (10.5) now that it reflects reality,
  not a guess.
```

The distinguishing question to ask yourself before choosing: **"Does something authoritative already claim to describe this behavior, and did the bug just prove that claim wrong?"** If yes, that claim is actively misleading everyone who reads it until it's corrected — fix the doc first. If no — if this is new ground where the fix itself *is* the first real discovery of how the thing should behave — document after, once the fix is verified, so the doc records something true instead of something guessed.

**Worked example.** A team's `README.md` for their caching layer states: "cache entries are invalidated on any mutation to the related resource." A bug report shows a stale cache entry surviving a mutation. Debugging confirms the real behavior: invalidation only fires for mutations that go through the primary API client, not the secondary bulk-import path some features use. This is brownfield — the README made a specific claim, and the bug just proved it false for a whole category of writes. The correct order is: update the README to state the real, narrower invalidation rule first (forcing the team to actually decide whether that's the *intended* behavior or a second bug to fix), and only then implement whichever fix follows from that corrected understanding — either extending invalidation to the bulk-import path, or, if that's intentionally out of scope, documenting the exception clearly enough that the next engineer doesn't rediscover this the same way. Compare this to a brand-new experimental caching prototype with no README yet, where the same investigation would run hypothesis → verify → fix → *then* write the first version of that documentation, since there was no prior claim to have been wrong.

---

## Key Takeaways

* A debugging prompt is only as good as the context behind it — expected, actual, error, stack trace, relevant code, recent changes, environment, and what's already been tried. Missing pieces get filled in with guesses.
* Never open with "why doesn't this work?" It has no falsifiable claim in it, so AI has nothing to reason against except guesses.
* Ask for causes before asking for a fix, and require each cause to come with a reason, evidence, and a way to verify it — then verify it yourself before letting AI touch code.
* "Do not modify code yet" is as important in debugging as "do not write code" was in planning: once a patch exists, it anchors the conversation around a theory nobody confirmed.
* Run the loop — problem, hypothesis, evidence, experiment, fix, verify — and treat a ruled-out hypothesis as progress, not failure, because it shrinks the search space.
* The random-change loop (error → AI changes something → maybe it works) can *feel* fast, but it doesn't converge, and it tends to hide the real bug under a patch that masks the symptom.
* Don't let a confirmed fix exist only as a diff. Force a short root-cause note — mechanism, wrong assumption, why the fix addresses the cause, where else the same assumption might live — durable outside any one session's context window (Module 1).
* When a bug disproves something an existing doc, comment, or convention actually claimed (brownfield), fix that documented understanding *first*, then implement against it — two disagreeing sources of truth is worse than a slower fix. When there's no prior claim to correct (greenfield/exploratory), implement and verify first, then document what turned out to be true.

## Try It Yourself

1. Take a bug you're currently facing (or one you recently "fixed" without being fully sure why it went away). Write it up using the full 10.1 checklist — expected, actual, error, stack trace, relevant code, recent changes, environment, what's already been tried — even the parts that feel obvious. Compare the analysis AI gives you against a one-line "why doesn't this work?" version of the same bug, and note what the structured version caught that the vague one missed.
2. For that same bug (or your next one), run the full hypothesis-driven loop end to end: get the top 3 causes with evidence and verification steps, actually verify one before asking for a fix, and only then request the patch. Write down which of the 3 hypotheses turned out to be right, and whether it was the one you — or AI — would have guessed first.
3. For your next confirmed bug fix, write the Section 10.5 root-cause note before merging — even if nobody asked for it. Then check: is there an existing README, comment, or ADR that made a claim your bug just proved wrong (brownfield), or is this new ground (greenfield)? Handle the doc in the order Section 10.6 recommends, and note whether skipping straight to the diff (your usual habit) would have left a misleading doc behind or documented an unverified guess.

---

*[← Previous: AI as a Coding Partner](./02-ai-coding-partner.md) | [Next: AI Refactoring →](./04-ai-refactoring.md)*
