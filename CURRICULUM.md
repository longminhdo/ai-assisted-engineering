# AI-ASSISTED FRONTEND ENGINEERING TRAINING CURRICULUM

## 1. Training Objective

### 1.1. Overall Objective

By the end of this training, FE team members should be able to use AI as an **engineering partner** throughout the software development lifecycle:

> **Understand → Decompose → Context → Plan → Implement → Verify → Refine**

The goal is **not**:

> “How can we make AI write more code?”

The goal is:

> **“How can engineers use AI to make better decisions and deliver software faster without compromising quality?”**

---

# 2. Expected Capabilities

After completing the training, team members should be able to:

### A. AI Mindset

* Understand where AI is strong and where it is weak.
* Know what AI can reasonably decide on its own.
* Know what decisions must remain with the engineer.
* Avoid blindly trusting AI output.
* Identify hallucinations, assumptions, and overconfidence.

### B. Session & Resourcing Mechanics

* Understand the context window and why long, unfocused sessions degrade (context rot).
* Know when to start a fresh session, compact, or clear versus continuing.
* Split work across sessions by task boundary, not by clock time.
* Restate load-bearing constraints instead of assuming they still carry weight.
* Match model tier and reasoning effort to a task's ambiguity and risk, not its size.

### C. Problem Decomposition

* Break requirements into smaller tasks.
* Define data contracts before implementation.
* Determine state ownership.
* Define data flow.
* Define component boundaries.
* Identify edge cases.
* Identify which decomposed tasks are independent and can run in parallel sessions or sub-agents (Divide & Conquer), versus which must stay sequential.

### D. Context Engineering

* Determine what context AI actually needs.
* Avoid context pollution.
* Find relevant patterns in the codebase.
* Use existing implementations as references.
* Provide the right types, API contracts, dependencies, and conventions.
* For design-derived tasks: analyze the visual source (screenshot, Figma frame) first, before writing a prompt or code, and verify the result against it afterward.

### E. AI Communication

* Write clear tasks.
* Define technical constraints.
* Specify expected output.
* Ask AI to analyze before coding.
* Know when to split a task into multiple interactions.

### F. AI Development Workflow

Use AI for:

* Analysis
* Planning
* Implementation
* Debugging (hypothesis-driven, with a documented root cause — not silent patching)
* Refactoring
* Testing
* Review

### G. Verification

* Review generated code.
* Check TypeScript.
* Check lint.
* Check tests.
* Check runtime behavior.
* Check accessibility.
* Check performance.
* Check security.
* Check business logic.

---

# MODULE 0 — AI MINDSET

## Objective

Shift the mindset from:

> “AI writes code for me.”

to:

> **“AI is an engineering tool that I am responsible for directing.”**

---

## 0.1. What Is AI Good At?

AI is particularly strong at:

* Code generation
* Code transformation
* Code explanation
* Pattern discovery
* Refactoring
* Test generation
* Documentation generation
* Debugging assistance
* Comparing approaches
* Boilerplate generation

Examples:

```text
Convert this Vue Options API component
to Composition API.
```

```text
Generate unit tests for this function
based on the existing testing pattern.
```

These tasks work well because their scope and inputs are relatively clear.

---

## 0.2. Where Is AI Weak?

AI can struggle when:

* Business requirements are unclear.
* Context is incomplete.
* Project conventions are unknown.
* Architecture is implicit.
* Multiple modules are tightly coupled.
* Trade-offs have not been decided.
* Real production behavior needs to be understood.

For example:

```text
Build the user management page.
```

AI does not automatically know:

* Where should state live?
* Which API should be used?
* Which Design System should be used?
* What is the pagination pattern?
* How should permissions work?
* How should errors be handled?
* Should filters sync with the URL?
* Should this be server state or client state?
* Is SSR involved?

AI will have to **make assumptions**.

---

## 0.3. Pilot vs. Copilot

### Engineer — Pilot

The engineer owns:

* Architecture
* Business logic
* Technical decisions
* Security
* Performance
* Correctness
* Final code review

### AI — Copilot

AI assists with:

* Exploration
* Proposals
* Generation
* Transformation
* Review
* Debugging
* Testing

Core principle:

> **AI can propose. Engineers decide.**

---

