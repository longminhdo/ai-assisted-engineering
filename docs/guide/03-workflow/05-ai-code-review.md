# Module 12 — AI Code Review

*[← Previous: AI Refactoring](./04-ai-refactoring.md) | [Next: Verification →](./06-verification.md)*

## Why this matters

Ask AI "is this checkout code good?" and you will almost always get "Looks good! The implementation is clean and follows best practices" — a rubber stamp that tells you nothing, because a validator's default posture is agreement, not scrutiny. Ask instead "try to prove this checkout flow is wrong — find inputs or interactions that produce an incorrect total," and the same model will often walk straight to the bug: what happens if the user changes quantity while the promo-code request is still in flight, and the stale response lands after the newer quantity has already recalculated the total. Same code, same model, radically different value — the only thing that changed is what you asked it to do with its attention. This module is about learning to ask for the second kind of answer on purpose, instead of getting the first kind by accident.

## Objective

Teach yourself not only to ask AI to write code, but also to **challenge the code** — yours, a teammate's, or AI's own — the way a skeptical senior engineer would in a real review, rather than the way a compliance checkbox gets ticked.

---

## 12.1 Review Prompt

A plain "review this" invites a plain answer, because you haven't told the model what a real review actually checks. Senior engineers reviewing frontend code aren't scanning for typos — they're checking a specific, learned list of failure categories: does the business logic match the requirement, is state owned by the right layer, can two async operations race, does a closure capture a value that's since gone stale, will this re-render more than it needs to, does the type signature lie about what can actually happen at runtime, can a screen reader user operate this, what happens when the network call fails, what happens at the edges (empty list, first item, last item, zero, negative, very large), and does this match how the rest of the codebase already solves this problem. Naming that list explicitly in the prompt is what turns "review this" from a vague request into a real review — the model does not spontaneously enumerate all ten categories on its own, but it applies them reliably once you supply them.

The curriculum's review prompt:

```text
Review this implementation as a critical senior frontend engineer.

Look for:

1. Incorrect business logic
2. State ownership problems
3. Race conditions
4. Stale closures
5. Unnecessary re-renders
6. TypeScript issues
7. Accessibility problems
8. Error handling gaps
9. Edge cases
10. Violations of existing project conventions

Do not assume the implementation is correct.
Try to find reasons it could fail.
```

Two lines do most of the work here, and both are easy to skip by accident. "As a critical senior frontend engineer" sets the posture — reviewer, not author — which matters because a model asked to review its own recent output has a mild bias toward defending it. "Do not assume the implementation is correct" closes the door on the laziest possible answer: agreement. Without that line, a model under time pressure (figuratively — token pressure) will often summarize what the code does instead of checking whether what it does is right.

**Worked example — checkout promo code.** Suppose a teammate opens a PR with this hook powering the checkout page's promo-code field:

```tsx
function usePromoCode(cartTotal: number) {
  const [code, setCode] = useState("");
  const [discount, setDiscount] = useState(0);

  const applyCode = async (value: string) => {
    setCode(value);
    const res = await validatePromoCode(value, cartTotal);
    setDiscount(res.discountAmount);
  };

  return { code, discount, applyCode, finalTotal: cartTotal - discount };
}
```

Run the ten-point checklist prompt against it and you get a review like this back, unprompted beyond the checklist itself:

```text
3. Race conditions: applyCode captures cartTotal in its closure at
   call time. If the user changes quantity (changing cartTotal)
   while validatePromoCode is still in flight, the response applies
   a discountAmount that was calculated against the old total —
   finalTotal becomes wrong until the user re-applies the code.

4. Stale closures: same root cause as above — applyCode is
   recreated on every render with the current cartTotal, but any
   in-flight call already closed over the previous one.

8. Error handling gaps: no catch around validatePromoCode. A 4xx
   for an invalid code will throw and leave discount at its last
   value, silently applying a stale discount instead of surfacing
   "invalid code" to the user.

9. Edge cases: no guard against calling applyCode with an empty
   string, and no handling for the promo code being valid but
   cartTotal dropping to $0 after a cart edit.
```

