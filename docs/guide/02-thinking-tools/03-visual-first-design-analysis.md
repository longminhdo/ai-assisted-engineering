# Module 6 — Visual-First Design Analysis

*[← Previous: Context Engineering](./02-context-engineering.md) | [Next: AI Communication Framework →](./04-ai-communication-framework.md)*

## Why this matters

Picture a ticket that reads: "Implement the new pricing page per the attached design — three tiers, the middle one highlighted, monthly/annual toggle." An engineer reads the sentence, skips the attachment because they've built pricing pages before, and asks AI to "build a pricing page with three tiers and a billing toggle." What comes back is a plausible pricing page — three cards, a toggle, a highlighted middle tier — and it is wrong in a dozen small ways nobody asked for explicitly: the highlighted tier doesn't scale up on desktop the way the design does, the toggle is a switch instead of the segmented control the design system actually uses here, the spacing between cards is generic instead of matching the 24px rhythm every other page in the app uses, and the "most popular" badge is positioned differently than every other badge in the product. None of this shows up in code review, because the code compiles, renders, and does roughly the right thing. It shows up when a designer looks at it in staging and asks why it doesn't match the file they spent two days on. This is a frontend-specific failure mode that the earlier modules don't fully cover: **the requirement was never actually a sentence — it was a picture — and describing a picture in words is exactly the "show, don't describe" problem from Module 5.5, except the thing being described is visual, not code.**

## Objective

Make "look at the design before writing anything — a prompt, a contract, or code" the default first step for any frontend task that originates from a visual source (a Figma file, a screenshot, a live reference site, a design system spec), the same way Module 4 makes decomposition the default first step for any task that originates from a requirement sentence.

---

## 6.1 Why Text Descriptions of Visual Design Fail the Same Way Text Descriptions of Code Patterns Do

Module 5.5 made a specific claim about code: describing a convention in prose ("we handle errors with a toast") is lossy compared to pointing at the actual implementation, because prose captures the outcome but not the exact shape. The identical claim is true of visual design, and it's easy to underestimate how true, because visual details feel like they should be "obvious" once you've seen the design once.

They aren't obvious to a model that never saw it. "The cards should have rounded corners and a shadow" doesn't specify a corner radius (4px? 8px? 16px?), a shadow's blur radius, spread, opacity, or color, or whether the shadow appears at rest or only on hover. "Highlight the middle tier" doesn't specify whether that means a border, a background color, a scale transform, an elevated position, a badge, or some combination — a real design usually uses two or three of these together in a specific, deliberate combination that a generic sentence flattens into one vague instruction. Every one of these gaps gets filled the same way Module 0.2 describes: with a plausible, confident, generic default that has nothing to do with the actual design.

The fix is the same shape as Module 5.5's fix, applied to a different medium: **show the AI the actual design — a screenshot, an exported frame, a live Figma reference — instead of describing it.** A model that can see the actual pixels, spacing, and hierarchy is pattern-matching against ground truth instead of reconstructing a guess from an adjective.

---

## 6.2 Visual-First as a Workflow Step, Not a Formality

"Visual-first" means the design is the *first* thing that enters the session for a design-derived task — before you write a prompt describing what you want built, and certainly before any code gets generated. Concretely, the order looks like this:

```text
Design (screenshot / Figma frame / live reference)
 ↓
Visual analysis (what am I actually looking at?)
 ↓
Translate to contract + component structure (Module 4)
 ↓
Gather supporting context (Module 5): design tokens, existing components
 ↓
Implementation
 ↓
Visual verification against the source (Section 6.5)
```

The step that's easy to skip is the second one. Engineers often jump straight from "here's the design" to "build this," treating the visual as decoration for the ticket rather than as the actual specification. Spend a deliberate pass first — either yourself, or by explicitly asking AI to describe what it sees before generating anything — identifying:

* **Layout structure**: how is this composed? Grid, flex, stacked sections? What's the hierarchy?
* **Spacing and rhythm**: is there a consistent unit (8px, 4px) visible across gaps and padding?
* **Typography**: how many distinct text styles are visible, and do they look like they map to existing design tokens or ad hoc sizes?
* **States implied by the design**: does the file show more than one state (empty, error, loading, hover, focus, disabled)? A single static frame almost never shows all of these, which is itself important information (Section 6.4).
* **Existing patterns**: does this look like a variant of a component that already exists elsewhere in the product, or something genuinely new?

**Worked example.** Handed a Figma frame for the pricing page from the intro, a visual-first pass — before any prompt is written — might read: "Three cards in a row, equal width, 24px gap. Middle card is visually elevated: it has a colored border, sits 8px higher via a negative margin, and carries a 'Most Popular' pill badge in the top-right corner overlapping the card edge. Typography looks like three tiers: a large price (looks like an existing `heading-lg` token based on size), a feature list in body text with a checkmark icon per line, and a CTA button using what looks like the app's primary button style. Toggle above the cards is a segmented control, not a switch — two-state, both options always visible, selected one has a filled background." That paragraph is a translation of pixels into words a design-familiar engineer would recognize — and crucially, it's specific enough to catch in review if it's wrong, the same way a written contract (Module 4.1) is specific enough to catch if it's wrong, and a vague "three tiers, middle highlighted" is not.

---

## 6.3 Getting the Design Into the Session

A visual-first pass requires the design to actually be visible to the AI, not just referenced. Depending on your tooling, this typically means one of:

* **A screenshot or exported image** of the relevant frame, pasted directly into the session — the most universally available option and often sufficient on its own.
* **Direct design-tool access** (e.g., a Figma integration that can read frame structure, exported assets, and design tokens directly), when available — this can surface information a flat screenshot can't, like exact spacing values, color tokens, or component variants defined in the design system itself, rather than requiring those to be inferred visually.
* **A live reference**, when the target is "match this existing page/product" rather than a new design file — a screenshot of the running reference, or direct browser inspection if your tooling supports it.

Whichever source you use, the same Module 5 principle applies: bring the *specific* frame or state relevant to the current task, not an export of the entire design file. A 40-screen Figma file pasted in wholesale is context pollution (Module 5.3) in visual form — the one pricing-page frame you need competes for attention with 39 irrelevant ones.

**Worked example.** Rather than exporting and pasting an entire "Marketing Site" Figma file when the task is "implement the pricing page," export just the pricing page frame, at both desktop and mobile breakpoints if the design specifies distinct layouts for each — two focused images instead of one file with forty frames. If the design tool integration can also surface the actual spacing tokens and color variables used in that frame, include those directly rather than asking the model to estimate a hex value or a pixel gap by eye from a flat image.

---

## 6.4 From Visual to Contract: What a Screenshot Doesn't Tell You

A static design file is a snapshot of one state, usually the happy path, at one point in time. This is exactly analogous to Module 4's requirement-to-contract step, and it deserves the same discipline: **the visual tells you the target shape; it does not tell you the full behavior**, and treating a screenshot as a complete spec produces the same category of gap as treating a one-sentence requirement as a complete spec.

Before implementation, explicitly work out what the design *doesn't* show, the same way Module 4.1 works out a contract from an ambiguous requirement sentence:

