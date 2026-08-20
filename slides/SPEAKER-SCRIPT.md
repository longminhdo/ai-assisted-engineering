# Speaker Script — AI-Assisted Frontend Engineering (47 slides)

How to use this file: each entry gives the slide's on-screen title, what's visually happening, and a **Say** block written as spoken narration — read it in your own voice, don't recite it word-for-word. The 14 "Real Case" slides are new since the last pass through this deck; they always follow the concept slide they illustrate, so pace them as one beat ("here's the idea — here's what it looks like when it goes wrong"), not as a separate topic. Trim the Real Case slides first if you're short on time, since the core loop still holds without them.

**Pacing:** the Say blocks alone run about 3,200 words, roughly 22-25 minutes read straight through at a natural pace. That is not the real runtime: slides 14, 18, and 25 are click-driven (7, 4, and 7 stops respectively) and each stop deserves a beat before you move on, and the two closing slides (46, 47) call for a deliberate pause. Budget 35-45 minutes for a full run including those pauses, and more on top of that for audience questions.

---

## 1. Title — "Better Decisions, Not More Code."

**On screen:** Full-bleed hero, kicker + massive display type.

**Say:** Welcome. Before we get into any tooling, I want to set the frame for this whole session. This is not a training about typing faster or getting AI to write more code for you. It's a training about making better engineering decisions, faster, with AI as part of how you work — not a replacement for the judgment you already bring. Everything today builds toward that one idea.

---

## 2. Training Objective — the 7-step loop

**On screen:** Radial loop diagram, 7 nodes, core quote beneath.

**Say:** Here's the loop we'll keep coming back to all session: Understand, Decompose, Context, Plan, Implement, Verify, Refine — and Refine feeds back into Understand. Notice it's a circle, not a line. It never "finishes" for a codebase you keep working in. The question underneath all of this is: how do engineers use AI to make better decisions and deliver faster, without giving up quality? That's the whole training in one sentence.

---

## 3. Expected Capabilities — "What You'll Walk Away With"

**On screen:** Asymmetric 7-tile bento grid, A through G.

**Say:** By the end of today you should be able to do seven things: read your own AI mindset correctly, manage sessions and model choice deliberately, decompose problems before coding, engineer context instead of dumping it, communicate with AI using a repeatable structure, run the full AI development workflow — planning, implementing, debugging, refactoring, reviewing, testing — and verify what comes back before you trust it. We'll go through each.

---

## 4. Section Divider — 01 Foundations

**Say:** Let's start with foundations — the mental model underneath everything else.

---

## 5. Pilot vs. Copilot

**On screen:** 60/40 split — Engineer column (accent-underlined) vs. AI column (muted).

**Say:** The shift I want you to make today is from "AI writes code for me" to "I direct AI." The engineer stays the pilot — architecture, business logic, security, final review are yours, always. AI is the copilot — exploration, proposals, generation, a first pass at review. AI can propose. Engineers decide. Watch what happens when that line gets blurry.

---

## 6. Real Case — Silent Assumptions

**On screen:** Case Study card — 3 beats + code example + best-practice strip.

**Say:** Here's what "blurry" looks like in practice. Someone asks for "the user management page" and nothing else. AI has to fill every gap itself — client-side pagination, local component state, no permission check, a generic error toast. Two sprints later the table breaks past 500 rows, some admins can see rows they shouldn't, and failed requests fail completely silently. Nobody lied to AI — nobody told it anything, so it guessed, confidently. The fix isn't "be more careful," it's: state every decision you're not making explicitly, because AI will make it for you, silently.

---

## 7. Everything Lives in the Window

**On screen:** Filling-glass metaphor, older token blocks fading toward the bottom.

**Say:** Here's the mechanic underneath all of this. AI has no memory between sessions — everything it can use to answer you is whatever is currently inside its context window. If it's not in the window, it does not exist for this response. That's not a limitation to work around occasionally, it's the central fact that every other technique in this deck is a strategy for managing.

---

## 8. Real Case — The Buried Constraint

**Say:** Concretely: turn one of a session, someone says "constraints — strict TypeScript, no new dependencies, do not modify the API layer." Forty turns later, deep in an unrelated refactor, AI "simplifies" a response shape in userService.ts to tidy up a type — and three other callers silently break. The constraint was never repeated, so by turn 40 it had quietly lost its weight. The lesson: restate load-bearing constraints before a risky step. Don't assume turn one still carries the same weight at turn forty.

---

## 9. Start Fresh, or Keep Going?

**On screen:** Horizontal decision flow — three gates routing to Start Fresh vs Keep Going.

