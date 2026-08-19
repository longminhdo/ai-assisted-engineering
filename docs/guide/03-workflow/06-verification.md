# Module 13 — Verification

*[← Previous: AI Code Review](./05-ai-code-review.md) | [Next: Frontend-Specific AI Review →](../04-frontend-quality/01-frontend-specific-review.md)*

## Why this matters

Picture a PR that looks clean by every signal you normally check: TypeScript is green, ESLint is green, the diff is small, and the AI-written summary reads like it knows exactly what it did. It ships. Two days later, support tickets start coming in from users on hotel wifi — the checkout button doesn't disable on click, so a slow network turns one purchase into three duplicate orders. Nothing in the toolchain caught it, because nothing in the toolchain was built to catch it: `tsc` checks types, not timing; ESLint checks patterns, not behavior under a flaky connection. AI is exceptionally good at producing code that satisfies every static check while still being wrong about something none of those checks can see. This module is about the layers of verification that sit between "it compiles" and "it's correct" — and why skipping them is the single most common way AI-assisted work turns into a production incident.

## Objective

Core principle:

> **Generated code is not verified code.**

Passing type-check, lint, and even a glance-read of the diff is the *minimum bar for looking at the code*, not evidence that it works. This module walks through the full verification pyramid — from the cheapest automated checks to the runtime behavior only a human can observe — and gives you a worked example at every layer showing a real bug that layer, and only that layer, would catch.

---

## 13.1 Verification Pyramid

AI does not replace verification — it changes what you're verifying against. When you write code yourself, you've already internalized dozens of assumptions while typing it (this list is paginated, this button needs a loading state, this field can be null). When AI writes it, those assumptions might be right, might be silently wrong, or might not exist at all — the code just has to look plausible enough to compile and pass a skim. The pyramid below orders your checks from cheapest and fastest at the bottom to most expensive and highest-signal at the top:

```text
                 Human Review
                      ▲
                 Runtime Test
                      ▲
                    E2E
                      ▲
                  Unit Test
                      ▲
                     Lint
                      ▲
                Type Checking
```

Read it bottom-up, because that's the order you should actually run it in. Type checking and lint cost you seconds and run on every keystroke in most editors — there's no reason to let a `tsc` error or an obvious hook violation survive past that layer, so let the cheapest tools catch the cheapest mistakes first. Unit tests cost more (you have to write or generate them, then run them) but they isolate one function or component at a time, so a failure tells you almost exactly where the bug is. E2E tests cost more still — they need a running app, real navigation, real network calls — but they're the first layer that actually exercises the thing users experience: a full action from click to rendered result. Human review and runtime testing sit at the top because they're the most expensive in engineer-time and the hardest to automate away, but they're also the only layers that catch the failure modes nothing below them is even designed to look for: is this the right feature, does this feel right under a bad network, would a user actually understand this state.

The mistake most teams make with AI-generated code is inverting this pyramid — they read the diff once (top layer), see that it type-checks (bottom layer), and skip everything in between. Working bottom-up instead means you never spend expensive human attention on a bug a five-second lint run would have caught for free, and you never let a wrong type slip past to become a "why does this crash in production" incident that costs an afternoon of debugging instead of one red squiggle in your editor.

---

## 13.2 TypeScript

TypeScript is your fastest and cheapest verification layer, and also the one AI is most likely to satisfy without actually being correct — because "make the type-checker happy" is a much lower bar than "handle this data correctly." A model under pressure to produce working-looking code will reach for `any`, a non-null assertion (`!`), or an overly permissive generic before it will stop and ask what the real shape of your data is. None of that fails compilation. All of it fails at runtime, later, on someone else's machine, with a much worse error message than the type-checker would have given you.

Check for:

* Type errors
* Incorrect generics
* Unsafe narrowing
* `any`
* Incorrect API contracts
* Null/undefined handling

**Worked example — settings page.** Say you asked AI to add a "save" handler to a user settings form that patches a subset of fields to `PATCH /api/users/:id/settings`. It hands back:

```ts
async function saveSettings(userId: string, changes: any) {
  const res = await fetch(`/api/users/${userId}/settings`, {
    method: "PATCH",
    body: JSON.stringify(changes),
  });
  return res.json();
}
```