# MODULE 1 — HOW AI PROCESSES YOUR REQUEST

## Objective

Understand the mechanics behind every AI session, so that context, session control, and model choice (Modules 2–3) become deliberate decisions instead of guesswork.

---

# 1.1. The Context Window

AI has no memory between sessions. Everything it can use to answer is limited to what is currently inside its context window: instructions, pasted files, prior messages, tool results.

```text
If it is not in the window,
it does not exist for this response.
```

---

# 1.2. Tokens

Context is measured in tokens, not files or lines.

```text
Large file
+
Repeated pastes
+
Generated/boilerplate code
=
Wasted budget, not more signal
```

---

# 1.3. Position Matters

Not all tokens are weighted equally.

* Recent content has more influence than old content.
* Content in the middle of a long session is the most likely to be underweighted.

```text
Constraint stated once, early
+
Many unrelated turns since
=
Constraint quietly loses influence
```

---

# 1.4. Context Rot

Long, unfocused sessions degrade in quality even though the model has not changed.

Causes:

* Accumulated irrelevant files
* Faded early constraints
* Contradictions between early and later instructions

```text
One focused task, long session
→ stays sharp

Many unrelated tasks, one session
→ degrades
```

Core principle:

> **Everything else in this guide is a strategy for controlling what is inside the context window.**

---

# MODULE 2 — CONTROLLING YOUR SESSION

## Objective

Turn session boundaries into a deliberate decision, not an accident of when you happened to open the tool.

---

# 2.1. When to Start Fresh

Start a new session when:

```text
Task category changes
Different files are needed
A prior approach was abandoned
You can no longer recall everything you've told it
```

Continue the same session when:

```text
You are iterating on the same piece of work
You are mid-way through a multi-step implementation
```

---

# 2.2. Compact vs. Clear

```text
Compact → same task, more room, summary is lossy
Clear   → different task, start from zero
```

---

# 2.3. Split Work by Task, Not by Time

```text
Sequential, dependent steps  → one session
Independent pieces           → separate sessions or sub-agents (see 4.6)
```

---

# 2.4. Restate Load-Bearing Constraints

Do not assume an instruction from early in a long session is still fully weighted.

```text
Before a risky step:
Restate the one or two constraints
that would be expensive to lose.
```

Core principle:

> **Session boundaries should track the task, not the clock.**

---

# MODULE 3 — CHOOSING YOUR MODEL & EFFORT

## Objective

Match model tier and reasoning effort to a task's ambiguity and risk — not its size.

---

# 3.1. Size Is Not the Right Signal

```text
Large but mechanical
(rename a prop across 30 files)
→ low effort

Small but judgment-heavy
(how should optimistic UI reconcile with a websocket push?)
→ high effort
```

---

# 3.2. What Higher Effort Buys

Higher effort means more exploration before committing to an answer.

```text
First plausible answer is often wrong  → higher effort pays off
Correct answer is already unambiguous  → higher effort is wasted
```

---

# 3.3. Decision Questions

```text
How reversible is a wrong answer?
How much judgment vs. mechanical pattern-matching does this need?
What's at stake if it's subtly wrong?
```

---

# 3.4. When Unsure, Round Up

Over-provisioning costs time and money. Under-provisioning on a judgment-heavy task produces a confidently wrong answer that looks just as finished as a correct one.

Core principle:

> **A fast, shallow answer to a hard question looks exactly as confident as a well-reasoned one. That is what makes under-provisioning dangerous.**

---

# MODULE 4 — PROBLEM DECOMPOSITION

## Objective

Do not immediately ask AI to code when receiving a requirement.

The engineer should first transform:

```text
Requirement
```

into:

```text
Contract
→ Data
→ State
→ Architecture
→ Implementation
→ Verification
```

---

# 4.1. Requirement → Contract

Example requirement:

> Users can filter the user list by keyword, role, and status.

Before coding, define:

```ts
type UserFilter = {
  keyword: string;
  roles: Role[];
  status?: UserStatus;
};
```

API:

```ts
type UserListResponse = {
  items: User[];
  total: number;
};
```

This is the **contract**.

AI should not invent a contract when an existing contract already exists in the system.

---

# 4.2. Identify the Data Flow

For example:

