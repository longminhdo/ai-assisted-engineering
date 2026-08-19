# Module 15 — AI for Testing

*[← Previous: Frontend-Specific AI Review](./01-frontend-specific-review.md) | [Next: AI for Documentation & Knowledge →](./03-ai-for-documentation.md)*

## Why this matters

Picture a PR where AI generated twenty tests for a new feature. All twenty pass. Coverage jumps from 61% to 89%, the CI badge turns green, and the PR gets an easy approval because "it's well tested" — the number says so. Three weeks later a production bug ships: a discount code that should cap at 50% off instead stacks with a second promo and gives a customer a negative-total order. Nobody catches it in review, because the review trusted the coverage number instead of reading what the tests actually asserted. It turns out four of those twenty tests exercised the exact function where the bug lived — and every one of them asserted that the function "returned an object" or "didn't throw," never what the object's values actually were. The tests ran every line of the buggy code and verified none of its behavior. This is the single most common failure mode of AI-assisted testing: it is extremely good at producing tests that pass, and only incidentally good at producing tests that would fail if your code were wrong.

## Objective

Use AI to improve test quality — not just test quantity. That means using it to generate meaningful coverage from existing patterns, to surface the edge cases you didn't think to ask about, and to critically review test suites (including its own) for the kind of green-but-worthless tests described above.

---

## 15.1 Generate Tests

Asking AI to "write tests for this" without more context tends to produce tests that mirror the happy path you showed it and stop there — because the happy path is usually all that's visible in the file you pasted in. The fix is the same context-engineering move from earlier modules: point AI at your project's actual testing conventions (render helpers, mock setup, naming scheme, assertion style) so the output looks like it was written by someone on your team, and explicitly name the categories of behavior you want covered instead of leaving "cover it" open to interpretation.

```text
Generate tests based on the existing testing pattern.

Cover:
- happy path
- empty state
- validation error
- API error
- boundary cases
```

The phrase "based on the existing testing pattern" is doing real work in that prompt — it's the difference between AI inventing its own testing style (a different mocking library than the rest of your suite uses, a different assertion helper, `describe`/`it` nesting that doesn't match your file) and AI producing a test file a teammate could review in thirty seconds because it looks like everything else in the directory. Always point it at a real sibling test file rather than describing your conventions from memory — descriptions drift, files don't.

**Worked example — promo-code form.** Say you have a `PromoCodeForm` component that submits a code, shows a loading state, and displays either the resulting discount or a rejection message. You give AI the component, a sibling test file (`CheckoutForm.test.tsx`) as the pattern reference, and the prompt above. It generates:

```tsx
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { PromoCodeForm } from "./PromoCodeForm";
import { server } from "../../test/mocks/server";
import { rest } from "msw";

describe("PromoCodeForm", () => {
  it("applies a valid promo code and shows the discount", async () => {
    render(<PromoCodeForm cartSubtotal={100} />);
    await userEvent.type(screen.getByLabelText(/promo code/i), "SAVE10");
    await userEvent.click(screen.getByRole("button", { name: /apply/i }));

    expect(await screen.findByText(/10% off applied/i)).toBeInTheDocument();
    expect(screen.getByText(/\$90\.00/i)).toBeInTheDocument();
  });

  it("shows an empty-state prompt before any code is entered", () => {
    render(<PromoCodeForm cartSubtotal={100} />);
    expect(screen.getByText(/have a promo code\?/i)).toBeInTheDocument();
  });

  it("shows a validation error for an empty submission", async () => {
    render(<PromoCodeForm cartSubtotal={100} />);
    await userEvent.click(screen.getByRole("button", { name: /apply/i }));
    expect(await screen.findByText(/enter a code/i)).toBeInTheDocument();
  });

  it("shows an error message when the API rejects the code", async () => {
    server.use(
      rest.post("/api/promo/validate", (_req, res, ctx) =>
        res(ctx.status(400), ctx.json({ message: "Code expired" }))
      )
    );
    render(<PromoCodeForm cartSubtotal={100} />);
    await userEvent.type(screen.getByLabelText(/promo code/i), "OLDCODE");
    await userEvent.click(screen.getByRole("button", { name: /apply/i }));
    expect(await screen.findByText(/code expired/i)).toBeInTheDocument();
  });

  it("caps the discount at the cart subtotal for a boundary case", async () => {
    render(<PromoCodeForm cartSubtotal={5} />);
    await userEvent.type(screen.getByLabelText(/promo code/i), "SAVE100");
    await userEvent.click(screen.getByRole("button", { name: /apply/i }));
    expect(await screen.findByText(/\$0\.00/i)).toBeInTheDocument();
  });
});
```