**Say:** So how do you manage that window deliberately? Three questions: has the task category changed? Is an old, abandoned approach still lingering in this conversation? Are you still iterating on one piece of work? If the category changed or an old approach is still hanging around — clear, blank slate. If you're mid-iteration — compact, same task, more room, but the summary is lossy. Session boundaries should track the task, not the clock.

---

## 10. Real Case — Compact vs Clear, in Practice

**Say:** Fifteen turns into a stale-closure bug, same file, same hypothesis chain — compact is the right call, it keeps your evidence trail and gives you more room. Now the next request is "add a dark mode toggle" — totally unrelated. If you keep going in that same session instead of clearing, AI can blend the abandoned debugging approach into the new toggle work and half-apply a fix nobody asked for. Same tool, wrong moment, and the mistake is invisible until you're debugging the debugging.

---

## 11. Resource by Risk, Not Size

**On screen:** 2x2 matrix — Size vs. Judgment Required.

**Say:** Same logic applies to model tier and reasoning effort. It's not about how big the task is, it's about how much judgment it needs. Renaming a prop across thirty files is large but mechanical — low effort. Deciding how optimistic UI should reconcile with a websocket push is small but judgment-heavy — that needs high effort. A fast, shallow answer to a hard question looks exactly as confident as a well-reasoned one. That's what makes under-provisioning dangerous.

---

## 12. Real Case — The Under-Provisioned Fix

**Say:** That websocket reconciliation task actually shipped once at default, low effort, because it "looked small." The first plausible answer overwrote optimistic state unconditionally whenever a socket message arrived. It shipped, and days later the team was fielding UI-flicker complaints because out-of-order messages were dropping legitimate optimistic updates. Nothing about the output looked uncertain — that's exactly the danger. When you're unsure, round up.

---

## 13. Section Divider — 02 Thinking Tools

**Say:** Next section: the thinking tools you apply before you ever open the editor.

---

## 14. Requirement → Contract, Not Code

**On screen:** 7-stage horizontal pipeline, clickable — Requirement through Verification.

**Say:** This is the core discipline of decomposition: don't go straight from a requirement to code. Go Requirement, Contract, Data, State, Architecture, Implementation, Verification. A contract is a type, not a sentence — "users can filter by keyword, role, and status" becomes an actual `UserFilter` type before anyone writes a line of UI.

*(Click through each stage live — the detail panel below updates as you go. One line per stage, pause a beat after each click for it to land:)*

1. **Requirement** — "Let people filter the user list by keyword, role, and status." Still ambiguous — role names undefined, edge cases unstated.
2. **Contract** — that sentence becomes the `UserFilter` type. Now break the work into small, ordered, reviewable tasks — independent ones run in parallel, dependent ones run in sequence.
3. **Data** — define the fields, their types, and nullability, before deciding how the UI will use them.
4. **State** — enumerate every state the component can be in, not just the happy path: idle, loading, error, ready.
5. **Architecture** — decide which layer owns the contract, hook, store, or service, and how it flows up to the UI.
6. **Implementation** — only now does code get written, against the contract, not from a paragraph of prose.
7. **Verification** — check the build against the contract: every field, every state, every edge case, all accounted for. Not just "does it run."

---

## 15. Good Context Beats a Clever Prompt

**On screen:** 5-tier vertical pyramid — Task at the tip, Full Repository at the base.

**Say:** Once you know what you're building, the next question is what AI needs to see to build it. Climb the pyramid from the task outward — task, contracts, patterns, related modules, full repo — and stop as soon as you have enough. More context doesn't mean safer; it can cause AI to pick the wrong pattern because everything looks equally weighted. Show AI the pattern, don't describe it in prose.

---

## 16. Real Case — Context Pollution in the Wild

**Say:** Here's what "more context" actually costs. Someone hands AI the entire /api folder, every component, every hook, every stylesheet — "just in case." AI picks the oldest modal pattern in the repo, from a page nobody maintains anymore, because it's sitting right next to the current pattern with equal weight. What should have gone in: the UserFilter type, the filter API, OrderFilterModal.tsx, the existing Form and Select, one similar feature. Nothing else. Minimum sufficient context — everything needed, nothing extra.

---

## 17. The Design Is the Spec

**On screen:** Before/after split — vague prose (muted) vs. an annotated wireframe with exact callouts.

**Say:** Same principle applies to visual work. "Rounded corners and a shadow, feels clean and modern" tells AI nothing concrete — no radius, no blur, no opacity. Bring the actual frame, annotated with real values, and AI has something it can implement against instead of guess at. The design is the specification — read it before you write anything else.

---

## 18. Real Case — The State Nobody Designed

