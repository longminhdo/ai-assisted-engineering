# AI-Assisted Frontend Engineering — Slide Deck Content Outline

*Speaker-led deck: minimal on-slide text, presenter narrates. Each block below = one slide's worth of content.*

---

## OPENING: Training Objective

**slide_title:** Better Decisions, Not More Code

**hook:** Set the frame before anything else — this training is not "how do I get AI to write more code," it's "how do I make better engineering decisions faster."

**key_points:**
- Not the goal: "make AI write more code"
- The goal: better decisions, faster delivery, no quality loss
- Core loop: Understand -> Decompose -> Context -> Plan -> Implement -> Verify -> Refine
- Loop repeats — Refine feeds back to Understand
- You are the pilot in this loop, always

**best_visual:** Circular/looping flow diagram of the 7-step wheel (Understand -> Decompose -> Context -> Plan -> Implement -> Verify -> Refine -> back to Understand), drawn as a closed loop, not a line — emphasize it never "ends."

**core_quote:** "How can engineers use AI to make better decisions and deliver software faster without compromising quality?"

---

## OPENING/AGENDA: Expected Capabilities (7 categories)

**slide_title:** What You'll Walk Away With

**key_points:**
- A. AI Mindset
- B. Session & Resourcing Mechanics
- C. Problem Decomposition
- D. Context Engineering
- E. AI Communication
- F. AI Development Workflow
- G. Verification

**best_visual:** 7-tile agenda grid (like a menu of chapters), letter A-G each as a card with a one-word icon.

---

# SECTION: Foundations

## 01-ai-mindset

**slide_title:** Pilot vs. Copilot

**key_points:**
- Shift: "AI writes code for me" -> "I direct AI"
- AI strong: well-scoped, locally-defined tasks
- AI weak: unclear requirements, implicit conventions
- Missing context -> AI guesses confidently, silently
- Engineer owns architecture, security, correctness, final review

**best_visual:** Two-column contrast: "Engineer (Pilot)" — architecture, business logic, security, final review — vs "AI (Copilot)" — exploration, proposals, generation, review.

**core_quote:** "AI can propose. Engineers decide."

**code_or_example:**
```
Build the user management page.
```
(AI silently guesses state, API, pagination, permissions, error handling)

---

## 02-how-ai-works

**slide_title:** Everything Lives in the Window

**key_points:**
- Context window = only thing AI can see, right now
- Measured in tokens, not files/lines
- Recency bias: recent content dominates
- Mid-context content quietly loses influence
- Context rot = compounding of stale + noisy + contradicted context

**best_visual:** A filling glass/bucket diagram — "context window" fills with tokens over a session timeline; early instruction fades in opacity as new content stacks on top.

**core_quote:** "If it is not in the window, it does not exist for this response."

---

## 03-controlling-your-session

**slide_title:** Start Fresh, or Keep Going?

**key_points:**
- New session: task category changes
- New session: abandoned approach still lingering
- Same session: iterating on one piece of work
- Compact = same task, more room (lossy)
- Clear = different task, blank slate

**best_visual:** Decision flowchart / traffic-light: three yes/no gates routing to "Keep Going" vs "Start Fresh."

**core_quote:** "Session boundaries should track the task, not the clock."

**code_or_example:**
```
/clear   -> blank slate, different task
/compact -> same task, summarized, more room
```

---

## 04-choosing-model-and-effort

**slide_title:** Resource by Risk, Not Size

**key_points:**
- Large + mechanical -> low effort (rename across files)
- Small + judgment-heavy -> high effort (state reconciliation call)
- Higher effort = more exploration before committing
- Danger: under-provisioning looks just as confident as correct
- When unsure, round up

**best_visual:** 2x2 matrix: axes "Size" (small/large) vs "Judgment Required" (low/high).

**core_quote:** "A fast, shallow answer to a hard question looks exactly as confident as a well-reasoned one."

---

# SECTION: Thinking Tools

## 01-problem-decomposition

**slide_title:** Requirement -> Contract, Not Code