```text
User
 ↓
Filter Form
 ↓
Filter State
 ↓
Query Parameters
 ↓
API
 ↓
React Query
 ↓
User List
```

Identify:

* Where does the data come from?
* Who owns the state?
* Who transforms the data?
* Where does server state live?
* Where does client state live?

---

# 4.3. State Ownership

This is one of the areas where AI can easily make poor decisions.

### Local UI State

```text
isModalOpen
selectedTab
inputValue
```

### Server State

```text
users
orders
products
```

### Global Client State

```text
authenticated user
theme
sidebar state
```

### URL State

```text
?page=2
&status=active
&keyword=minh
```

The engineer should decide where state belongs **before implementation**.

---

# 4.4. Component Decomposition

Example:

```text
UserPage
├── UserToolbar
│   └── UserFilterButton
│
├── UserFilterModal
│   ├── KeywordInput
│   ├── RoleSelect
│   └── StatusSelect
│
└── UserTable
```

AI can suggest the structure.

The engineer makes the final decision.

---

# 4.5. Task Decomposition

Avoid:

```text
Build the entire user management feature.
```

Prefer:

```text
Task 1: Define filter contract
Task 2: Implement filter form
Task 3: Connect filter to query
Task 4: Implement table
Task 5: Add pagination
Task 6: Add tests
```

Principle:

> **One meaningful change at a time.**

---

# 4.6. Divide & Conquer: Parallelizing with Sub-Agents

Once the task list exists, draw the dependency graph between tasks. Tasks with no edge between them don't need to run in sequence.

```text
Task 1 (contracts)
   │
   ├──▶ Task 2 (independent)  ──┐
   │                             ├──▶ Task 4 (integrate) ──▶ Task 5 ──▶ Task 6
   └──▶ Task 3 (independent)  ──┘
```

Independent tasks → separate sessions or parallel sub-agents, each with its own minimum sufficient context (Module 5). Dependent tasks → stay sequential in one session (Module 2.3).

Parallel work needs an explicit integration step, reviewed as carefully as any other task — the most common failure is two independently-built pieces silently disagreeing about a contract that was ambiguous in one direction.

Core principle:

> **Decompose to find the dependency graph. Parallelize only where there is no edge.**

---

# MODULE 5 — CONTEXT ENGINEERING

## Objective

Understand that:

> **Good context is more important than a clever prompt.**

---

# 5.1. What Does AI Need to Know?

For a typical frontend task, useful context may include:

```text
Requirement
+
Types
+
API Contract
+
Existing Components
+
Existing Hooks
+
Design System
+
Relevant Business Rules
+
Constraints
```

You do not necessarily need to provide the entire repository.

---

# 5.2. Context Pyramid

Prioritize context by relevance:

```text
                    Full Repository
                         ▲
                         │
                  Related Modules
                         ▲
                         │
                 Existing Patterns
                         ▲
                         │
                    Contracts
                         ▲
                         │
                       Task
```

Start with the smallest amount of context that is sufficient to solve the task.

---

# 5.3. Context Pollution

Bad example:

```text
20 files
+
entire API folder
+
all components
+
all CSS
+
all hooks
```

The AI now has to process a large amount of irrelevant information.

Possible consequences:

* Loss of focus
* Wrong pattern selection
* More assumptions
* Unnecessary code
* Increased hallucination risk

---

# 5.4. Minimum Sufficient Context

Principle:

> **Give AI everything it needs, and nothing it doesn't.**

For a Filter Modal, the relevant context might be:

```text
UserFilter type
Filter API
Existing FilterModal
Existing Form
Existing Select
Validation schema
One similar feature
```

Not the entire repository.

---

# 5.5. Repository Pattern Learning

Instead of explaining the project through text:

```text
Our project uses a custom query hook.
Our project uses a custom modal.
Our project handles errors with toast.
```

Provide actual implementations:

```text
OrderFilterModal.tsx
useOrderQuery.ts
OrderFilter.test.tsx
```

Then tell AI:

```text
Use these files as reference implementations.

Follow their:
- component structure
- naming conventions
- state management
- error handling
- testing patterns
```

Core principle:

> **Show AI the pattern instead of describing the pattern.**

---

# MODULE 6 — VISUAL-FIRST DESIGN ANALYSIS

## Objective

