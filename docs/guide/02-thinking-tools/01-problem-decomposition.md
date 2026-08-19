# Module 4 — Problem Decomposition

*[← Previous: Choosing Your Model & Effort](../01-foundations/04-choosing-model-and-effort.md) | [Next: Context Engineering →](./02-context-engineering.md)*

## Why this matters

Picture a ticket that reads: "Add a checkout flow with a promo code field and an order summary." An engineer pastes that straight into their AI tool and asks it to "build it." Ten minutes later they have a component that looks plausible, stores the cart in local component state, refetches the whole cart on every keystroke in the promo field, and has no idea that promo codes are supposed to be validated server-side before the summary updates. None of that is because the AI is bad at coding — it produced working syntax on the first try. It's because nobody decided the contract, the data flow, or the state ownership before asking for code, so the AI quietly decided all of it for you, badly. This module is about doing that thinking yourself, first, so the AI is executing your design instead of inventing its own.

## Objective

Do not immediately ask AI to code when you receive a requirement. Instead, transform the requirement into a sequence of decisions before a single line of implementation exists:

```text
Requirement
→ Contract
→ Data
→ State
→ Architecture
→ Implementation
→ Verification
```

Everything in this module is about how to do the first few steps of that sequence — Contract, Data, State, and Architecture — on paper or in a scratch file, before you open a prompt.

---

## 4.1. Requirement → Contract

A requirement is a sentence. A contract is a type. The gap between those two things is exactly where most AI-generated frontend code goes wrong, because natural language is ambiguous and TypeScript is not. If you hand an AI tool a vague sentence, it has to silently resolve every ambiguity itself — what shape is a "role," is status optional, is the list paginated — and it will resolve them however is statistically convenient, not however your backend actually works. Writing the contract first forces you to resolve those ambiguities as a human, using knowledge the AI doesn't have (your actual API, your actual domain model), and it gives the AI something unambiguous to implement against instead of something to guess at.

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

This is the **contract**. Once it exists, "implement the filter form" becomes a much smaller, much less ambiguous task, because the shape of the answer is already fixed.

AI should not invent a contract when an existing contract already exists in the system. If `UserFilter` or `UserListResponse` is already defined somewhere in the codebase, your job is to find it and hand it to the AI, not to let the AI re-derive its own version that happens to almost match.

**Worked example — checkout flow.** Take the ticket from the intro: "Add a checkout flow with a promo code field and an order summary." Before touching AI, write the contract:

```ts
type PromoCodeValidationRequest = {
  code: string;
  cartId: string;
};

type PromoCodeValidationResult =
  | { status: "valid"; discountAmount: number; discountType: "fixed" | "percentage" }
  | { status: "invalid"; reason: "expired" | "not-found" | "not-applicable" };

type OrderSummary = {
  subtotal: number;
  discount: number;
  shipping: number;
  total: number;
};
```

Notice what this already forces you to decide: promo validation is a discriminated union, not a boolean, because "invalid" has reasons the UI needs to display differently. That decision belongs to you, informed by how your backend actually responds — it is not something you want an AI guessing at from the word "promo code field" alone.

Avoid:

```text
Build a checkout page with a promo code field.
```

Prefer:

```text
Implement the checkout summary component using this contract:

type PromoCodeValidationResult = ...
type OrderSummary = ...

Do not invent additional fields.
Do not change the shape of PromoCodeValidationResult.
```

---

## 4.2. Identify the Data Flow

A contract tells you the shape of data. Data flow tells you where that data travels and what transforms it along the way. This step matters because frontend bugs — a stale total, a filter that resets on refresh, a summary that flashes the wrong value — are almost always data-flow bugs, not syntax bugs. If you can't draw the path data takes from user input to rendered output, you can't reliably ask AI to implement it, and you definitely can't judge whether what it hands back is doing the right thing at the right step.

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

**Worked example — notifications panel.** Say the requirement is: "Show a bell icon with an unread count; clicking it opens a panel of recent notifications; marking one as read updates the count immediately." Draw the flow before prompting:

```text
WebSocket / Poll
 ↓
Notification Cache (React Query)
 ↓
Unread Count (derived, not stored)
 ↓
Bell Icon
 ↓
User clicks bell
 ↓
Panel opens (local UI state: isOpen)
 ↓
User marks one as read
 ↓
Optimistic mutation
 ↓
Cache updated
 ↓
Unread Count re-derives automatically
```

The key decision this diagram exposes: the unread count is *derived* from the cached notification list, not a separately-tracked number. If you skip this step, an AI implementation will very plausibly track `unreadCount` as its own piece of state updated by a separate `useEffect` — which then drifts out of sync with the actual list the moment a notification arrives from the websocket while the panel is closed. Naming that derivation explicitly, before prompting, is what prevents that class of bug.

---

## 4.3. State Ownership

This is one of the areas where AI can easily make poor decisions, because "where should this state live" is an architectural judgment call that depends on how the state is used elsewhere in your app, not on the current task in isolation. An AI tool sees a single component that needs a boolean or a list, and the path of least resistance is always `useState` in that component or, if it looks "global," a new global store. Neither instinct considers whether the value is actually server data, whether it should be shareable via URL, or whether three other components already need it. Deciding ownership yourself, before implementation, is what keeps state colocated where it belongs instead of duplicated or promoted to global scope by default.

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

**Worked example — settings page.** Requirement: "Users can edit their profile, notification preferences, and connected apps from a tabbed settings page. The active tab should be shareable via link, and unsaved changes should warn before navigating away." Classify before prompting:

| State | Category | Why |
|---|---|---|
| `activeTab` | URL state (`?tab=notifications`) | Must be shareable and survive refresh |
| `profile`, `notificationPrefs`, `connectedApps` | Server state | Sourced from and persisted to the API; React Query owns caching |
| `isDirty` per tab | Local UI state | Only meaningful while the form is mounted; not persisted |
| `currentUser.id` used to scope the fetch | Global client state | Already owned by your auth store; settings just reads it |