* What does this look like **empty** (no pricing tiers configured), in an **error** state (pricing data failed to load), and **loading**? A single Figma frame almost never shows all three.
* What are the **interactive states** — hover, focus, active, disabled — for every interactive element in the design? A static export shows one state per element; a real component needs all of them, and they should be visually and behaviorally consistent with how the rest of the product already handles these states (Module 5's "existing patterns" layer).
* What is the **responsive behavior**? If you only have a desktop frame, does the design system have an established pattern for how a three-column layout like this collapses on mobile (stack? horizontal scroll? accordion?), or does that need an explicit decision the same way Module 4.3's state-ownership table needs an explicit decision?
* Does this map to an **existing component** with a new variant, or is it genuinely a new component? Getting this wrong in either direction either fights the design system (building a bespoke card when a `<PricingCard>` already exists) or forces an existing component into a shape it wasn't meant for.

Once these questions are answered, the visual analysis converts into the same kind of contract Module 4.1 produces from a text requirement — except now it's grounded in what was actually seen, not what was generically assumed.

**Worked example, continued.** For the pricing page, the visual-only spec ("three cards, middle highlighted, toggle") becomes a real implementation plan once you've asked the missing-state questions: loading state reuses the existing `<CardSkeleton>` pattern already used elsewhere in the app (a convention to reuse, not reinvent); the mobile breakpoint stacks the three cards vertically with the "most popular" one reordered to appear first, matching how the existing `<FeatureComparisonTable>` component handles the same three-column-to-stack problem elsewhere in the product; the CTA button's disabled state (for an already-subscribed tier) needs a treatment the design didn't show at all, so it's flagged as an explicit open question for the designer rather than silently invented.

---

## 6.5 Closing the Loop: Verify Against the Visual, Not Just Against the Code

Module 9 makes the point that "it compiles" is not "it's correct." For design-derived work, there's a frontend-specific corollary: **"it renders" is not "it matches."** A component can be fully functional, pass every test, and still be visually wrong in ways that only show up by looking at it next to the source design — a spacing value that's close but not exact, a font weight that's one step off, a shadow that reads as flatter or heavier than the reference.

Close the loop the same way you opened it: visually. Take a screenshot of the implemented result, at the same states and breakpoints identified in Section 6.4, and compare it directly against the original design — either yourself, or by handing both images to AI and asking it to identify concrete visual discrepancies (spacing, alignment, color, typography), the same way Module 8's code review asks AI to actively look for problems rather than assume correctness. This is a real, checkable verification step, not a formality — it belongs in the same category as the runtime verification checklist in Module 9.6, and it's the step most likely to be skipped under deadline pressure, which is exactly when a design mismatch is most likely to slip into production.

**Worked example.** After implementing the pricing page, screenshot the built result at desktop and mobile and place it next to the original Figma export. A side-by-side pass catches that the badge's overlap amount is 4px off, that the segmented toggle's selected-state transition is missing (present in the design's interaction notes but easy to miss in a static export), and that the card shadow is using the design system's default elevation token instead of the slightly heavier one this specific design calls for. None of these would show up in a type check, a lint pass, or even a casual glance at the rendered page — they show up specifically when the implementation is checked *against the source it was supposed to match*.

---

## Key Takeaways

* A design file or screenshot is a specification, not decoration for the ticket — treat "look at the design" as a mandatory first step for any design-derived task, the same way Module 4 treats decomposition as mandatory for any requirement-derived task.
* Text descriptions of visual design are as lossy as text descriptions of code conventions (Module 5.5) — show the actual design (screenshot, exported frame, design-tool integration) instead of describing it in adjectives.
* Bring the specific frame the task needs, not the whole design file — an entire multi-screen export is context pollution in visual form.
* A static design shows one state at one moment. Explicitly work out the states it doesn't show (empty, loading, error, hover/focus/disabled, responsive breakpoints) before implementation, the same way Module 4.1 works out a contract from an ambiguous sentence.
* "It renders" is not "it matches." Close the loop by screenshotting the implementation and comparing it directly against the source design — this is a real verification step, not a nice-to-have.

## Try It Yourself

1. Take a design-derived task you're about to start (or one you finished recently). Before touching code — or looking back at what you built — write out a visual analysis paragraph the way Section 6.2's worked example did: layout, spacing, typography, and what looks like an existing pattern versus something new. Compare that against the actual implementation. Any mismatch is a gap that a text-only requirement would have hidden entirely.
2. For a component you've already shipped from a design, do the Section 6.5 side-by-side comparison now: screenshot the live result next to the original design file at the same breakpoint. Note every discrepancy, however small. If you find more than one, that's a concrete argument for making this comparison a standard last step, not an occasional gut check.

---

*[← Previous: Context Engineering](./02-context-engineering.md) | [Next: AI Communication Framework →](./04-ai-communication-framework.md)*