For any task that starts from a design — a Figma file, a screenshot, a live reference — look at it before writing a prompt or a line of code.

---

# 6.1. Text Descriptions of Design Are Lossy

```text
"Rounded corners and a shadow"
```

does not specify a radius, a blur amount, an opacity, or when the shadow appears.

Core principle:

> **Show the design. Do not describe it in adjectives.**

---

# 6.2. Visual-First Workflow

```text
Design (screenshot / Figma frame)
 ↓
Visual analysis: layout, spacing, typography, states, existing patterns
 ↓
Contract + component structure (Module 4)
 ↓
Supporting context (Module 5)
 ↓
Implementation
 ↓
Visual verification against the source
```

Bring the specific frame the task needs, not the whole design file — an entire multi-screen export is context pollution (5.3) in visual form.

---

# 6.3. What a Screenshot Doesn't Show

A static design shows one state. Before implementation, decide explicitly:

```text
Empty state?
Loading state?
Error state?
Hover / focus / disabled?
Responsive breakpoints?
Existing component, or a new one?
```

---

# 6.4. Verify Against the Visual, Not Just the Code

```text
"It renders" is not "it matches."
```

Screenshot the implementation. Compare it directly against the source design — spacing, alignment, color, typography — with the same rigor Module 12 applies to reviewing code instead of assuming correctness.

Core principle:

> **The design is the specification. Read it before you write anything else.**

---

# MODULE 7 — AI COMMUNICATION FRAMEWORK

## Objective

Standardize how the team communicates with AI.

---

# 7.1. Context → Goal → Constraints → References → Output

Use this as the team's default prompt structure.

### 1. Context

What system is AI working in?

```text
We use React, TypeScript, React Query and Zustand.
```

### 2. Goal

What do we want to achieve?

```text
Implement a user filter modal.
```

### 3. Constraints

What must or must not happen?

```text
- No new dependencies.
- Strict TypeScript.
- Use existing Design System components.
- Do not modify the API layer.
```

### 4. References

Which existing code should AI follow?

```text
Use OrderFilterModal.tsx
as the reference implementation.
```

### 5. Output

What should AI return?

```text
First return an implementation plan.
Do not write code yet.
```

---

# 7.2. Persona Is Not the Most Important Part

Avoid overusing:

```text
You are a world-class senior frontend architect...
```

A real code example is usually more valuable than a long persona.

Prioritize:

* Real context
* Real code
* Real constraints
* Real requirements

---

# 7.3. Atomic Prompts

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

Then:

```text
Now connect this schema to the existing form.
```

---

# 7.4. Output Control

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

This gives engineers more control over the AI's scope.

---

# MODULE 8 — PLAN BEFORE CODE

## Objective

Teach the workflow:

> **Analyze → Plan → Review → Implement**

---

# 8.1. Analyze

Example:

```text
Analyze this requirement and the existing code.

Identify:
1. Relevant components.
2. Existing patterns.
3. State ownership.
4. Data dependencies.
5. Potential edge cases.

Do not write code.
```

---

# 8.2. Plan

Then:

```text
Propose an implementation plan.

Include:
- component structure
- state ownership
- data flow
- files to modify
- files to create
- potential risks
```

---

# 8.3. Human Review

The engineer evaluates:

```text
Is the architecture correct?
Is the state in the right place?
Does this follow our project conventions?
Are there unnecessary changes?
```

If incorrect:

```text
Revise the plan.

The filter state should be URL-owned,
not component-owned.
```

---

# 8.4. Implement

Only after the plan is approved:

```text
Implement step 1 only.

Do not modify unrelated files.
```

---

# 8.5. Incremental Implementation

Avoid:

```text
Generate 1,000 lines of code.
```

Prefer:

```text
Step 1
→ Review
→ Step 2
→ Review
→ Step 3
→ Review
```

Benefits:

* Smaller diffs
* Easier debugging
* Easier rollback
* Easier detection of assumptions
* Better architectural control

---

# MODULE 9 — AI AS A CODING PARTNER

## Objective

Use AI effectively during implementation.

---

# 9.1. Generate

Good use cases:

* Boilerplate
* Components
* Hooks
* Types
* Utility functions
* Tests

---

# 9.2. Transform

AI is particularly useful for transformations:

```text
Convert this JavaScript function to TypeScript.
```

