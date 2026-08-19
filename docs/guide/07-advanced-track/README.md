# Advanced Track Overview

*[← Previous: Team AI Coding Standard](../06-team-standard/02-team-coding-standard.md) | [Next: Anatomy of a Claude Project →](./01-anatomy-of-a-claude-project.md)*

## Why this track exists

Modules 0–19, plus the Playbooks group, all teach the same underlying skill from different angles: how to run *your own* AI session well — how to scope a task, supply the right context, plan before implementing, verify before merging. Every technique in that material makes one engineer's next hour with AI better. None of it, on its own, makes anyone else's session better. Two engineers on the same team, both individually excellent per Modules 0–19, can still each independently re-explain the same testing convention to AI in a dozen separate sessions a month, each hand-write the same "how we do a dependency upgrade" prompt from scratch, each get burned by the same missing constraint — because nothing about being good at directing your own session automatically shares what you've learned with the next person's session.

That gap is what this track closes. It's for engineers who've internalized Modules 0–19 and are ready to stop optimizing only their own sessions and start shaping the structure that *every* session on the team runs on — the standing instructions, the packaged procedures, the guardrails, and the automation Claude Code itself supports. This is a later, narrower stage on purpose: shaping shared tooling well requires already knowing, from firsthand experience, which habits are worth encoding and which are just personal preference — and that judgment only comes from having done the work the foundation track teaches.

Concretely, this connects back to one page you've already read: the Playbooks group's [Using a Skill](../05-playbooks/06-using-a-skill.md) page taught you the **consumer** side of this — recognizing when a packaged skill already exists for what you're doing and reaching for it instead of prompting from scratch. This track teaches the **author** side — how that skill (or a `CLAUDE.md` rule, or a subagent, or a hook) gets written, scoped, and landed for the team in the first place. If Using a Skill was about noticing the tool on the shelf, this track is about learning to build the shelf.

## What's in this track

| Page | What you'll walk away with |
|---|---|
| [Anatomy of a Claude Project](./01-anatomy-of-a-claude-project.md) | The moving parts — `CLAUDE.md`, auto memory, path-scoped rules, skills, settings/permissions, subagents, hooks — and a heuristic for which one a given piece of team knowledge belongs in |
| [Skills vs. Rules](./02-skills-and-rules.md) | The concrete trade-off between an always-on `CLAUDE.md` rule and an on-demand skill, and what makes a skill discoverable versus dead weight |
| [Workflows & Contributing a Skill](./03-workflows-and-contributing.md) | When multi-agent orchestration actually earns its complexity, and the practical loop for turning something you keep doing by hand into a skill the whole team uses |

---

*[← Previous: Team AI Coding Standard](../06-team-standard/02-team-coding-standard.md) | [Next: Anatomy of a Claude Project →](./01-anatomy-of-a-claude-project.md)*
