# Module 7 — AI Communication Framework

*[← Previous: Visual-First Design Analysis](./03-visual-first-design-analysis.md) | [Next: Plan Before Code →](../03-workflow/01-plan-before-code.md)*

## Why this matters

Picture two engineers on the same team fixing the same class of bug. The first pastes `"the notifications panel is buggy, please fix it"` into the chat. Ten minutes later they get back a 400-line diff that rewrites the panel's state management, renames three props, and quietly introduces a new dependency — because the AI had to guess what "buggy" meant and filled every gap with an assumption. The second engineer writes four short, structured sentences: what stack it's in, what's actually wrong, what must stay untouched, and what file to imitate. They get back a 12-line fix that touches exactly one function. Same model, same task, wildly different outcomes — the difference is entirely in how the request was structured.

This module gives you that structure. It is not about being polite to AI or finding magic words — it is about applying the same discipline you'd apply to writing a good Jira ticket or a good PR description, so the AI's output is scoped, predictable, and reviewable on the first try instead of the third.

## Objective

Standardize how you and your team communicate with AI, so every non-trivial request carries the same five pieces of information — regardless of who is typing it or which tool they're using.

## 7.1 Context → Goal → Constraints → References → Output

### Why a fixed structure

Left unstructured, prompts drift toward whatever is top of mind — usually the goal, sometimes the constraints, rarely the references. The AI then fills every missing piece with its own default assumption, and defaults are trained on the average of every codebase on the internet, not yours. A fixed five-part structure closes those gaps deliberately, every time, so you stop re-explaining the same project facts in every conversation and stop discovering — after the fact — that the AI used a component or pattern you'd already deprecated. Treat this as your default prompt template, adapting the amount of detail in each part to the size of the task, but never skipping a part outright for anything beyond a one-line question.

**1. Context** — what system is AI working in?

```text
We use React, TypeScript, React Query and Zustand.
```

**2. Goal** — what do we want to achieve?

```text
Implement a user filter modal.
```

**3. Constraints** — what must or must not happen?

```text
- No new dependencies.
- Strict TypeScript.
- Use existing Design System components.
- Do not modify the API layer.
```

**4. References** — which existing code should AI follow?

```text
Use OrderFilterModal.tsx
as the reference implementation.
```

**5. Output** — what should AI return?

```text
First return an implementation plan.
Do not write code yet.
```

Each part answers a question the AI would otherwise have to guess at. Skip "Constraints" and it might add a library. Skip "References" and it might invent a modal pattern that doesn't match the twelve others already in your codebase. Skip "Output" and it might dump a full implementation when you only wanted a plan to review.

### Worked example: a checkout flow

Here is the same five-part structure applied end to end to a different feature, written as a single prompt you could paste as-is:

```text
Context:
We use React, TypeScript, React Hook Form, and React Query.
Payment state lives in a usePaymentSession hook backed by
React Query; UI state (selected payment method, form errors)
is local component state.

Goal:
Add a "save this card for next time" checkbox to the checkout
payment step, and persist that preference through the existing
createOrder mutation.

Constraints:
- No new dependencies.
- Do not change the createOrder API contract; the flag must fit
  into the existing `paymentOptions` object already sent to it.
- Must work with the existing PCI-compliant card input; do not
  touch how card details are collected or transmitted.
- Keep the diff limited to the checkout step component and its
  immediate form schema.

References:
Use ShippingStep.tsx as the reference for how this codebase adds
a new optional field to a multi-step checkout form, including how
it wires validation with React Hook Form.

Output:
First return an implementation plan listing the files you would
touch and the shape of the change to the form schema and the
createOrder payload. Do not write code yet.
```

Notice this is deliberately no longer than a well-written ticket. If you find a part hard to fill in — for instance you don't know the reference file — that's a signal you need to go find one before prompting, not a reason to drop the section.

### When to compress it

For small, low-risk asks (one-line utility function, a straightforward type definition) you can compress this to a sentence or two — the structure is a checklist for your own thinking as much as it is a literal template. The point is that you've considered all five, not that you always type all five out in full.

## 7.2 Persona Is Not the Most Important Part

### Why personas underdeliver

A common instinct when prompting is to open with a persona: "You are a world-class senior frontend architect with 15 years of experience..." This feels like it should raise the quality bar, but it mostly doesn't — a persona is a vague instruction about *tone and confidence*, not a source of information about *your system*. It doesn't tell the AI which state management library you use, what your error-handling convention is, or which of your fifteen existing modal components is the canonical one to copy. Time spent writing an elaborate persona is time not spent supplying the one thing that actually changes the output: real, specific information about the task at hand.

Avoid overusing:

```text
You are a world-class senior frontend architect...
```

A real code example is usually more valuable than a long persona. Prioritize:

* Real context
* Real code
* Real constraints
* Real requirements

### What to do instead

If you want a certain *style* of response — for example, a more skeptical review rather than a generative one — say that directly and specifically, tied to the task, rather than through a persona. Compare:

```text
You are a meticulous 10x senior engineer. Review my settings page.
```

against:

```text
Review the settings page component below for state ownership
problems: is any field being duplicated between local state,
the settingsStore, and the server response from useSettingsQuery?
Point out only that class of issue.
```