```text
Convert this component to a custom hook.
```

```text
Replace lodash usage with a native implementation.
```

```text
Migrate this component from API A to API B.
```

---

# 9.3. Explain

Use AI to understand unfamiliar code:

```text
Explain this component.

Focus on:
- state flow
- side effects
- rendering conditions
- data dependencies
```

Rather than:

```text
What does this code do?
```

Ask AI to focus on a specific dimension.

---

# 9.4. Discover Existing Patterns

AI can help explore the codebase:

```text
Find the pattern used by this repository
for handling mutation errors.

Do not create a new abstraction.
```

AI should be used to **discover the codebase**, not only generate code.

---

# MODULE 10 — AI DEBUGGING

## Objective

Turn AI into a debugging partner.

---

# 10.1. Debugging Context

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

---

# 10.2. Avoid

```text
Why doesn't this work?
```

Prefer:

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

---

# 10.3. Hypothesis-Driven Debugging

Ask AI:

```text
List the top 3 possible causes.

For each cause:
- explain why
- identify evidence
- suggest how to verify it
```

The engineer verifies the hypothesis before applying a fix.

---

# 10.4. Debugging Loop

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

Avoid:

```text
Error
 ↓
AI randomly changes code
 ↓
Maybe it works
```

---

# 10.5. Debug Through Documentation, Not Silent Patches

Once a cause is confirmed, don't let the fix exist only as a diff. Require a short root-cause note before or alongside the fix:

```text
What was actually happening (the mechanism, not the symptom)
What assumption was wrong, and where it was made
Why this fix addresses the cause, not just the symptom
What else in the codebase might share the same wrong assumption
```

A diff answers "what changed." A root-cause note answers "what did we believe that turned out to be false" — the question that prevents the same bug elsewhere, and the one thing that survives outside any single session's context window (1.4).

---

# 10.6. Brownfield vs. Greenfield: Docs First, or Code First?

```text
Brownfield — an existing doc/comment/convention made a claim,
and the bug just proved it wrong:
  Confirm cause → fix the documented understanding FIRST →
  implement against the corrected doc → verify.

Greenfield — no prior claim exists, this is uncharted territory:
  Confirm cause → implement and verify the fix →
  THEN write the root-cause note / doc, now that it reflects reality.
```

Ask: **does something authoritative already claim to describe this behavior, and did the bug just disprove it?** If yes, that claim is actively misleading everyone until corrected — fix the doc first. If no, document after, once the fix is verified — don't document a guess.

Core principle:

> **Two disagreeing sources of truth — code and doc — are worse than a slower fix. Never patch and leave a wrong doc standing.**

---

# MODULE 11 — AI REFACTORING

## Objective

Use AI to improve existing code, not just generate new code.

---

# 11.1. Analyze Before Refactoring

```text
Analyze this component.

Identify:
- duplicated logic
- unnecessary state
- unnecessary renders
- coupling
- poor component boundaries
- difficult-to-test code

Do not refactor yet.
```

---

# 11.2. Compare Approaches

```text
Propose two refactoring approaches.

For each:
- complexity
- maintainability
- performance
- risk
```

The engineer chooses the approach.

---

# 11.3. Refactor Safely

```text
Implement approach 1.

Constraints:
- Preserve behavior.
- Do not change the public API.
- Do not introduce dependencies.
- Keep the diff minimal.
```

---

# MODULE 12 — AI CODE REVIEW

## Objective

Teach members not only to ask AI to write code, but also to **challenge the code**.

---

# 12.1. Review Prompt

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

---

# 12.2. Try to Break It

A useful pattern:

```text
Try to prove this implementation is wrong.

Find inputs, states or user interactions
that could cause incorrect behavior.
```

AI becomes an adversarial reviewer rather than a validator.

---

# 12.3. Review One Dimension at a Time

Avoid:

```text
Is this code good?
```

Instead:

```text
Review only for performance.
```

or:

```text
Review only for accessibility.
```

or:

```text
Review only for race conditions.
```

Focused reviews produce better results.

---

# MODULE 13 — VERIFICATION

## Objective

Core principle:

> **Generated code is not verified code.**

---

# 13.1. Verification Pyramid

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

AI does not replace verification.

---

# 13.2. TypeScript

Check for:

* Type errors
* Incorrect generics
* Unsafe narrowing
* `any`
* Incorrect API contracts
* Null/undefined handling

---

# 13.3. Lint

Check for:

* Hook misuse
* Unused code
* Incorrect dependency arrays
* Project convention violations

---

# 13.4. Tests

AI can generate tests, but engineers must verify:

> **Does the test actually verify behavior?**

Do not accept tests simply because they increase coverage.

---

# 13.5. E2E

For user-facing behavior:

```text
User action
→ UI
→ API
→ State
→ UI result
```

E2E tests provide an important source of truth.

---

# 13.6. Runtime Verification

Compilation does not mean correctness.

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

---

# MODULE 14 — FRONTEND-SPECIFIC AI REVIEW

## Objective

Apply AI specifically to frontend engineering concerns.

---

# 14.1. React / Vue

Review:

* Hooks/composables
* Dependency arrays
* Effects
* Reactive dependencies
* Unnecessary renders
* State duplication
* Component boundaries

---

# 14.2. State Management

Ask:

```text
Local state?
Server state?
Global state?
URL state?
```

AI can easily put server state into a global store when it does not belong there.

---

# 14.3. Data Fetching

Review:

* Cache behavior
* Stale data
* Race conditions
* Duplicate requests
* Retry behavior
* Cancellation
* Loading state
* Error state

---

# 14.4. Performance

```text
Analyze potential performance issues.

Focus on:
- unnecessary renders
- expensive calculations
- large lists
- memoization
- bundle size
- network requests
- image loading
```

---

# 14.5. Accessibility

Review:

* Semantic HTML
* Keyboard navigation
* Focus management
* ARIA
* Screen reader behavior
* Dialog behavior
* Form labels

---

# 14.6. Responsive UI

Check:

```text
Mobile
Tablet
Desktop
Long content
Empty state
Error state
Loading state
```

---

# MODULE 15 — AI FOR TESTING

## Objective

Use AI to improve test quality.

---

# 15.1. Generate Tests

```text
Generate tests based on the existing testing pattern.

Cover:
- happy path
- empty state
- validation error
- API error
- boundary cases
```

---

# 15.2. Discover Edge Cases

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

---

# 15.3. Review Test Quality

```text
Review these tests.

Identify tests that:
- test implementation details
- have weak assertions
- duplicate coverage
- do not test meaningful behavior
```

---

# MODULE 16 — AI FOR DOCUMENTATION & KNOWLEDGE

AI is not only for coding.

Use AI to:

* Explain unfamiliar code.
* Generate README files.
* Generate API documentation.
* Summarize architecture.
* Create migration guides.
* Create onboarding documentation.
* Explain legacy code.

Example:

```text
Analyze this module and generate a developer-oriented
architecture overview.

Include:
- responsibilities
- dependencies
- data flow
- important abstractions
- known risks
```

---

# MODULE 17 — AI ANTI-PATTERNS

## 17.1. Vague Prompting

```text
Make this better.
```

No context, criteria, or expected output.

---

## 17.2. Premature Coding

```text
Build the entire feature.
```

before architecture has been determined.

---

## 17.3. Context Dumping

Providing the entire repository without identifying what is relevant.

---

## 17.4. Blind Acceptance

```text
AI generated it.
It compiles.
Done.
```

Compilation does not prove correctness.

---

## 17.5. AI-Driven Architecture

```text
AI suggested Zustand.
Therefore, we should use Zustand.
```

Wrong mindset.

AI can suggest.

Engineers evaluate the trade-offs.

---

## 17.6. Endless Conversation

A single conversation keeps accumulating unrelated work:

```text
Task 1
→ Fix
→ Another fix
→ Another feature
→ Refactor
→ Bug
→ Another feature
```

The context becomes polluted.

When the scope changes significantly:

> **Start a new focused context.**

---

## 17.7. Over-Automation

Not every task should be delegated to AI.

Important decisions such as:

* Architecture
* Security
* Data model
* Major dependencies
* Breaking changes

should have explicit human ownership.

---

# MODULE 18 — STANDARD AI WORKFLOW FOR THE TEAM

The team can standardize on this workflow:

```text
                REQUIREMENT
                     │
                     ▼
                 UNDERSTAND
                     │
                     ▼
                 DECOMPOSE
                     │
                     ▼
                   CONTEXT
                     │
                     ▼
                    PLAN
                     │
                     ▼
              HUMAN REVIEW PLAN
                     │
                     ▼
                 IMPLEMENT
                     │
                     ▼
                  VERIFY
                     │
              ┌──────┼──────┐
              ▼      ▼      ▼
             Type   Test   Runtime
              │      │      │
              └──────┼──────┘
                     ▼
                  AI REVIEW
                     │
                     ▼
                HUMAN REVIEW
                     │
                     ▼
                    MERGE
```

---

# MODULE 19 — TEAM AI CODING STANDARD

The team can adopt the following principles.

### Rule 1

> **AI should not make architectural decisions without human review.**

### Rule 2

> **Non-trivial tasks should be planned before implementation.**

### Rule 3

> **Use existing project patterns whenever possible.**

### Rule 4

> **Prefer minimum sufficient context.**

### Rule 5

> **Keep AI-generated changes small and reviewable.**

### Rule 6

> **Never trust generated code without verification.**

### Rule 7

> **Do not introduce dependencies without justification.**

### Rule 8

> **Business requirements are the source of truth, not AI output.**

### Rule 9

> **Existing code is a better reference than a textual description.**

### Rule 10

> **The engineer owns the final result.**

---

# FINAL FRAMEWORK

The entire training can be summarized as one continuous loop:

```text
                    ┌─────────────┐
                    │ Understand  │
                    └──────┬──────┘
                           ↓
                    ┌─────────────┐
                    │ Decompose   │
                    └──────┬──────┘
                           ↓
                    ┌─────────────┐
                    │   Context   │
                    └──────┬──────┘
                           ↓
                    ┌─────────────┐
                    │    Plan     │
                    └──────┬──────┘
                           ↓
                    ┌─────────────┐
                    │ Implement   │
                    └──────┬──────┘
                           ↓
                    ┌─────────────┐
                    │   Verify    │
                    └──────┬──────┘
                           ↓
                    ┌─────────────┐
                    │   Review    │
                    └──────┬──────┘
                           ↓
                    ┌─────────────┐
                    │   Refine    │
                    └──────┬──────┘
                           │
                           └──────────→ Understand
```

# OFFICIAL CLAUDE CODE DOCUMENTATION

This curriculum teaches transferable habits. For exact commands, flags, and mechanics in the tool itself, the primary sources are:

* [Best practices](https://code.claude.com/docs/en/best-practices) — canonical version of most of Modules 0–17.
* [Memory (CLAUDE.md & auto memory)](https://code.claude.com/docs/en/memory) — real mechanics behind the Advanced Track.
* [Sessions](https://code.claude.com/docs/en/sessions) — real commands behind Module 2 and the Managing a Session playbook.
* [Common workflows](https://code.claude.com/docs/en/common-workflows) — ready-to-adapt prompt recipes, same spirit as the Playbooks section.
* [Prompt library](https://code.claude.com/docs/en/prompt-library) — a larger, searchable set of copy-paste prompts by task and role.

When a module here and the official docs disagree, the official docs win — Claude Code changes faster than this curriculum will be updated.

---

# THE 5 CORE PRINCIPLES

> **1. AI doesn't know your codebase unless you show it.**

> **2. Don't ask AI to solve a problem you haven't decomposed.**

> **3. Plan before coding when the task is non-trivial.**

> **4. Give AI patterns, not just instructions.**

> **5. Never trust generated code without verification.**

## Final Message to the Team

> **The goal of AI-assisted development is not to make AI write more code.**
>
> **The goal is to help engineers deliver correct, maintainable software faster.**

---

# PLAYBOOKS — BEST PRACTICE BY SITUATION

Modules 0–19 teach the underlying concepts. Playbooks are short, situation-specific recipes on top of them — what to actually do when you're facing one of these scenarios, with pointers back to the concept for the reasoning. Full versions live in `docs/guide/05-playbooks/`.

### Starting a New Feature

Requirement → contract (Module 4) → minimum sufficient context (Module 5) → Analyze → Plan → Human Review Plan (Module 8) → incremental implementation, one reviewable step at a time. The failure to watch for is skipping straight to code before the contract exists.

### Migrating Code

A migration is a faithful port, not a redesign. Establish the explicit mapping between old and new shapes first, migrate one small vertical slice as proof of the mapping, verify behavior parity old-vs-new, then repeat slice by slice. State "preserve behavior exactly, do not improve along the way" as an explicit constraint — the single biggest failure mode is AI quietly "improving" code mid-migration, which makes parity impossible to verify.

### Debugging a Bug

The fast-reference version of Module 10: give full context (expected, actual, error, what's been tried), ask for hypotheses with evidence before a patch, confirm the cause yourself, write the short root-cause note, and run the brownfield/greenfield check on any doc your bug just disproved.

### Updating Existing Code

Distinct from both "new feature" and "migration": you're changing code that already works, for a reason other than a bug. Ask AI to find the existing pattern first — "do not create a new abstraction" — define the smallest diff, and explicitly scope out unrelated cleanup riding along with the change.

### From Docs to Code

When a spec, RFC, or ticket is the source of truth, treat it as the contract the same way Module 4 treats a requirement sentence. Ask AI to extract the contract and list what's ambiguous before generating code. Resolve ambiguity against the doc owner — don't let AI silently fill gaps with plausible defaults.

### Using a Skill

The consumer side of a packaged, on-demand instruction set (a skill): recognizing one is relevant, letting it front-load conventions you'd otherwise restate every session, and knowing when to fall back to plain prompting instead of forcing a poor-fit task through a skill. The Advanced Track below covers the author side.

### Dividing Work with Sub-Agents

The recipe version of Module 4.6: draw the dependency graph, run independent branches as separate sessions or parallel sub-agents scoped to their own minimum sufficient context, keep dependent work sequential, and always plan an explicit integration step reviewed as carefully as any other task.

### Managing a Session

The recipe version of Module 2: start fresh when the task category changes; continue when iterating on the same work; compact to keep going on the same task with more room; clear when the next task shares nothing with the current one; restate load-bearing constraints before a risky step.

---

# ADVANCED TRACK — CONTRIBUTING TO THE TEAM'S AI TOOLING

Modules 0–19 and the Playbooks above teach using AI well on your own tasks. This track is for engineers ready to make the *next* engineer's AI usage better too — by shaping the shared instructions, skills, and workflows the whole team's sessions run on, instead of only consuming what already exists. Full versions live in `docs/guide/07-advanced-track/`.

### Anatomy of a Claude Project

A Claude Code project has several distinct mechanisms for carrying knowledge between sessions, and picking the right one matters:

* **Standing instructions** (e.g. `CLAUDE.md`) — always-on context and rules, loaded into every session whether relevant to the current task or not.
* **Skills** — packaged, named, on-demand instruction sets, invoked when a task matches, costing nothing until triggered.
* **Settings/permissions** — project-level configuration for what tools and commands are auto-allowed vs. require approval.
* **Subagents** — separate agent configs with their own scoped tools and prompt, delegated to for a specific kind of task to keep the main session's context clean.
* **Hooks** — shell commands that fire automatically on events, for enforcement or automation that shouldn't depend on the model remembering to do something.

### Skills vs. Rules

The trade-off: a rule in standing instructions costs context budget on every single turn of every session, whether or not it's relevant — but it's guaranteed to be seen. A skill costs nothing until invoked — but only fires if it's discoverable, which means it needs a specific, well-scoped name and description, not a vague one. Practical heuristic: a constraint that applies to *almost every* task in the repo belongs in standing instructions; a specific, occasionally-needed procedure belongs in a skill.

### Workflows & Contributing a Skill

Multi-agent workflows orchestrate many sub-agents in parallel for structured work at a scale beyond one sequential session — a large migration, a broad audit. They are not a default upgrade over plain prompting or a single skill; most day-to-day tasks don't need that structure, and over-applying it is its own anti-pattern (Module 17's "Over-Automation," aimed at the tooling instead of the task).

The practical contribution loop: notice a pattern repeating across your own sessions (a prompt you keep re-typing, steps you keep re-explaining) → draft it as a skill with a clear name and description → test it on a real task, not a toy one → get a teammate to try it and give feedback → land it for the team under the same review bar as any other shared code. This is the Team AI Coding Standard's ten rules applied to the team's shared tooling instead of to one task.