Without this table, a common AI-generated mistake is putting `activeTab` in `useState` (so refreshing the page always lands back on the first tab, which fails the "shareable" requirement) or putting the fetched `profile` object into a global Zustand store "for consistency" (which then fights with React Query's own cache and creates two sources of truth for the same server data). Neither is a syntax error — both compile fine — which is exactly why this decision has to be made by you, not discovered by debugging later.

Avoid:

```text
Add state for the settings page.
```

Prefer:

```text
activeTab is URL state, synced via useSearchParams.
profile, notificationPrefs, connectedApps are server state via React Query.
isDirty is local component state, one per tab, not persisted.
Implement the tab container using this ownership model.
```

---

## 4.4. Component Decomposition

Once data flow and state ownership are settled, component boundaries mostly fall out of them — a component's boundary should usually match the scope of the state and data it's responsible for. This is a place where AI is genuinely useful as a proposer: it's good at pattern-matching a reasonable tree shape from a description. But "reasonable-looking" and "correct for your codebase's conventions" are different bars, and only you know the second one — whether your team nests presentational components under a `components/` folder next to the feature, whether you split "container" and "view" components, or whether a `Modal` in this codebase is always a compound component. Treat the AI's proposed tree as a draft, not a decision.

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

AI can suggest the structure. The engineer makes the final decision.

**Worked example — product comparison table.** Requirement: "Let users select up to 4 products and compare their specs side by side, with the ability to remove a product from the comparison." A first-pass AI suggestion, and the revision it typically needs:

```text
AI proposal:
ComparisonPage
├── ComparisonTable          (fetches products, owns selection state, renders everything)

Revised, after applying the data-flow and state-ownership decisions from 4.2/4.3:
ComparisonPage
├── ProductPicker                     (local: search input, dropdown open state)
├── ComparisonTable
│   ├── ComparisonColumn (× selected products)   (server state: one product's specs)
│   │   └── RemoveFromComparisonButton
│   └── ComparisonRow (× spec attribute)          (pure, presentational)
└── selectedProductIds                            (URL state: ?compare=id1,id2,id3)
```

The single-component version an AI defaults to isn't wrong syntactically, but it collapses everything into one place: the search-input state, the fetch-per-product logic, and the pure row-rendering logic all end up entangled, which makes the "remove a product" interaction and the "shareable comparison link" requirement both harder to implement correctly and harder to test in isolation. Deciding the boundaries first — one column per product because each is its own server-state fetch, rows as pure presentational components because they only format data — is what keeps the eventual implementation testable.

---

## 4.5. Task Decomposition

The last step before you open your AI tool is turning the whole plan into an ordered list of small, independently reviewable tasks. This matters for a reason that has nothing to do with AI capability: a single giant diff is unreviewable. Even a perfect implementation, delivered in one 800-line response, is something you cannot meaningfully evaluate line by line — you'll skim it, half-trust it, and merge assumptions you never actually checked. Small tasks also let you catch a wrong assumption (wrong state ownership, wrong contract) after task 1 instead of after task 6, when it's cheap to fix instead of tangled through everything downstream.

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

**Worked example — checkout flow, decomposed.** Returning to the checkout ticket from the intro, once the contract, data flow, and component tree are decided, the task list looks like this:

```text
Task 1: Define PromoCodeValidationResult and OrderSummary contracts
Task 2: Implement OrderSummary component (pure, given a summary object)
Task 3: Implement promo code input + validation mutation
Task 4: Wire promo result into order summary (discount applied)
Task 5: Handle invalid/expired/not-applicable states in the UI
Task 6: Add tests for each PromoCodeValidationResult variant
```

Each task is independently reviewable and, importantly, independently *revertable* — if Task 3's validation mutation turns out to hit the wrong endpoint, you find that out before Task 4 has built the discount-display logic on top of it, and undoing one task doesn't unwind five others with it.

---

## 4.6. Divide & Conquer: Parallelizing with Sub-Agents

A task list like the one above is usually presented — and usually executed — as a sequence: Task 1, then Task 2, then Task 3. That's the right default when tasks genuinely depend on each other's output, the way Module 8's incremental implementation assumes. But decomposition has a second, easy-to-miss payoff once the list exists: it also tells you exactly which tasks *don't* depend on each other, and those are candidates for running in parallel — either as separate AI sessions you juggle yourself, or as multiple sub-agents an orchestrating session dispatches at once — instead of forcing everything through one sequential thread.

**Decide dependency before parallelism, not the other way around.** Look at the finished task list and ask, for each pair of tasks, "does this one need the other one's actual output to start?" Two tasks are independent if each could be hand-checked in isolation without knowing the other exists. Two tasks are dependent if one produces a contract, a component, or a decision the other consumes.

Take the checkout task list from Section 4.5:

```text
Task 1: Define PromoCodeValidationResult and OrderSummary contracts
Task 2: Implement OrderSummary component (pure, given a summary object)
Task 3: Implement promo code input + validation mutation
Task 4: Wire promo result into order summary (discount applied)
Task 5: Handle invalid/expired/not-applicable states in the UI
Task 6: Add tests for each PromoCodeValidationResult variant
```

Task 1 blocks everything — nothing else can start until the contracts exist, so it has to run first and alone. But once Task 1 is done, Tasks 2 and 3 don't depend on each other at all: `OrderSummary` only needs the `OrderSummary` shape, and the promo input only needs the `PromoCodeValidationResult` shape — neither needs the other's implementation to be built correctly. Task 4 depends on both 2 and 3 being done, so it has to wait. This is the same shape as the state/data-flow diagram from Sections 4.2–4.3, just applied to *execution order* instead of *runtime behavior*: draw the dependency graph, and the tasks with no edge between them are your parallelization candidates.

```text
Task 1 (contracts)
   │
   ├──▶ Task 2 (OrderSummary)      ──┐
   │                                  ├──▶ Task 4 (wire together) ──▶ Task 5 ──▶ Task 6
   └──▶ Task 3 (promo input)       ──┘
```

**Why this matters given Module 1–2.** Running Tasks 2 and 3 in the same sequential session, one after the other, means Task 3's context window still carries all of Task 2's exploration, files, and back-and-forth — noise for a task that never needed it (Module 5.3's context pollution, arriving from an unrelated *prior task* instead of an over-eager paste). Running them as two separate sessions, or two sub-agents dispatched from one orchestrating session, means each one gets a clean window scoped to exactly what it needs (Module 5's minimum sufficient context), they genuinely run concurrently instead of one waiting on the other, and a wrong turn in Task 3 (Module 6's "AI-Driven Architecture" caveat aside) never contaminates Task 2's context at all.

**When to keep it sequential instead.** Not everything on a task list should be parallelized just because it's technically possible to split further. Force tasks into one sequential thread when:

* One task's output is the next task's actual input (Task 1 → everything, in the example above) — there's nothing to parallelize because the dependency is real, not incidental.
* The tasks are small enough, and reviewing each one takes long enough, that the overhead of managing multiple parallel threads (context-switching between them, reconciling their outputs) costs more than the time saved.
* You need to hold a single evolving decision in your head across steps — Module 8's plan-then-implement flow, where each increment is reviewed against a shared mental model that's expensive to duplicate across parallel threads.

**Merging parallel work back together.** The one step Divide & Conquer adds that pure sequential work doesn't need is an explicit integration step — Task 4 in the example above exists specifically to reconcile what Tasks 2 and 3 produced. Treat that integration task with the same care as Module 8's human review: verify the two independently-built pieces actually agree on the contract from Task 1 (a common failure is two parallel branches each interpreting an ambiguous part of the contract slightly differently), not just that each one compiles on its own.

**Worked example — a larger feature.** A settings page redesign touches five independent tabs (profile, notifications, security, billing, connected apps), each following the same established tab pattern (Module 4.4's component-decomposition example) and each reading and writing a different slice of user data with no interaction between tabs. This is a strong Divide & Conquer candidate: one task defines the shared tab-container contract (sequential, blocking), then all five tab implementations are genuinely independent of each other and can be dispatched as five parallel sub-agents, each given only its own tab's contract and the one reference tab pattern (Module 5's minimum sufficient context, applied per sub-agent) rather than all five tabs' worth of context stacked into one session. A human then reviews each tab independently, exactly the way Module 4.5 already argues small tasks should be reviewed — the only new step is that "independently reviewable" here also means "independently *built*," concurrently, instead of one after another.

---

## Key Takeaways

* Convert every requirement into `Contract → Data → State → Architecture` before opening your AI tool — don't skip straight to "build this."
* A contract is a type, not a sentence. Write it down, and reuse an existing one instead of letting AI invent a new one that almost matches.
* Draw the data flow so you know who owns state, who transforms it, and where the server/client boundary sits — most frontend bugs live in this diagram, not in syntax.
* Classify every piece of state as local UI, server, global client, or URL state *before* implementation — this is one of the areas AI gets wrong most often when left to decide alone.
* Let AI propose component structure, but you decide the final boundaries based on your team's conventions, not on what looks plausible.
* Break the task into small, ordered, independently reviewable steps — one meaningful change at a time, never "build the whole feature."
* Once the task list exists, draw the dependency graph between tasks. Tasks with no edge between them are Divide & Conquer candidates — run them as separate sessions or parallel sub-agents instead of forcing them through one sequential, increasingly noisy thread. Tasks with a real dependency stay sequential.
* Parallel work needs an explicit integration step. Review it as carefully as any other task — the most common failure is two independently-built pieces silently disagreeing about a contract that was ambiguous in one direction.

## Try It Yourself

1. Before you open your AI tool for your next ticket, do not type anything into it yet. On paper or in a scratch file, write out: the contract (types), the data flow diagram, and a table classifying every piece of state the feature needs (local / server / global / URL). Only after that's done, turn it into task list of 4-8 small steps.
2. Take a feature you already shipped recently — one that had a state-related bug (stale data, wrong tab on refresh, state that didn't sync with the URL, etc.). Reconstruct, after the fact, which state-ownership category the buggy piece of state should have been in, and compare it to where it actually lived. Note what decomposition step, if you'd done it up front, would have caught the mistake before it shipped.
3. Take your current task list for an in-progress feature and draw the dependency graph from Section 4.6. Identify at least one pair of tasks that has no real dependency between them, and run them as two separate sessions (or sub-agents, if your tooling supports dispatching them) instead of one sequential thread. Compare the total time and the context-window cleanliness of each task's output against doing the same two tasks back-to-back in one session.

---

*[← Previous: Choosing Your Model & Effort](../01-foundations/04-choosing-model-and-effort.md) | [Next: Context Engineering →](./02-context-engineering.md)*