The second one produces a focused, checkable answer because it names the actual failure mode you're worried about, in your actual system, instead of hoping a persona will summon the right level of scrutiny.

## 7.3 Atomic Prompts

### Why bundling backfires

A single prompt that asks for UI, API, validation, tests, and a refactor all at once forces the AI to make five sets of interlocking decisions in one pass, with no checkpoint for you to catch a wrong turn before it propagates into the next piece. If the validation logic is wrong, that error is now baked into the UI, the tests, and the refactor built on top of it — and you won't discover it until you review a diff so large that a single wrong assumption is hard to even spot. Atomic prompts trade a small amount of back-and-forth for a large reduction in blast radius: each step produces a diff small enough to actually review, and a mistake in step 2 never contaminates step 4.

Avoid:

```text
Build the UI + API + validation + tests + refactor.
```

Prefer:

```text
Implement the validation schema.

Do not modify the UI.
Do not modify the API layer.
```

Then, once that's reviewed and correct:

```text
Now connect this schema to the existing form.
```

### Worked example: a settings page

Suppose the task is "let users change their notification preferences on the settings page, validated, saved to the backend, and covered by tests." Bundled into one prompt, this reads like Module 8's premature-coding trap. Split atomically, it becomes a short sequence, each one a clean, independently reviewable request:

```text
Prompt 1 — schema only:
Define a Zod schema for NotificationPreferences (email, push, sms
booleans plus a digestFrequency enum). Do not touch any component.

Prompt 2 — wire it to the form, after reviewing the schema:
Connect the NotificationPreferences schema to the existing
SettingsForm using React Hook Form, following the pattern in
ProfileForm.tsx. Do not change the save mutation.

Prompt 3 — connect the save call, after reviewing the form change:
Wire the validated form values into the existing
useUpdateSettingsMutation. Do not add new endpoints or change
the mutation's request shape.

Prompt 4 — tests, after reviewing the wiring:
Generate tests for the notification preferences section following
the existing SettingsForm.test.tsx pattern. Cover valid input,
validation failure, and mutation error.
```

Each prompt is small enough that if something goes wrong, you know exactly which 20-line diff caused it — you're never debugging a 300-line diff to find which of five simultaneous changes broke something.

## 7.4 Output Control

### Why scope the response, not just the request

Even a well-structured Context/Goal/Constraints/References prompt can still get an unwanted result if you don't say what *kind* of output you want back. By default, AI tools lean toward producing code — it's the most common thing asked of them — so if what you actually need is an analysis, a comparison, or a partial change, you have to say so explicitly. This is the cheapest lever in the whole framework: a single added sentence turns an assistant that defaults to "write the whole thing" into one that respects the exact boundary you need, whether that's "don't write code yet" during a planning phase or "touch only this one file" during a targeted fix.

AI can be explicitly instructed to:

```text
Analyze only.
```

```text
Propose two approaches.
```

```text
Do not write code.
```

```text
Modify only this file.
```

```text
Return only the diff.
```

```text
Explain the trade-offs before implementation.
```

This gives you more control over the AI's scope than the goal or constraints alone can — constraints say what the *code* must respect, output control says what *form the response itself* should take.

### Worked example: a notifications panel

Say you're investigating why a notifications panel occasionally shows duplicate entries, but you're not ready to accept a fix yet — you want to understand the problem first.

```text
Avoid:
Fix the duplicate notifications bug in NotificationsPanel.tsx.

Prefer:
Analyze NotificationsPanel.tsx and useNotificationsSubscription.ts.
Identify the likely cause of duplicate entries appearing after a
websocket reconnect.

Propose two possible fixes and their trade-offs.
Do not write code yet.
```

The second version gets you a diagnosis and options you can weigh — including the possibility that the "right" fix touches the subscription hook, not the panel, which you'd never have surfaced if you'd asked for a fix directly. Once you've picked an approach, a follow-up prompt with `Implement approach 2. Modify only useNotificationsSubscription.ts. Return only the diff.` keeps the actual change just as tightly scoped.

## Key Takeaways

* Use Context → Goal → Constraints → References → Output as your default structure for any non-trivial AI request; compress it for trivial ones, but don't skip parts outright.
* A persona is a weak substitute for real information — spend your words on actual context, code, constraints, and requirements instead.
* Keep prompts atomic: one meaningful change per request, so a wrong turn stays small and reviewable instead of contaminating everything built after it.
* Explicitly control the output form ("analyze only," "propose two approaches," "return only the diff") — don't rely on the AI to guess whether you want code or a conversation.
* Constraints control what the code must respect; output control governs what shape the response itself takes. You usually need both.
* A well-structured prompt is judged by the size and clarity of the diff it produces, not by how clever it sounds.

## Try It Yourself

1. Take the next non-trivial prompt you were about to send to AI on your current task. Before sending it, rewrite it using the full Context → Goal → Constraints → References → Output structure, filling in every part explicitly (even if some are short). Send both versions if you can, and compare the diffs you get back.
2. Find a request in your current sprint that bundles more than one concern (for example, "add the field, validate it, and wire it to the API"). Split it into an atomic sequence of prompts like the settings-page example above, reviewing the output after each step before sending the next.

*[← Previous: Visual-First Design Analysis](./03-visual-first-design-analysis.md) | [Next: Plan Before Code →](../03-workflow/01-plan-before-code.md)*
