# AI-Assisted Frontend Engineering — Team Guide

This guide is the self-study companion to [`CURRICULUM.md`](../../CURRICULUM.md). The curriculum is the map; this guide is the terrain — each page below expands the curriculum's outline into full explanations, realistic frontend examples, and short exercises you can run against your own codebase.

## Who this is for

Frontend engineers on the team, from **beginner** (new to using AI as more than autocomplete) to **intermediate** (already using Copilot/ChatGPT/Claude day-to-day but inconsistently). You don't need prior AI experience. You do need to already know React, TypeScript, and the basics of state management and data fetching — this guide teaches you how to *direct* AI on those tasks, not how to code from scratch.

## How the guide is organized

The guide is grouped into seven topic areas instead of one long numbered list:

1. **Foundations** — the mindset and mechanics behind every session: what AI is actually doing, when to start/stop a session, how to pick a model and effort level.
2. **Thinking Tools** — habits you apply to every task before you touch code: decomposition, context, visual analysis, how you phrase a request.
3. **Workflow** — the plan-then-build loop: analyze, plan, implement, debug, refactor, review, verify.
4. **Frontend & Quality Depth** — applying that workflow specifically to React/state/data-fetching concerns, testing, documentation, and a mirror of the anti-patterns to watch for in yourself.
5. **Playbooks** — short, situation-specific recipes (starting a feature, migrating code, debugging, updating existing code, going from docs to code, using a skill, dividing work with sub-agents, managing a session). These don't teach new concepts; they tell you what to do in a concrete scenario and link back to Groups 1–4 for the reasoning.
6. **Team Standard** — how the team operationalizes all of the above: the standard end-to-end workflow and the 10 rules the team holds each other to.
7. **Advanced Track** — for engineers ready to go beyond using AI well individually, toward shaping how the *whole team's* sessions work: Claude Code's own project structure (skills, rules, subagents, workflows) and how to contribute to it.

## How to use this guide

- **Read in order the first time.** Groups 1–4 build on each other and are referenced constantly by everything after them. Group 5 (Playbooks) is meant to be read once for orientation, then returned to on-demand when you're actually facing that situation. Groups 6–7 are for once you're comfortable with the rest.
- **Come back as a reference later.** Once you've read a page, its prompt templates and checklists are meant to be reused, not memorized.
- **Do the exercises.** Every concept page ends with a short "Try It Yourself" — do it against a real task in your current sprint, not a toy example. That's where this actually sticks.
- **This is not a rulebook to recite to AI.** It's a set of habits. The goal, per the curriculum, is never "make AI write more code" — it's **"use AI to make better decisions and deliver software faster without compromising quality."**

## Contents

### 1. Foundations

| Page | What you'll walk away with |
|------|------------------------------|
| [AI Mindset](./01-foundations/01-ai-mindset.md) | The Pilot vs. Copilot model — what AI decides, what you decide |
| [How AI Processes Your Request](./01-foundations/02-how-ai-works.md) | The context window, tokens, and why long sessions quietly degrade |
| [Controlling Your Session](./01-foundations/03-controlling-your-session.md) | When to start fresh, compact, or split work across sessions |
| [Choosing Your Model & Effort](./01-foundations/04-choosing-model-and-effort.md) | Matching model tier and reasoning effort to a task's ambiguity and risk, not its size |

### 2. Thinking Tools

| Page | What you'll walk away with |
|------|------------------------------|
| [Problem Decomposition](./02-thinking-tools/01-problem-decomposition.md) | Turning a requirement into a contract before you touch AI — plus Divide & Conquer with sub-agents |
| [Context Engineering](./02-thinking-tools/02-context-engineering.md) | Giving AI the minimum sufficient context, not the whole repo |
| [Visual-First Design Analysis](./02-thinking-tools/03-visual-first-design-analysis.md) | Analyzing the design visually before writing a prompt or a line of code |
| [AI Communication Framework](./02-thinking-tools/04-ai-communication-framework.md) | The Context → Goal → Constraints → References → Output prompt structure |

### 3. Workflow

| Page | What you'll walk away with |
|------|------------------------------|
| [Plan Before Code](./03-workflow/01-plan-before-code.md) | Analyze → Plan → Review → Implement, instead of jumping to code |
| [AI as a Coding Partner](./03-workflow/02-ai-coding-partner.md) | Using AI well for generation, transformation, explanation, and discovery |
| [AI Debugging](./03-workflow/03-ai-debugging.md) | Hypothesis-driven debugging, plus documenting root causes and the brownfield/greenfield order of docs vs. fix |
| [AI Refactoring](./03-workflow/04-ai-refactoring.md) | Analyze → compare approaches → refactor safely |
| [AI Code Review](./03-workflow/05-ai-code-review.md) | Making AI an adversarial reviewer of your own code |
| [Verification](./03-workflow/06-verification.md) | Why "it compiles" is not "it's correct" |