None of that is exotic — it's the kind of bug that survives code review by a human because the diff looks clean, and survives QA because the race window is narrow. It shows up in production as "the discount was wrong for one customer" tickets that are hard to reproduce. That's exactly the class of bug this checklist is built to catch before merge, not after.

Compare that against what "is this code good?" tends to produce for the same hook: a paragraph about how the hook is "concise and uses React state appropriately," with no mention of the race condition at all. The checklist isn't a nicety — it's the difference between catching this bug in review and catching it in an incident channel.

---

## 12.2 Try to Break It

The review-prompt checklist works because it directs *what* to look for; "try to break it" changes *who the model is pretending to be* while it looks. A reviewer's job is to find problems that exist; an adversary's job is to actively construct scenarios where the code fails — a stronger and more specific framing that pushes the model from "checking a list" toward "hunting for a counterexample," which is a more natural fit for how race conditions and edge cases actually get found: not by category, but by trying combinations of inputs and timing until one breaks.

```text
Try to prove this implementation is wrong.

Find inputs, states or user interactions
that could cause incorrect behavior.
```

This reframes the model from validator to adversary — its job is no longer to check whether the code looks reasonable, it's to actively construct a scenario where the code is provably wrong. That's a small wording change with a real effect: "look for problems" lets a model stop at the first plausible-sounding issue, while "prove this is wrong" keeps it searching until it has a concrete failing scenario, or none.

**Worked example — settings page autosave.** Consider a settings page where each field autosaves on blur, debounced by 500ms, with a "Saved" indicator:

```tsx
function useAutosaveField(fieldName: string, initialValue: string) {
  const [value, setValue] = useState(initialValue);
  const [status, setStatus] = useState<"idle" | "saving" | "saved">("idle");

  const debouncedSave = useMemo(
    () =>
      debounce(async (v: string) => {
        setStatus("saving");
        await api.updateSetting(fieldName, v);
        setStatus("saved");
      }, 500),
    [fieldName]
  );

  const onChange = (v: string) => {
    setValue(v);
    debouncedSave(v);
  };

  return { value, status, onChange };
}
```

Fed this hook, "try to prove this implementation is wrong" surfaces a scenario a category checklist might not phrase as sharply:

```text
Scenario: user edits the field, waits 300ms (save not yet fired),
edits it again to a different value, then immediately navigates
away from the settings page.

- The debounced save fires 500ms after the *second* edit, but the
  component has already unmounted. setStatus("saving") and
  setStatus("saved") both run against an unmounted component,
  which either warns or silently no-ops depending on React version.

Scenario: user edits Field A, then within 500ms edits Field B, then
returns to Field A and edits it a third time before either save
has completed.

- Because debouncedSave is memoized per fieldName, Field A's first
  and third edits share one debounce timer. The third edit's save
  call carries the correct latest value, so no data is lost here —
  but status flips to "saved" only for the last call, so if the
  first save's request happens to resolve *after* the third one
  (out-of-order responses), the "Saved" indicator briefly shows
  before an earlier, now-stale response resolves and could
  overwrite status back to a misleading state.

Scenario: two different fields, A and B, autosave within the same
500ms window. If updateSetting is not atomic per-field on the
backend, a caller that expects a full-settings PATCH silently drops
whichever field's PATCH occupies the same debounce cycle.
```

Whether every one of those pans out depends on details of `api.updateSetting` you'd confirm in the actual codebase — but that's the point of this prompt: it doesn't hand you a verdict, it hands you specific, testable scenarios to check, which is a far more actionable output than "the debounce logic looks fine." A prompt that only asks "review this for problems" tends to stop at "there's no cleanup for the debounce on unmount" and not go looking for the out-of-order-response and unmount-race scenarios above, because those require actively trying to construct a failing timeline rather than pattern-matching against a known issue.

---

## 12.3 Review One Dimension at a Time

A review prompt that asks for everything at once gets you a little of everything and not much of anything — the model's attention has to spread across ten-plus concerns in one pass, and broad, general concerns (does this look reasonable) tend to crowd out narrow, specific ones (does this closure capture a stale value) unless you force the narrow one into focus by asking for it alone. This is the same principle Module 7 taught for writing requests in general — one meaningful concern per prompt — applied specifically to review.

Avoid:

