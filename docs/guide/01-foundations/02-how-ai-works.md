# Module 1 — How AI Processes Your Request

*[← Previous: AI Mindset](./01-ai-mindset.md) | [Next: Controlling Your Session →](./03-controlling-your-session.md)*

## Why this matters

Picture a session that starts clean: you paste in a component, ask for a focused fix, and get a tight, correct diff back. Ninety minutes later, in the *same* session, you've fixed two more bugs, pasted in four other files "just in case," and argued with the model about a naming convention. Now you ask for one more small change, and the response ignores a constraint you stated an hour ago, reintroduces a bug you already fixed, and quotes a variable name from a file you pasted early on that's no longer even relevant. Nothing about the model changed between the first request and the last one. What changed is everything sitting in front of it — the context window filled with your entire session history, and the signal you actually need is now buried under everything else that happened. Understanding *why* that happens — not just that it happens — is what lets you prevent it instead of discovering it the hard way in hour two.

## Objective

Build an accurate mental model of what "context" physically is inside an AI session — the context window, tokens, and how the model actually reads what's in front of it — so that Module 2 (session control) and Module 3 (model/effort selection) are decisions you can reason about, not folklore you follow because someone said so.

---

## 1.1 The Context Window: Everything AI Knows, Right Now

An AI model has no persistent memory of you, your codebase, or your last session. Each time it generates a response, the *only* thing it has access to is whatever text is currently sitting inside its **context window** — a fixed-size buffer that holds the entire visible conversation: your system prompt/instructions, every file you've pasted or it has read, every message either of you have sent, and every tool result it has seen. When that buffer is full, older content either falls out entirely or gets compressed (Module 2 covers how). There is no separate long-term memory it quietly consults — if something isn't in the window, as far as this response is concerned, it doesn't exist.

This is the single most load-bearing fact in this entire guide. Every technique in Modules 4 through 9 — writing contracts before code, giving minimum sufficient context, structuring prompts, planning before implementing — is a strategy for controlling *what's in the window* at the moment you need a good answer. There is no other lever. You cannot make the model "try harder to remember" or "pay closer attention" in the abstract; you can only change what's actually inside the window and where it sits.

**Worked example.** You open a session and paste in `CheckoutForm.tsx`, `useCheckoutMutation.ts`, and the `PromoCodeValidationResult` type, then ask for a new field. The context window right now contains exactly those three things, your prompt, and nothing else about your codebase — not the sibling `OrderSummary.tsx` two folders over, not the team's ESLint config, not the PR description from last week. If the correct implementation depends on something not in that window, the model has no way to know it exists unless you put it there. This is why Module 5 (Context Engineering) is framed as "climb from the task outward" rather than "describe your whole codebase" — you are literally deciding the contents of the only thing the model can see.

---

## 1.2 Tokens: What Actually Gets "Read"

The context window isn't measured in characters, words, or files — it's measured in **tokens**, a model's internal unit of text, roughly ¾ of a word in English (shorter for common words, longer for unusual identifiers, and often *more* tokens per character for dense syntax, deeply nested JSX, or minified code). A 2,000-line component isn't "one file" from the model's perspective — it's tens of thousands of tokens competing for space with everything else you've included. This matters practically in three ways.

First, **budget is finite and shared.** Every model has a maximum context window — commonly ranging from roughly 200K to 1M+ tokens depending on the model — and everything you paste, every file the model reads via a tool, every prior turn of the conversation, and its own responses all draw from that same pool. Pasting one enormous file doesn't just cost you "some space" — it can crowd out room for the model's own reasoning and for later turns in the same session.

Second, **verbose or repetitive content is expensive for no benefit.** Pasting the same file twice because you weren't sure it "took," including generated build output, or dumping an entire `node_modules`-adjacent config doesn't make the model more thorough — it just spends tokens on content that adds no new signal, which is a milder version of the context pollution problem from Module 5.

Third, **code is not free just because it's structured.** A deeply nested component tree, heavily typed generics, or a file with a huge autogenerated section (GraphQL codegen output, a giant enum) can burn far more tokens than the same number of lines of plain prose. If you're not sure why a session feels "full" faster than expected, generated or boilerplate-heavy files are frequently why.

**Worked example.** Say you're debugging a hook and, to be thorough, you paste in the entire `api/generated/` directory (a 4,000-line auto-generated API client) alongside the 40-line hook that actually has the bug. The hook is a rounding error in token terms; the generated client is the overwhelming majority of what you just spent. You haven't given the model more useful signal — you've spent most of your budget on content it likely won't reference at all, and left less room for the rest of the session.

---

## 1.3 Attention Isn't Uniform — Position and Recency Matter

Not every token in the context window gets equal weight when the model generates its next response. Two effects matter enough to change how you work:

**Recency bias.** Content nearer the end of the conversation — your most recent messages, the most recently pasted file, the last tool result — tends to have outsized influence on the immediate response compared to something said much earlier in a long session. This is *not* a bug to route around with tricks; it's a direct consequence of what "generate the next token given everything before it" means. The practical implication: a constraint or decision you stated at the very start of a long session ("never touch the API layer," "state must stay in the URL") is more likely to be silently dropped the further the conversation drifts from that point, especially once several unrelated fixes have piled on top of it.

