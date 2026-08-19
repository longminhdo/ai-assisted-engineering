# Playbooks Overview

*[← Previous: AI Anti-Patterns](../04-frontend-quality/04-ai-anti-patterns.md) | [Next: Starting a New Feature →](./01-starting-a-new-feature.md)*

## What a playbook is

Modules 0–17 taught concepts: how AI processes context, how to decompose a problem, how to plan before coding, how to debug, refactor, and review. Each one is general — it applies whether you're building a form, fixing a race condition, or migrating an API client. A playbook doesn't teach a new concept. It's a short, situation-specific recipe: you're standing in front of one of a handful of recurring scenarios, and you want a fast answer to "what do I actually do here," with a pointer back to the module that explains *why*, not a re-explanation of it.

Every playbook in this group follows the same shape:

* **Situation** — the scenario you're in, specific enough that you recognize it immediately.
* **Steps** — the ordered moves to make, each one linking to the concept module behind it.
* **Watch-outs** — the specific way this scenario tends to go wrong in practice.
* **Related reading** — where to go for the full reasoning if a step isn't enough on its own.

## Why this group exists

The team asked for something more concrete than "here are the principles" — a runbook they could open mid-task and match against what they're actually doing right now, without re-reading a full module to remember which section applied. Playbooks are that layer: concrete best practices per scenario, sitting on top of the general concepts in Modules 0–17, not replacing them. If a playbook's steps feel thin, that's on purpose — click through to the linked module for the depth.

## The playbooks

| # | Playbook | What you'll walk away with |
|---|----------|------------------------------|
| 1 | [Starting a New Feature](./01-starting-a-new-feature.md) | The contract-first sequence for building something that doesn't exist yet |
| 2 | [Migrating Code](./02-migrating-code.md) | How to port code between patterns/APIs without silently redesigning it |
| 3 | [Debugging a Bug](./03-debugging-a-bug.md) | A fast-reference summary of Module 10's hypothesis-driven loop |
| 4 | [Updating Existing Code](./04-updating-existing-code.md) | How to change working code without it turning into a drive-by refactor |
| 5 | [From Docs to Code](./05-from-docs-to-code.md) | Treating a spec or ticket as a contract, and surfacing its gaps before coding |
| 6 | [Using a Skill](./06-using-a-skill.md) | When to reach for a packaged skill instead of prompting from scratch |
| 7 | [Dividing Work with Sub-Agents](./07-dividing-work-with-subagents.md) | The recipe version of Module 4's Divide & Conquer |
| 8 | [Managing a Session](./08-managing-a-session.md) | The recipe version of Module 2's continue/compact/clear decision |
| 9 | [Prompt Quick Reference](./09-prompt-quick-reference.md) | Real, copy-paste prompts from Anthropic's official prompt library, mapped to each playbook above |

## How to use these

Read a playbook when you're about to start the scenario it describes, not before — they're meant to be opened in the moment, not studied in advance. If you find yourself unsure why a step matters, that's the signal to follow its link back into the concept module rather than guessing at the reasoning.

---

*[← Previous: AI Anti-Patterns](../04-frontend-quality/04-ai-anti-patterns.md) | [Next: Starting a New Feature →](./01-starting-a-new-feature.md)*