**key_points:**
- Requirement -> Contract -> Data -> State -> Architecture -> Implementation -> Verification
- Contract = a type, not a sentence
- Classify state: local / server / global / URL
- Break into small, ordered, reviewable tasks
- Independent tasks -> parallel sub-agents; dependent -> sequential

**best_visual:** Horizontal pipeline diagram, each stage a labeled box with an arrow; small dependency-graph inset for parallel work.

**core_quote:** "One meaningful change at a time."

**code_or_example:**
```ts
type UserFilter = {
  keyword: string;
  roles: Role[];
  status?: UserStatus;
};
```

---

## 02-context-engineering

**slide_title:** Good Context Beats a Clever Prompt

**key_points:**
- Climb the pyramid: Task -> Contracts -> Patterns -> Related Modules -> Full Repo
- More context != safer — causes wrong pattern selection
- Minimum sufficient context: everything needed, nothing extra
- Show the pattern, don't describe it in prose

**best_visual:** Pyramid diagram (Context Pyramid) — Task at tip, Full Repository at base.

**core_quote:** "Show AI the pattern instead of describing the pattern."

**code_or_example:**
```
Avoid: "Our project uses a custom modal."
Prefer: point at OrderFilterModal.tsx directly.
```

---

## 03-visual-first-design-analysis

**slide_title:** The Design Is the Spec

**key_points:**
- Text description of visual design is as lossy as prose code conventions
- Show the actual screenshot/frame, not adjectives
- A static frame hides empty/loading/error/hover states
- Bring the specific frame, not the whole Figma file
- "It renders" != "it matches" — verify against the source visually

**best_visual:** Before/after contrast: vague text ("rounded corners and a shadow") vs. an annotated screenshot with spacing/radius/shadow values.

**core_quote:** "The design is the specification. Read it before you write anything else."

---

## 04-ai-communication-framework

**slide_title:** Context -> Goal -> Constraints -> References -> Output

**key_points:**
- 5 parts: Context, Goal, Constraints, References, Output
- Persona is a weak substitute for real information
- Keep prompts atomic — one change per request
- Explicitly control output form ("analyze only," "don't write code")

**best_visual:** 5-step horizontal template/checklist card, each labeled slot filled with a one-line example.

**code_or_example:**
```
Context: React, TS, React Query, Zustand.
Goal: Implement a user filter modal.
Constraints: No new deps. Use existing DS.
References: OrderFilterModal.tsx
Output: Plan only. Do not write code yet.
```

---

# SECTION: Workflow

## 01-plan-before-code

**slide_title:** Analyze -> Plan -> Review -> Implement

**key_points:**
- Analyze first — "do not write code" as an explicit instruction
- Plan is a cheap, correctable artifact
- Human reviews the plan against team conventions AI can't see
- Implement one step at a time, review each

**best_visual:** 4-stage relay diagram: Analyze -> Plan -> Human Review -> Implement, with a "cost to fix" callout curve rising sharply after Review is skipped.

**core_quote:** "Analyze -> Plan -> Review -> Implement"

**code_or_example:**
```
Analyze this requirement and the existing code.
Identify: components, patterns, state ownership, edge cases.
Do not write code.
```

---

## 02-ai-coding-partner

**slide_title:** Generate, Transform, Explain, Discover

**key_points:**
- Generate: best for small, self-contained units
- Transform: risk is silent alteration — state what must stay identical
- Explain: name the dimension (state flow, timing) not "what does this do"
- Discover: ask AI to find existing patterns before inventing new ones

**best_visual:** Four-quadrant grid, one job per quadrant with icon + one-line risk/tip.

**code_or_example:**
```
Find the pattern used by this repository
for handling mutation errors.
Do not create a new abstraction.
```

---

## 03-ai-debugging

**slide_title:** Hypotheses Before Fixes

**key_points:**
- Give full context: expected, actual, error, recent changes
- Ask for top 3 causes + evidence + how to verify — before any fix
- "Do not modify code yet" prevents anchoring on unconfirmed theory
- Loop: Problem -> Hypothesis -> Evidence -> Experiment -> Fix -> Verify
- Write a root-cause note; brownfield -> fix doc first, greenfield -> fix then document