**Mid-context dilution.** Information placed in the middle of a very long context — neither in the system-level instructions nor in the recent turns — is the most likely to be underweighted relative to its actual importance. A type definition you pasted forty messages ago, still technically "in the window," competes for attention with everything that came after it and everything the model is about to generate. It hasn't been forgotten in the sense of Module 1.1 — it's still physically present — but its *influence* on the current response has quietly diminished.

Neither effect means the model is "bad" — it means position in the window is not neutral, and long-running, meandering sessions are exactly the conditions under which important-but-old context loses its grip. This is the direct mechanical reason behind two rules you'll see recur elsewhere in this guide: Module 13's "Endless Conversation" anti-pattern, and Module 2's practice of restating load-bearing constraints instead of assuming they're still "in mind."

**Worked example.** Forty minutes into a session, you stated once: "this project has no `any` — strict TypeScript only." You then worked through six unrelated tasks. On the seventh, the model hands back a diff with `catch (e: any)`. This isn't the model "forgetting" in a human sense — the instruction is still technically present in the window — but its influence has been diluted by everything generated since, and it was never in the highest-attention zones (very start / very recent) to begin with. The fix isn't a cleverer phrasing of the original rule — it's restating the constraint at the point where it matters, which Module 2 covers as a concrete habit.

---

## 1.4 Why Long Sessions Degrade (Context Rot)

Put 1.1–1.3 together and you get a phenomenon worth naming explicitly: **context rot** — the tendency for output quality to decline over the course of a single long-running session, even though the model itself hasn't changed at all. It isn't one failure mode; it's the compounding of several:

* The window fills with content that was relevant to task 1 but is now noise for task 7 (Module 5's context pollution, but accumulated over time instead of dumped all at once).
* Load-bearing constraints stated early lose influence to recency bias (1.3).
* Contradictions accumulate — you corrected an approach in message 12, but a similar-looking instruction from message 3 is still technically in the window and occasionally still gets pattern-matched against.
* Once the window's practical limit is approached, older content may be summarized or dropped to make room for new content (Module 2 covers this directly) — and a summary is lossy by definition; it keeps the gist and loses exact details like a specific line number or a rejected approach's reasoning.

None of this means a session has a hard countdown timer. A well-scoped session that stays on one task can run long and stay sharp. What actually drives context rot is *accumulated, undifferentiated history* — many unrelated tasks, corrections, and pasted files stacking up without ever being cleared or refocused. This is why Module 2 exists as its own module rather than a footnote: once you understand *why* long sessions degrade, "should I start a new session for this" stops being a vague instinct and becomes a decision you can reason about from first principles.

**Worked example.** Compare two engineers debugging the same class of bug over an afternoon. Engineer A keeps one session open all day: fixes the checkout spinner, then a filter bug, then a styling regression, then comes back to checkout because QA found a related issue — all in the same thread, with a dozen pasted files accumulated along the way. Engineer B starts a fresh, tightly-scoped session per bug, pasting in only what that bug needs. By the fourth task, Engineer A's session is fighting context rot: old files compete with new ones, an earlier "don't touch the API layer" instruction has faded, and the model occasionally reintroduces an approach that was already rejected two bugs ago. Engineer B's fourth session performs identically to their first, because each one started from a clean, minimal window. Same model, same tasks, different session discipline.

---

## Key Takeaways

* The context window is the *only* thing the model can see for a given response — no file, fact, or prior decision matters unless it's currently inside that window. Every technique later in this guide is a way of controlling what's in it.
* Context is measured in tokens, not files or lines. Verbose, repetitive, or generated content spends budget without adding signal — know when a "quick paste for safety" is actually a meaningful tax on the rest of the session.
* Position in the window isn't neutral: recent content has outsized influence (recency bias), and content stuck in the middle of a long session loses influence even though it's technically still present.
* Context rot — quality decline over a long session — isn't random. It's the compounding of context pollution, faded early constraints, and accumulated contradictions. A long session on *one* focused task doesn't automatically rot; an unfocused one reliably does.
* This module is the mechanics. Module 2 turns it into habits (when to clear, compact, or split a session) and Module 3 turns it into a resourcing decision (which model and how much reasoning effort a given task's context and complexity actually justify).

## Try It Yourself

1. Open your longest-running recent AI session (or start tracking a new one). At the one-hour mark, ask the model to restate every hard constraint you gave it in the first ten minutes, without looking back yourself first. Compare its answer to what you actually said — any drop or drift is context rot from Section 1.3–1.4, made visible.
2. Pick a task you're about to hand off. Before you paste anything, estimate: how many files, and roughly how large, does this actually need (tie this back to Module 5's minimum sufficient context)? Then check what you *actually* pasted by habit. If there's a gap, that gap is exactly the token budget from Section 1.2 you were about to spend on signal the model didn't need.

---

*[← Previous: AI Mindset](./01-ai-mindset.md) | [Next: Controlling Your Session →](./03-controlling-your-session.md)*