**Say:** The Figma frame handed over showed one populated table, twelve rows, nothing else. So that's all that got built — no empty state, no loading skeleton, no error state, because none of them were in the frame. The first customer with zero users saw a bare white card floating under the header. A static frame hides every state that isn't the happy path — decide empty, loading, error, and hover explicitly, before implementation, not after a support ticket.

---

## 19. Write Docs Agents Can Actually Use

**On screen:** Two stacked doc cards — dimmed prose paragraph vs. accent-highlighted type signature + example.

**Say:** The same "show, don't describe" rule applies to documentation. "This component handles user filtering with various options" is exactly the kind of sentence a human fills in with tacit knowledge and an agent can't. A prop signature plus a real usage example is something AI can actually act on correctly. One doc, one concern — not the whole feature, just this component.

---

## 20. Real Case — The Doc That Lied

**Say:** Here's the sharper failure mode: a stale doc doesn't just under-inform, it actively misleads. The README said "filters sync to URL query params" — true eight months ago. Since then, a refactor moved filter state into a Zustand store and nobody touched the doc. Told to "follow the README," AI reintroduced URL-based filter state right next to the store — now there are two sources of truth for the same piece of state, and nobody asked for either duplicate. Outdated docs are worse than none — the agent trusts what's written, wrong or not.

---

## 21. Make It Show You What It Saw

**On screen:** Three input nodes (screenshot, Figma frame, live HTML) converging into one lit checkpoint node.

**Say:** Before any of this turns into code, there's one cheap habit that catches most of the damage above: ask AI to restate what it understood — the data shape, the states, a rough layout — before it writes anything. A mismatch here gets caught at the cheapest possible point in the entire loop. It costs you one extra turn.

---

## 22. Real Case — Caught Before Code

**Say:** Concretely: "before implementing, describe back the data shape, the states, and a rough layout." AI restates its understanding and assumes `status` is a required field on every user row. Caught immediately — "status is optional, remember guest accounts" — and fixed before a single line of code existed. One extra turn, one wrong implementation avoided.

---

## 23. Context → Goal → Constraints → References → Output

**On screen:** 5-row vertical stepper, each row a labeled slot with a filled example.

**Say:** This is the team's default prompt shape: Context — what system are we in. Goal — what are we trying to achieve. Constraints — what must or must not happen. References — which existing code should this follow. Output — what do you want back, and in what form. Persona is a weak substitute for real information — a real code reference beats a long "you are a world-class engineer" preamble every time.

---

## 24. Section Divider — 03 Workflow

**Say:** Now let's put this into the actual day-to-day workflow.

---

## 25. Analyze → Plan → Review → Implement

**On screen:** 4-stage relay + a cost curve that spikes if Review is skipped.

**Say:** The default workflow for anything non-trivial: analyze first — tell AI explicitly not to write code yet — then get a plan, then a human reviews that plan against conventions AI can't see, and only then do you implement, one step at a time. Look at the cost curve: catching a wrong direction at the plan stage is flat and cheap. Catching it after implementation is the spike.

---

## 26. Real Case — The Plan Caught It

**Say:** Here's that spike avoided. The plan for a new reports search page proposed component-owned state — a plain useState inside the filter panel. Human review flagged it immediately: these filtered views need to be shareable and bookmarkable, which means the state has to be URL-owned, not component-owned. Revised right there, before a single line of component code existed. That's the plan doing its job.

---

## 27. Generate, Transform, Explain, Discover

**On screen:** Four-quadrant bento — one job per card with its own risk/tip. Clickable, same pattern as slide 14: selecting a card swaps the detail panel below to a fuller explanation and a concrete example.

**Say:** Four jobs AI is good at during implementation, each with a different risk profile.

*(Click through all four live — this is worth slowing down for, especially Transform:)*

1. **Generate** — asking AI to produce new code from a description, when nothing like it exists yet. Safest for small, self-contained units: a single function, a single component, something you can check correctness on in one read.
2. **Transform** — this is the one people mix up with Generate, so slow down here. Transform means the code already exists and already works. You're not asking AI to write something new, you're asking it to change something that's already correct — convert JS to TypeScript, rename a pattern across files, extract a hook, swap a library. The risk that's unique to Transform: because the input already worked, AI can silently "improve" something you never asked it to touch — a null check it assumes is dead code, a default it thinks is safer — and that's now a regression, not a fix. That's why a Transform prompt needs an explicit list of what must NOT change, not just what to change.
3. **Explain** — works best when you name a dimension, state flow, timing, error handling, instead of "what does this do." Naming the dimension forces AI to trace the code along that axis instead of paraphrasing the visible lines.
4. **Discover** — asking AI to find the existing pattern in your own codebase before it invents a new one.