**best_visual:** Two contrasted loops side by side: converging "Hypothesis-Driven Loop" vs. flat "Random-Change Loop."

**code_or_example:**
```
List the top 3 possible causes.
For each: explain why, identify evidence, suggest how to verify it.
```

---

## 04-ai-refactoring

**slide_title:** Refactor Is Not "Clean This Up"

**key_points:**
- Analyze first: name duplication, unnecessary state, coupling — don't refactor yet
- Propose 2 approaches with trade-offs; human picks
- Constrain implementation: preserve behavior, no API changes, no new deps, minimal diff
- "Preserve behavior" can mean preserving existing bugs on purpose

**best_visual:** 3-step gate sequence (Analyze -> Compare -> Refactor Safely), padlock icon per gate.

**code_or_example:**
```
Implement approach 1.
Constraints: preserve behavior, no public API changes,
no new dependencies, keep the diff minimal.
```

---

## 05-ai-code-review

**slide_title:** Ask It to Prove You Wrong

**key_points:**
- "Review this" -> generic agreement (useless)
- 10-point checklist: race conditions, stale closures, a11y, edge cases...
- "Try to prove this is wrong" — adversarial framing finds real bugs
- Review one dimension at a time (performance only, a11y only)

**best_visual:** Before/after contrast: AI response to "is this code good?" vs. "try to prove this is wrong" — side-by-side speech bubbles.

**core_quote:** "Do not assume the implementation is correct. Try to find reasons it could fail."

**code_or_example:**
```
Try to prove this implementation is wrong.
Find inputs, states, or interactions that
could cause incorrect behavior.
```

---

## 06-verification

**slide_title:** Generated Code Is Not Verified Code

**key_points:**
- Pyramid (bottom->top): Type Check -> Lint -> Unit Test -> E2E -> Runtime -> Human Review
- Work bottom-up: cheap checks first
- A weak assertion (`toBeDefined()`) is coverage theater
- Runtime states (loading, empty, slow network, dup clicks) are where AI defaults fail

**best_visual:** Verification pyramid, literal 6-tier pyramid, annotate each tier with "catches: ___."

**core_quote:** "Generated code is not verified code."

**code_or_example:**
```
// weak: expect(result).toBeDefined();
// real: expect(result.total).toBe(180);
```

---

# SECTION: Frontend & Quality Depth

## 01-frontend-specific-review

**slide_title:** Six Frontend-Only Blind Spots

**key_points:**
- React/Vue: dependency arrays lie about their contract
- State management: server state duplicated into global store
- Data fetching: race conditions from rapid open/close
- Performance: unmemoized filters at scale
- Accessibility: modal with no focus trap, no ARIA
- Responsive: fixed-width table breaks on phone

**best_visual:** Six-lens grid (hooks, state, fetch, perf, a11y, responsive), each tile with a one-line bug example.

**code_or_example:**
```tsx
useEffect(() => {
  const id = setInterval(fetchData, 30000);
  return () => clearInterval(id);
}, []); // stale closure — dateRange never updates
```

---

## 02-ai-for-testing

**slide_title:** Coverage Is Not Correctness

**key_points:**
- Generate against a real sibling test file, not from scratch
- Name categories explicitly: happy path, empty, error, boundary
- Ask for edge cases as a separate step (stale data, permission changes)
- Review test quality: implementation details? weak assertions? duplicates?
- The test: "would this go red if I broke the logic?"

**best_visual:** Before/after test snippet: `toBeDefined()` (useless) vs. `expect(result.total).toBe(180)` (meaningful).

**core_quote:** "If I introduced a real bug here, would this test go red?"

**code_or_example:**
```ts
it("calculates order total", () => {
  expect(result).toBeDefined(); // passes even if logic is deleted
});
```

---

## 03-ai-for-documentation

**slide_title:** AI as a Knowledge Tool, Not Just a Coder

**key_points:**
- Explain legacy code: "what would break if I deleted this?"
- Generate READMEs/API docs from actual exports — "don't invent props"
- Summarize architecture: ask for "known risks," not just a description
- Migration/onboarding guides: ask what AI *couldn't* determine