This compiles cleanly and even runs correctly in the happy path during manual testing. But `changes: any` means nothing stops a caller from passing `{ theam: "dark" }` (typo'd key) or `{ notifications: "yes" }` (string instead of boolean) — TypeScript has been told to stop checking the one place a mistake is most likely to happen, right at the API boundary. The fix isn't a bigger effort, just a more specific type:

```ts
type SettingsPatch = Partial<Pick<UserSettings, "theme" | "notifications" | "timezone">>;

async function saveSettings(userId: string, changes: SettingsPatch) {
  const res = await fetch(`/api/users/${userId}/settings`, {
    method: "PATCH",
    body: JSON.stringify(changes),
  });
  if (!res.ok) throw new Error(`Failed to save settings: ${res.status}`);
  return res.json() as Promise<UserSettings>;
}
```

Notice this fix also caught a second bug for free: the original never checked `res.ok`, so a 400 from a typo'd key would have been silently parsed as success. That's exactly the kind of thing a real type for `changes` forces you to think about, and `any` lets you skip thinking about entirely.

---

## 13.3 Lint

Lint exists to enforce the conventions that are true for your codebase but invisible to a general-purpose model — the specific hook rules, the naming scheme, the "we don't use default exports here" decisions that were never going to show up just from reading a function signature. AI tends to write code that looks like idiomatic React in general, which is not the same as code that matches your project's specific rules, especially around effect dependencies where "looks right" and "is right" diverge constantly.

Check for:

* Hook misuse
* Unused code
* Incorrect dependency arrays
* Project convention violations

**Worked example — notifications panel.** Continuing the notifications panel from Module 8: suppose a later request asks AI to auto-mark a notification as read once it's been visible for two seconds. It produces:

```tsx
function NotificationItem({ notification, markAsRead }: Props) {
  useEffect(() => {
    const timer = setTimeout(() => markAsRead(notification.id), 2000);
    return () => clearTimeout(timer);
  }, []);

  return <li>{notification.text}</li>;
}
```

This renders correctly on first mount and looks fine in a quick read. `react-hooks/exhaustive-deps` flags it immediately: the effect closes over `notification.id` and `markAsRead`, neither of which is in the dependency array. In practice this means if the same `NotificationItem` instance ever gets reused for a different notification (React reordering a virtualized list, for example) the timer fires and marks the *original* notification's ID as read, not the one currently displayed — a bug that's invisible in manual testing because it only shows up when the list reorders while a timer is still pending. The lint rule doesn't know any of that context; it doesn't need to. It just knows the effect claims no dependencies while reading two, and that mismatch is enough to catch a bug that would otherwise require someone to notice a wrong notification getting marked read under specific list-reordering conditions.

---

## 13.4 Tests

AI can generate tests as easily as it generates implementation code, and that's exactly the problem: a model asked to "add tests for this function" is optimizing for tests that pass and that plausibly relate to the function, not necessarily tests that would fail if the function were wrong. It's trivial to end up with 100% line coverage and zero behavioral verification, because a test can execute every line of a function while asserting nothing about what those lines were supposed to accomplish.

> **Does the test actually verify behavior?**

Do not accept tests simply because they increase coverage.

**Worked example — checkout flow.** Suppose AI adds this test for a `calculateOrderTotal` function that applies a discount code:

```ts
it("calculates order total", () => {
  const result = calculateOrderTotal(cartItems, "SAVE10");
  expect(result).toBeDefined();
});
```

This test runs every line of `calculateOrderTotal`, reports green, and adds to your coverage number. It also would not fail if the discount logic were deleted entirely and the function returned `0`, or if `SAVE10` silently applied 100% off instead of 10% — `toBeDefined()` is true for almost any return value. The question from the curriculum — does this actually verify behavior — has an obvious "no" here. A test that verifies behavior asserts on the actual value the business logic is responsible for getting right:

```ts
it("applies a 10% discount to the cart subtotal", () => {
  const cartItems = [{ price: 100, qty: 2 }]; // subtotal = 200
  const result = calculateOrderTotal(cartItems, "SAVE10");
  expect(result.total).toBe(180);
  expect(result.discountApplied).toBe(20);
});

it("ignores an invalid discount code without throwing", () => {
  const cartItems = [{ price: 100, qty: 1 }];
  const result = calculateOrderTotal(cartItems, "NOT-A-REAL-CODE");
  expect(result.total).toBe(100);
});
```

The second version would have caught a real bug: if someone later "simplifies" the discount lookup and it throws on an unrecognized code instead of ignoring it, this test fails and the first one wouldn't have noticed either way.

---

## 13.5 E2E

Unit tests verify a function or component in isolation, which is exactly their strength and exactly their blind spot: they can't tell you whether the pieces actually connect correctly once wired together through real state, real routing, and a real (or realistically mocked) API. E2E tests are the layer that exercises the full chain a user actually depends on:

```text
User action
→ UI
→ API
→ State
→ UI result
```

E2E tests provide an important source of truth precisely because they don't care how the code is organized internally — they only care whether the observable outcome is correct, which is the same thing your users care about.

**Worked example — checkout flow.** Say unit tests confirm `calculateOrderTotal` is correct and a separate unit test confirms the `PaymentForm` component renders a success message on a mocked success response. Both pass. Wire them together in the real app and here's a bug unit tests structurally cannot see: clicking "Place Order" calls the checkout mutation, but the button isn't disabled while the request is in flight, so a user who clicks twice — impatient, or on a slow connection where nothing visibly happens for a second — fires two `POST /api/orders` requests and gets charged twice. Neither unit test above touches the button's disabled state or the timing between click and response; that connection only exists in the full flow:

```text
User clicks "Place Order"
→ PaymentForm calls placeOrder()
→ POST /api/orders (slow — 2s)
→ button remains clickable
→ user clicks again
→ second POST /api/orders
→ two orders created, one charge doubled
```

An E2E test that drives the real button, throttles the network, and asserts on the number of orders created is the only layer of the pyramid positioned to catch this, because it's the only layer where "the button was clickable during the request" is even an observable condition.

---

## 13.6 Runtime Verification

Compilation does not mean correctness — and this is where the pyramid's top layers earn their expense. A model that has never run your app has no way to know what it looks like on a throttled connection, what happens when an array the code assumed was non-empty comes back empty, or what a screen reader announces when a modal opens. These are not edge cases in the dismissive sense; for real users they are Tuesday. AI-generated code very reliably handles the state it was shown in the prompt (usually the happy path) and very unreliably handles every other state, because it was never asked to think about them.

Check:

* Loading
* Empty
* Error
* Slow network
* Retry
* Duplicate clicks
* Refresh
* Navigation
* Permissions
* Responsive behavior

**Worked example — notifications panel.** Take the notifications panel one more time. It works perfectly when you open it with 12 notifications already loaded from a fast local API. Actually clicking through the states above surfaces problems a compiler was never going to see:

* **Empty** — zero notifications renders an empty `<ul>` with no message, so first-time users see a panel that looks broken rather than "you're all caught up."
* **Slow network** — throttling to Slow 3G shows the panel opening on a permanent blank space with no spinner, because the loading state was never wired to a visible skeleton or spinner.
* **Duplicate clicks** — clicking "mark all as read" twice in quick succession fires the mutation twice; the second call 404s on notifications the first call already cleared, and that unhandled rejection surfaces as a console error a user never sees but that pollutes your error tracking.
* **Refresh** — reloading the page while the panel is open loses the open/closed state, which may be fine, but if it was supposed to persist per Module 8's local-state decision, this is where you'd catch that it doesn't.
* **Permissions** — a user without notification access still sees the bell icon, because nothing in the generated component checked a permission flag before rendering it.

None of these break the build. None of them would show up in a code review that only reads the diff. All of them ship straight to production if runtime verification is the layer you skip — which is exactly why it sits at the top of the pyramid instead of being treated as optional polish.

> **In Claude Code, specifically.** The [official best-practices guide](https://code.claude.com/docs/en/best-practices#give-claude-a-way-to-verify-its-work) frames this whole module as one instruction: *"give Claude a check it can run: tests, a build, a screenshot to compare. It's the difference between a session you watch and one you walk away from."* Without a check, "looks done" is the only signal available and you become the verification loop yourself. Three ways to make that check actually gate the work instead of being optional: ask Claude to run it and iterate in the same prompt (works today, no setup); set it as a [`/goal` condition](https://code.claude.com/docs/en/goal) that a separate evaluator re-checks after every turn; or write a [Stop hook](https://code.claude.com/docs/en/hooks#stop) that runs the check as a script and blocks the turn from ending until it passes — deterministic, not advisory. For a second opinion that isn't biased toward code it just wrote, a fresh subagent reviewing only the diff and your criteria is more reliable than asking the same session to grade its own work; see the Advanced Track's [Anatomy of a Claude Project §1.4](../07-advanced-track/01-anatomy-of-a-claude-project.md) on subagent containment.

---

## Key Takeaways

* Generated code is not verified code — passing type-check and lint is the floor, not the finish line.
* Work the pyramid bottom-up: let the cheap, fast layers (types, lint) catch what they can before spending expensive human or E2E time on the same class of bug.
* A test that passes without asserting a specific, meaningful value (`toBeDefined()`, "it runs without throwing") is coverage theater, not verification — ask "would this test fail if the logic were wrong?"
* E2E tests catch the bugs that only exist in the seams between correct pieces — a correct total-calculator and a correct-looking form can still combine into a double-charge bug neither unit test could see.
* Runtime verification (loading, empty, error, slow network, duplicate clicks, refresh, permissions) is where most AI-generated code actually fails, precisely because AI defaults to the happy path it was shown in the prompt.
* If you only have time for one extra check on an AI-generated PR, make it the runtime walkthrough — it's the layer with the highest ratio of production incidents caught to minutes spent.

## Try It Yourself

1. Pick a piece of AI-generated code you accepted into a PR in the last two weeks — something you reviewed by reading the diff and confirming it compiled and passed lint. Run it through the rest of the pyramid you skipped: read its tests and ask, for each one, "would this fail if the logic were subtly wrong?" Then click through it manually against the runtime checklist in 13.6 (loading, empty, error, slow network, duplicate clicks, refresh). Write down every gap you find — even "none, it held up" is a useful data point about how much you can trust that kind of change going forward.
2. Take one AI-generated test file from your codebase and, without looking at the implementation, try to guess what bug each test would catch if introduced. If you can't answer for a given test, that test is verifying that the code ran, not that it's correct — rewrite it to assert a specific value, then deliberately break the implementation to confirm the test actually goes red.

---

*[← Previous: AI Code Review](./05-ai-code-review.md) | [Next: Frontend-Specific AI Review →](../04-frontend-quality/01-frontend-specific-review.md)*