### 4. Frontend & Quality Depth

| Page | What you'll walk away with |
|------|------------------------------|
| [Frontend-Specific AI Review](./04-frontend-quality/01-frontend-specific-review.md) | React, state, data fetching, performance, a11y, responsive checks |
| [AI for Testing](./04-frontend-quality/02-ai-for-testing.md) | Generating tests that verify behavior, not just coverage |
| [AI for Documentation & Knowledge](./04-frontend-quality/03-ai-for-documentation.md) | Using AI to explain, document, and onboard |
| [AI Anti-Patterns](./04-frontend-quality/04-ai-anti-patterns.md) | The failure modes to recognize in yourself and reviews |

### 5. Playbooks

| Page | What you'll walk away with |
|------|------------------------------|
| [Playbooks Overview](./05-playbooks/README.md) | What a playbook is, and how this group differs from Groups 1–4 |
| [Starting a New Feature](./05-playbooks/01-starting-a-new-feature.md) | The recipe for a requirement that starts from nothing |
| [Migrating Code](./05-playbooks/02-migrating-code.md) | Faithful ports vs. accidental refactors, and how to keep the two apart |
| [Debugging a Bug](./05-playbooks/03-debugging-a-bug.md) | The fast-reference version of the debugging loop |
| [Updating Existing Code](./05-playbooks/04-updating-existing-code.md) | Finding the existing pattern before extending it |
| [From Docs to Code](./05-playbooks/05-from-docs-to-code.md) | Treating a spec or ticket as the contract |
| [Using a Skill](./05-playbooks/06-using-a-skill.md) | Recognizing and invoking a packaged team skill instead of prompting from scratch |
| [Dividing Work with Sub-Agents](./05-playbooks/07-dividing-work-with-subagents.md) | The recipe version of Divide & Conquer |
| [Managing a Session](./05-playbooks/08-managing-a-session.md) | The recipe version of session control |
| [Prompt Quick Reference](./05-playbooks/09-prompt-quick-reference.md) | Real, copy-paste prompts from Anthropic's official prompt library, mapped to each playbook |

### 6. Team Standard

| Page | What you'll walk away with |
|------|------------------------------|
| [Standard AI Workflow for the Team](./06-team-standard/01-standard-ai-workflow.md) | The end-to-end loop the team standardizes on |
| [Team AI Coding Standard](./06-team-standard/02-team-coding-standard.md) | The 10 rules the team holds each other to |

### 7. Advanced Track

| Page | What you'll walk away with |
|------|------------------------------|
| [Advanced Track Overview](./07-advanced-track/README.md) | Why this track exists, and who it's for |
| [Anatomy of a Claude Project](./07-advanced-track/01-anatomy-of-a-claude-project.md) | CLAUDE.md, skills, settings, subagents, hooks — what each piece is for |
| [Skills vs. Rules](./07-advanced-track/02-skills-and-rules.md) | When team knowledge belongs in an always-on rule vs. an on-demand skill |
| [Workflows & Contributing a Skill](./07-advanced-track/03-workflows-and-contributing.md) | Multi-agent orchestration, and the practical loop for landing a new skill |

## Official Claude Code documentation

This guide teaches transferable habits; the pages below are the primary source for exact commands, flags, and mechanics in the tool itself. When a guide page and the official docs seem to disagree, the official docs win — Claude Code changes faster than this guide will be updated.

* [Best practices](https://code.claude.com/docs/en/best-practices) — the canonical version of most of Groups 1–4: context management, verify-before-trusting, plan-then-code, effective prompting.
* [Memory (CLAUDE.md & auto memory)](https://code.claude.com/docs/en/memory) — the real mechanics behind the Advanced Track's "standing instructions" and "skills vs. rules" pages.
* [Sessions](https://code.claude.com/docs/en/sessions) — the real commands behind Foundations' Controlling Your Session and the Managing a Session playbook: `/clear`, `/compact`, `/rewind`, `/branch`, naming and resuming.
* [Common workflows](https://code.claude.com/docs/en/common-workflows) — ready-to-adapt prompt recipes for exploring, debugging, refactoring, testing, and PRs, in the same spirit as this guide's Playbooks group.
* [Prompt library](https://code.claude.com/docs/en/prompt-library) — a larger, searchable set of copy-paste prompts by task and role.

## The 5 core principles

> **1. AI doesn't know your codebase unless you show it.**
> **2. Don't ask AI to solve a problem you haven't decomposed.**
> **3. Plan before coding when the task is non-trivial.**
> **4. Give AI patterns, not just instructions.**
> **5. Never trust generated code without verification.**

Everything in this guide is one of these five ideas applied to a specific situation.