**best_visual:** Risk/reward 2x2: documentation tasks vs. code-generation tasks — docs land high-reward/low-risk quadrant.

**code_or_example:**
```
Explain this component.
What would break if I deleted the useEffect on line 340?
Flag anything that looks like a workaround for a bug elsewhere.
```

---

## 04-ai-anti-patterns

**slide_title:** The Seven Ways This Goes Wrong

**key_points:**
- Vague Prompting — "make this better"
- Premature Coding — build before architecture decided
- Context Dumping — whole repo "just in case"
- Blind Acceptance — it compiles, done
- AI-Driven Architecture — "AI suggested Zustand, so..."
- Endless Conversation — one thread, five unrelated tasks
- Over-Automation — architecture/security decided by default

**best_visual:** 7-card "wall of shame" grid, each card a red-flagged anti-pattern icon with its one-line violation quote.

**code_or_example:**
```
Make this better.
```
(No context, criteria, or expected output)

---

# SECTION: Team Standard

## 01-standard-ai-workflow

**slide_title:** The One Loop the Whole Team Runs

**key_points:**
- Requirement -> Understand -> Decompose -> Context -> Plan
- Human Review Plan (gate 1 — cheapest place to catch a wrong direction)
- Implement -> Verify (Type / Test / Runtime, parallel)
- AI Review -> Human Review (gate 2 — final accountability)
- Merge = "a human is willing to put their name on this"

**best_visual:** Vertical pipeline/stepper diagram, 10 stacked stages, two "HUMAN REVIEW" gates visually distinguished, VERIFY branching into 3 parallel lanes that rejoin before AI Review.

---

## 02-team-coding-standard

**slide_title:** Ten Rules, One Team

**key_points (all 10, condensed to short phrases):**
1. AI doesn't decide architecture alone
2. Non-trivial tasks get planned first
3. Use existing patterns first
4. Minimum sufficient context
5. Keep changes small and reviewable
6. Never trust unverified code
7. No new dependencies without reason
8. Requirements outrank AI output
9. Existing code beats text description
10. The engineer owns the result

**best_visual:** Kinetic marquee / rule ticker — the ten rules scroll past continuously, mono-numbered, one accent-colored word per rule. (Deck's single marquee moment — used once.)

---

# SECTION: Playbooks

**slide_title:** A Recipe for Every Situation

**key_points:**
- Situation -> Steps -> Watch-outs -> Related reading, same shape every time
- Covers 9 recurring scenarios: New Feature, Migrating Code, Debugging, Updating Existing Code, Docs->Code, Using a Skill, Dividing Work w/ Sub-Agents, Managing a Session, Prompt Quick Reference

**best_visual:** 3x3 grid of the 9 playbook titles as situation cards.

---

# SECTION: Advanced Track

**slide_title:** Beyond Your Own Session

**key_points:**
- Anatomy of a project: CLAUDE.md (always-on) vs. Skills (on-demand) vs. Subagents vs. Hooks
- Skills vs. Rules: rule = costs context every turn but guaranteed seen; skill = free until invoked but must be discoverable
- Contributing loop: notice a repeated pattern -> draft as skill -> test on real task -> team review

**best_visual:** Layered "stack" diagram: bottom layer = your own session habits, top layer = shared team tooling (CLAUDE.md, skills, subagents, hooks).

---

## CLOSING: 5 Core Principles

**slide_title:** The Five Things to Remember

**key_points (verbatim):**
1. AI doesn't know your codebase unless you show it.
2. Don't ask AI to solve a problem you haven't decomposed.
3. Plan before coding when the task is non-trivial.
4. Give AI patterns, not just instructions.
5. Never trust generated code without verification.

**best_visual:** Numbered stacked reveal, large display type, one per beat.

---

## CLOSING: Final Message

**slide_title:** The Actual Goal

**core_quote (verbatim):**
> "The goal of AI-assisted development is not to make AI write more code.
> The goal is to help engineers deliver correct, maintainable software faster."

**best_visual:** Full-bleed centered quote, largest type in the deck, no other elements.