This is a genuinely good first draft — it uses your project's MSW mock server the same way `CheckoutForm.test.tsx` does, and it maps directly onto every category from the prompt: happy path, empty state, validation error, API error, and a boundary case (a discount that would otherwise push the total negative). That last test is exactly the kind of case that would have caught the bug from the intro if it had existed. Treat this as a strong starting point, not a finished suite — you still read every assertion and confirm each one checks a specific, meaningful value (it does here: `$90.00`, `$0.00`, specific error text) rather than something vague. Section 15.3 covers how to audit that systematically.

---

## 15.2 Discover Edge Cases

Generation prompts anchor on whatever context you handed over, which is usually the component and maybe one API shape — so the tests that come back are only as imaginative as the input was. Edge-case discovery is a deliberately separate step: instead of asking AI to write tests, you ask it to think about what could go wrong that *isn't* in the file yet. This works because listing failure modes is a different cognitive task than writing passing assertions, and separating the two steps stops AI from quietly skipping cases it would have to admit it doesn't have a good test for.

```text
List edge cases that are currently not covered.

Focus on:
- empty arrays
- null values
- duplicate values
- rapid user interactions
- network failures
- stale data
- permission changes
```

Run this against your actual test file, not just the implementation — AI needs to see what's already covered to tell you what's missing, otherwise it re-lists cases you've already handled and calls it new information.

**Worked example — notifications feed.** You have a `NotificationsPanel` with three existing tests: renders a list, marks a single notification as read, and shows an empty state. You paste the component, the API hook it uses, and the existing test file, then run the prompt above. AI returns:

```text
Not currently covered:

- empty arrays: covered (empty state test exists), but not the transition
  FROM a populated list TO empty (e.g. user reads the last notification —
  does the empty-state message appear immediately, or does the panel just
  render a blank <ul>?)
- null values: a notification with `actionUrl: null` — does clicking it
  navigate to "null" or silently no-op?
- duplicate values: two notifications with the same `id` from a paginated
  API response (page overlap on fast scroll) — does React key collision
  cause one to silently not render, or does "mark as read" affect both?
- rapid user interactions: clicking "mark all as read" twice before the
  first request resolves — does the second call error on already-read
  notifications, and does that surface to the user?
- network failures: the panel fetches notifications on open; what renders
  if that fetch fails — nothing currently distinguishes "loading" from
  "errored" from "empty"?
- stale data: panel is opened, left open in a background tab for an hour,
  then a notification is clicked — is the read-state mutation sent against
  data that's since changed server-side (e.g. notification was deleted)?
- permission changes: user's notification access is revoked mid-session
  (e.g. removed from a workspace) — does a fetch that returns 403 crash
  the panel, show nothing, or show a misleading empty state?
```

Not every item on this list deserves a test today — that judgment call stays with you. The duplicate-`id` and stale-data cases might genuinely be rare enough to defer with a tracked follow-up. But the populated-to-empty transition and the 403-vs-empty distinction are cheap to test and describe real, likely user paths, so those get written up using the 15.1 prompt against this specific list. The value of this step isn't "test everything AI lists" — it's that you now have a documented set of gaps to make a deliberate decision about, instead of gaps nobody knew existed until a user hit one.

---

## 15.3 Review Test Quality

A test suite that's green tells you the tests ran without throwing — it does not tell you the tests would catch a regression. Reviewing AI-written tests (or your own) needs the same skepticism you'd apply to reviewing implementation code, because the two failure modes that make tests worthless are both things AI produces readily under time pressure: asserting on *how* the code works internally instead of *what* it produces, and assertions loose enough to pass regardless of whether the logic is right.

```text
Review these tests.

Identify tests that:
- test implementation details
- have weak assertions
- duplicate coverage
- do not test meaningful behavior
```

Run this as its own pass, separate from generating the tests in the first place — the same model that just wrote a weak assertion will often catch it immediately when asked to review rather than author, because reviewing puts it in a different, more critical frame than "produce something that passes."

**Worked example — settings page.** A `SettingsPage` test file accumulated over a few PRs. You run the review prompt against it and get back:

```tsx
// 1. Implementation detail: asserts on internal hook call, not on what
//    the user sees or what request goes out.
it("calls the settings mutation on save", () => {
  const mutateSpy = jest.spyOn(useSaveSettings(), "mutate");
  render(<SettingsPage />);
  fireEvent.click(screen.getByRole("button", { name: /save/i }));
  expect(mutateSpy).toHaveBeenCalled();
});
// Problem: passes even if `mutate` is called with the WRONG payload —
// a typo'd field name or a stale value would still satisfy this test.
// It verifies a function was called, not that saving actually works.

// 2. Weak assertion: coverage without a meaningful check.
it("renders the settings form", () => {
  render(<SettingsPage />);
  expect(screen.getByTestId("settings-page")).toBeDefined();
});
// Problem: `toBeDefined()` on a root container passes for nearly any
// render output, including a form that's silently missing half its
// fields. It would not fail if the "notifications" toggle disappeared
// entirely.

// 3. Duplicate coverage: same behavior as test 4, different wording.
it("updates the theme setting", async () => {
  render(<SettingsPage />);
  await userEvent.click(screen.getByLabelText(/dark mode/i));
  await userEvent.click(screen.getByRole("button", { name: /save/i }));
  expect(await screen.findByText(/saved/i)).toBeInTheDocument();
});
it("shows a save confirmation after saving", async () => {
  render(<SettingsPage />);
  await userEvent.click(screen.getByLabelText(/dark mode/i));
  await userEvent.click(screen.getByRole("button", { name: /save/i }));
  expect(await screen.findByText(/saved/i)).toBeInTheDocument();
});
// Problem: both tests toggle the same field and assert the same
// confirmation text. Neither tests theme persistence specifically —
// they're the same test written twice under different names.
```

The fix for test 1 is to assert on the observable request, not the internal call:

```tsx
it("saves the updated theme via a PATCH request", async () => {
  const patchSpy = jest.fn();
  server.use(
    rest.patch("/api/users/:id/settings", async (req, res, ctx) => {
      patchSpy(await req.json());
      return res(ctx.status(200), ctx.json({ theme: "dark" }));
    })
  );
  render(<SettingsPage />);
  await userEvent.click(screen.getByLabelText(/dark mode/i));
  await userEvent.click(screen.getByRole("button", { name: /save/i }));
  await screen.findByText(/saved/i);
  expect(patchSpy).toHaveBeenCalledWith({ theme: "dark" });
});
```

This version fails correctly if the payload key gets typo'd, if the value gets inverted, or if the mutation stops firing at all — it checks the thing a user and a backend actually depend on, not which internal hook happened to be invoked. Test 2 gets replaced by assertions on specific fields the form is supposed to render (`getByLabelText(/dark mode/i)`, `getByLabelText(/timezone/i)`, etc.), and one of the two duplicate tests in test 3 gets deleted outright, with the survivor renamed to make clear it's the canonical persistence test.

The pattern across all three problems is the same question, applied test by test: *if I introduced a real bug here, would this test go red?* If you can't answer yes with a specific mechanism, the test is decoration.

---

## Key Takeaways

* Coverage percentage is not test quality — a test can execute every line of buggy code and still pass, because nothing forces it to check the right value.
* Generate tests against a real sibling test file as the pattern reference, and name the categories you want covered explicitly (happy path, empty, validation, API error, boundary) — an open-ended "write tests" prompt anchors on the happy path alone.
* Ask for edge cases as a separate step from generation. It surfaces gaps (stale data, permission changes, duplicate IDs, rapid double-clicks) that never show up when AI is only looking at the happy-path context you handed it.
* Not every discovered edge case deserves a test today — deciding which ones matter is an engineering judgment call, not something to delegate back to AI.
* Review tests with the same skepticism as implementation code: watch for assertions on internal implementation (spy calls, internal state) instead of observable behavior, assertions loose enough to always pass (`toBeDefined`, "renders without crashing"), and near-duplicate tests wearing different names.
* The single question that catches most bad tests: if I broke this logic on purpose, would this specific test go red? If you're not sure, it's decoration, not verification.

## Try It Yourself

1. Pick one existing test file in your codebase — ideally one with more than five tests. For each test, ask yourself (or ask AI, using the 15.3 prompt) whether it would fail if you introduced a realistic bug into the function or component it covers. For any test where the answer is "no" or "not sure," rewrite the assertion to check a specific value, then actually introduce the bug temporarily and confirm the rewritten test goes red before reverting.
2. Take a component in your codebase that currently has only happy-path tests. Run the 15.2 edge-case prompt against its implementation and its existing test file. Pick two of the returned edge cases that represent real risk for your users (not the two easiest to write), and generate tests for them using the 15.1 prompt against your project's existing testing pattern.

---

*[← Previous: Frontend-Specific AI Review](./01-frontend-specific-review.md) | [Next: AI for Documentation & Knowledge →](./03-ai-for-documentation.md)*