```text
Is this code good?
```

Instead, ask one question at a time:

```text
Review only for performance.
```

```text
Review only for accessibility.
```

```text
Review only for race conditions.
```

**Worked example — notifications panel.** Take a notifications panel component: it opens in a popover, fetches unread items on open, polls every 30 seconds while open, and lets the user mark items as read. Run three separate, narrowly-scoped reviews against the same component and each one earns its keep by surfacing something the others wouldn't have prioritized:

```text
Review only for performance.
```
```text
The panel subscribes to the entire NotificationsContext, so every
poll tick re-renders the bell icon, the panel, and every
NotificationItem — including items whose read state didn't change.
Memoize NotificationItem and select only the fields each subscriber
needs instead of the whole context value.
```

```text
Review only for accessibility.
```
```text
Opening the panel does not move focus into it, so keyboard and
screen-reader users don't know it opened. New notifications arriving
via poll aren't announced (no aria-live region), and there's no
visible focus trap — Tab can escape the popover into content behind
it while it's open.
```

```text
Review only for race conditions.
```
```text
Marking an item as read fires a PATCH and optimistically updates
local state. If the 30-second poll's GET response was already in
flight and resolves after the optimistic update, its (still-unread)
copy of that item overwrites the optimistic read state — the item
flips back to "unread" a moment after the user cleared it.
```

Ask a single "is this panel good?" instead, and a realistic answer mentions maybe one of those three — usually whichever is most visually obvious in the code — and calls it done. Three focused passes cost more of your time to run than one broad pass, but each one returns a finding dense enough to act on directly, instead of a paragraph you have to squint at to extract anything from.

> **In Claude Code, specifically.** A bundled `/code-review` skill runs exactly this pattern for correctness bugs: it reviews the current diff in a fresh subagent and returns findings to your session, without you having to draft the "review as a critical senior engineer" prompt yourself. The reason it runs as a subagent rather than in the same session that wrote the code matters — per the [official best-practices guide](https://code.claude.com/docs/en/best-practices#add-an-adversarial-review-step), *"a reviewer running in a fresh subagent context sees only the diff and the criteria you give it, not the reasoning that produced the change, so it evaluates the result on its own terms."* The same guide also warns about the failure mode this invites: a reviewer told to find gaps will usually report some even when the work is sound — tell it to flag only what affects correctness or the stated requirements, and treat the rest as optional, or you'll chase phantom findings into the over-engineering Module 17 warns against.

---

## Key Takeaways

* A validator's default answer is agreement — "is this good?" reliably produces "looks good," which tells you nothing. Name the failure categories you actually want checked, or don't expect them to be checked.
* The ten-point review checklist (business logic, state ownership, race conditions, stale closures, re-renders, TypeScript, accessibility, error handling, edge cases, convention violations) is your default lens for reviewing any non-trivial frontend change — yours, a teammate's, or AI's own.
* "Do not assume the implementation is correct" is not filler — it's the line that keeps the review from collapsing into a summary of what the code does.
* "Try to prove this is wrong" reframes the model from checking a list to hunting for a concrete failing scenario — a stronger posture that tends to surface race conditions and timing bugs a category checklist states more generically.
* Split broad reviews into single-dimension passes (performance only, accessibility only, race conditions only) when you need depth on a specific concern — a component with real problems in three dimensions will not get all three from one general request.
* None of this replaces your own judgment on the findings — AI proposes plausible failure scenarios, you're the one who confirms which are real against the actual backend, browser, and user behavior.

## Try It Yourself

1. Take a real PR you have open right now (yours or a teammate's, before or after human review). Run the ten-point checklist prompt against the diff, then separately run "try to prove this implementation is wrong" against the same diff. Compare the two outputs: did the adversarial prompt find anything — a race condition, a stale closure, an edge case — that the checklist missed or stated too generically to act on?
2. Pick one component from that same PR and run three single-dimension reviews against it: performance only, accessibility only, and one more dimension of your choice. Note whether each pass surfaced something distinct enough that a single "review this" prompt would likely have buried or skipped it.

---

*[← Previous: AI Refactoring](./04-ai-refactoring.md) | [Next: Verification →](./06-verification.md)*