---

## 28. Real Case — Migrating Without Breaking Parity

**Say:** Transform's risk, in the wild: "convert this JS function to TypeScript." AI also "fixed" a null-check it assumed was a leftover bug. That null-check was actually load-bearing for one legacy caller — parity broke silently, because nothing was ever stated as off-limits. State the constraint up front: behavior must stay identical for every existing caller, don't "fix" anything along the way.

---

## 29. Hypotheses Before Fixes

**On screen:** Dual loop comparison — converging spiral (accent) vs. flat dead-end loop (muted).

**Say:** Debugging gets the same discipline. Give full context — expected, actual, error, recent changes — then ask for the top three possible causes with evidence for each, before any fix. That converges. The alternative — guess, change something, rerun, guess again — is flat and eventually dead-ends.

---

## 30. Real Case — The Modal That Wouldn't Close

**Say:** This is the textbook version. Expected: the modal closes after a successful mutation. Actual: the API succeeds and the modal just sits there. Three hypotheses requested: onSuccess never wired to close the modal, a stale closure over an old setter, or the mutation resolving after the component unmounted. Evidence confirmed it was the stale closure. A root-cause note got written before the fix even shipped — what was believed, and where that belief was wrong. That note is the part that survives outside this one session.

---

## 31. Refactor Is Not "Clean This Up"

**On screen:** 3-gate padlock sequence — Analyze, Compare, Refactor Safely.

**Say:** Refactoring gets three gates, not one step. Analyze first — name the duplication, the unnecessary state, the coupling — without touching anything yet. Then propose two approaches with trade-offs and let a human pick. Then implement with explicit constraints: preserve behavior, no public API changes, no new dependencies, minimal diff.

---

## 32. Real Case — Preserving a Bug on Purpose

**Say:** "Preserve behavior" sometimes means preserving a bug you already know about. A dedup pass on a filter-validation function used in three places came back with an "improvement" — trimming whitespace on an empty keyword, technically more correct. But a downstream analytics dashboard depended on the exact old string match. The structural refactor shipped as-is; the "fix" became its own separate ticket. Refactor and bugfix are two different changes — don't let one smuggle in the other.

---

## 33. Ask It to Prove You Wrong

**On screen:** Two speech bubbles, diagonally offset — weak prompt vs. strong prompt.

**Say:** "Is this code good?" gets you generic agreement — useless. "Try to prove this implementation is wrong, find inputs, states, or interactions that could cause incorrect behavior" turns AI into an adversarial reviewer instead of a validator. That single reframe changes what comes back.

---

## 34. Real Case — The Race Condition

**Say:** Same PR, two prompts. "Is this code good?" — "Looks solid, follows conventions." "Try to prove this implementation is wrong" — surfaced that a rapid double-click on Apply Filter fires two overlapping requests, and the older response can resolve last and silently overwrite the newer result. Do not assume the implementation is correct. Try to find reasons it could fail.

---

## 35. Generated Code Is Not Verified Code

**On screen:** 6-tier verification pyramid — Type Check up through Human Review.

**Say:** Whatever comes out of AI still has to climb this pyramid: type check, lint, unit test, E2E, runtime, human review — bottom-up, cheapest checks first. Generated code is not verified code. None of what we've covered so far replaces this.

---

## 36. Section Divider — 04 Frontend & Quality

**Say:** Let's go one layer deeper into what "verified" actually means for frontend work specifically.

---

## 37. AI as a Knowledge Tool, Not Just a Coder

**On screen:** Risk/reward 2x2 — documentation plotted high-reward/low-risk vs. unreviewed code-gen elsewhere.

**Say:** AI is also useful for things that aren't code generation — explaining legacy code, generating docs from actual exports, summarizing architecture with an explicit "known risks" ask. Risk here means the cost if AI gets it wrong, not how often you'd use it — a wrong doc gets caught the moment someone reads it; unreviewed code-gen ships silently.

---

## 38. Real Case — Explaining the Legacy useEffect

**Say:** A forty-line useEffect, three dependencies, no comments, untouched for two years. "What does this do?" gets a vague paraphrase of the visible lines. "What would break if I deleted the useEffect on line 340?" surfaced that it silently re-fetches permissions after a websocket reconnect — the one code path with zero test coverage and nobody left on the team who remembered it. Flag anything that looks like a workaround for a bug elsewhere — legacy code often is one.

---

## 39. The Seven Ways This Goes Wrong

**On screen:** Masonry wall, 7 uneven cards — anti-patterns with their violation quotes. Clickable, like the Requirement pipeline back in slide 14: selecting a card swaps the detail panel below to a real example and a corrective "Instead" line.

**Say:** Seven failure modes to recognize in yourself: vague prompting, premature coding before architecture is decided, context dumping the whole repo just in case, blind acceptance once it compiles, letting AI's suggestion become the architecture decision by default, one thread accumulating five unrelated tasks, and over-automating decisions that need a human name attached to them.

*(Click through a few live — pick the ones the room seems least sure about. One line per card, pause after each click:)*

1. **Vague Prompting** — no context, no criteria, no expected output. AI picks something plausible-looking and calls it done. Instead: state the specific problem, the failing constraint, and what "done" looks like.
2. **Premature Coding** — "build the settings page" lands as working code before anyone agreed on what state it owns. Instead: get a plan reviewed first, even a rough one.
3. **Context Dumping** — the whole repo "just in case" makes the oldest pattern carry the same weight as the current one. Instead: point at the 2-3 files that are the actual reference.
4. **Blind Acceptance** — it compiles and renders, which says nothing about edge cases or whether it matches the requirement. Instead: run it through the verification pyramid first.
5. **AI-Driven Architecture** — a suggestion from inside one prompt quietly becomes the team's state-management decision. Instead: treat it as an input to a human decision, not the decision.
6. **Endless Conversation** — one thread drifts from a bug fix to a refactor to a feature to a copy change. Instead: when the task category changes, `/clear`.
7. **Over-Automation** — AI decides dependencies or security boundaries by default because asking felt like friction. Instead: name who is deciding, and make sure it's a person.

---

## 40. Section Divider — 05 Team Standard

**Say:** Let's bring this together into what the team actually standardizes on.

---

## 41. The One Loop the Whole Team Runs

**On screen:** Tall vertical 11-stage pipeline stepper, two lit "Human Review" gates, Verify branching into 3 parallel lanes, closing on a final "Merge" stage.

**Say:** Requirement through Understand, Decompose, Context, Plan — then the first human review gate, the cheapest place to catch a wrong direction. Implement, then Verify branches into type, test, and runtime checks in parallel before rejoining. AI Review, then the second human review gate — final accountability. Merge means a human is willing to put their name on this.

---

## 42. Ten Rules, One Team

**On screen:** Kinetic marquee — the ten rules scrolling continuously, one accent word each.

**Say:** *(Let it scroll a beat before talking over it.)* Ten rules, condensed: AI doesn't decide architecture alone. Non-trivial tasks get planned first. Use existing patterns first. Minimum sufficient context. Keep changes small and reviewable. Never trust unverified code. No new dependencies without reason. Requirements outrank AI output. Existing code beats a text description. The engineer owns the result.

---

## 43. Section Divider — 06 Playbooks & Beyond

**Say:** Last section — quick-reference recipes, and a look at contributing to the team's shared tooling.

---

## 44. A Recipe for Every Situation

**On screen:** 3x3 uniform grid — 9 situation cards (deliberately uniform, "menu" metaphor).

**Say:** Nine recurring situations, each with the same shape: situation, steps, watch-outs, related reading. New feature, migrating code, debugging, updating existing code, docs-to-code, using a skill, dividing work with sub-agents, managing a session, and a prompt quick-reference. Full versions live in the guide — treat this as the index you come back to.

---

## 45. Beyond Your Own Session

**On screen:** Two stacked horizontal bands — session habits below, shared team tooling above.

**Say:** Everything so far has been about your own session. This layer is about the next engineer's session too — CLAUDE.md for always-on context, skills for on-demand procedures, subagents, hooks. A rule in standing instructions costs context on every turn but is guaranteed to be seen; a skill costs nothing until invoked but has to be discoverable. The contribution loop: notice a pattern repeating across your sessions, draft it as a skill, test it on a real task, get a teammate's feedback, land it under the same review bar as any other shared code.

---

## 46. The Five Things to Remember

**On screen:** Numbered stack reveal, large display type, left-aligned.

**Say:** *(Read these slowly, one at a time — let each land before the next reveals.)* AI doesn't know your codebase unless you show it. Don't ask AI to solve a problem you haven't decomposed. Plan before coding when the task is non-trivial. Give AI patterns, not just instructions. Never trust generated code without verification.

---

## 47. The Actual Goal

**On screen:** Full-bleed centered quote, largest type in the deck.

**Say:** *(Pause before this one. Let the room settle. Read the quote as written, then stop — no summary after it.)* The goal of AI-assisted development is not to make AI write more code. The goal is to help engineers deliver correct, maintainable software faster. That's it. That's the whole session.
