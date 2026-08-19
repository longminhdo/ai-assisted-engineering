# Advanced Track — Anatomy of a Claude Project

*[← Previous: Advanced Track Overview](./README.md) | [Next: Skills vs. Rules →](./02-skills-and-rules.md)*

## Why this matters

Picture a five-person frontend team, all individually strong at directing AI per Modules 0–19. Engineer A opens a session, and the first thing they type is a paragraph explaining that the team never uses `any`, always reaches for the existing `<Button>` component instead of a raw `<button>`, and writes changelog entries in a specific format under a `## Unreleased` heading. Engineer B, working on a different ticket that same afternoon, types almost the same paragraph into their own session, phrased slightly differently, and forgets the changelog part entirely. Engineer C doesn't type it at all — they didn't know the convention existed until it came up in review. Nothing about any of this is a Module 0–19 failure. Each engineer is scoping their own session reasonably well. The problem is that "reasonably well" is being reinvented from scratch, independently, every single session, by every single person — and reinvention that depends on memory is reinvention that will eventually get skipped.

Now picture the same team a few weeks later, after someone took the time to write the standing conventions into a file Claude Code reads automatically, package the changelog procedure as a named, invokable skill, and configure which commands run without a permission prompt. Engineer A, B, and C now start every session with the same baseline already loaded or one command away — not because they each remembered to restate it, but because the structure did it for them, once, in a place all three sessions draw from. That structure — not any single clever prompt — is what this track teaches you to build. This page is the map of what that structure is actually made of.

## Objective

Learn the mechanisms Claude Code gives a project for carrying knowledge and control *between* sessions instead of living only inside one — `CLAUDE.md`, auto memory, path-scoped rules, skills, settings/permissions, subagents, and hooks — what each one is for, what it costs, and a working heuristic for deciding which one a given piece of team knowledge belongs in.

---

## 1.1 CLAUDE.md — Always-On Instructions You Write

`CLAUDE.md` is a file Claude Code reads automatically at the start of every session in a project, without anyone asking it to. Whatever's in it is loaded into context before the first prompt is even typed — it's the closest thing this tooling has to a standing briefing every session gets by default. It can live at the project root (shared with the team via version control), in your home directory (personal, applies to every project you work in), or as a gitignored `CLAUDE.local.md` for project-specific preferences you don't want to commit.

**Realistic example:**

```markdown
# CLAUDE.md

This is a Next.js 14 app using the App Router.

- Server state lives in React Query; client/UI state lives in Zustand. Do not
  introduce a third state mechanism without discussion.
- Never use `any`. If a type is genuinely unknown, use `unknown` and narrow it.
- Always use the design system's `<Button>` (`@ds/button`) instead of a raw
  `<button>` element.
- Never commit directly to `main`. All changes go through a PR.
```

The defining trade-off is that this content is **always paid for**: it consumes context-window budget on every single turn of every single session in the project, whether or not the current task has anything to do with state management, `any`, buttons, or git workflow (this is the same context-budget lesson Module 1 covers for anything sitting in the window — a `CLAUDE.md` line is no exception just because it loaded automatically instead of being pasted in). That cost is worth paying for things that are relevant almost all the time. It's wasted, turn after turn, for things that only matter occasionally. The [official memory guide](https://code.claude.com/docs/en/memory#write-effective-instructions) puts a concrete number on it — keep a `CLAUDE.md` under roughly 200 lines, and for every line ask "would removing this cause Claude to make mistakes?" If not, cut it. Module 2 (Skills vs. Rules) goes deeper on exactly where that line sits.

## 1.1a Auto Memory — Notes Claude Writes Itself

`CLAUDE.md` is only half of how Claude Code carries knowledge across sessions, and it's the half *you* write. The other half, **auto memory**, is knowledge Claude writes for itself, based on your corrections and preferences as you work — a build command it had to be told twice, a debugging insight from a session last week, a preference you stated once in conversation instead of adding to `CLAUDE.md` directly. It lives in a per-project memory directory, with a `MEMORY.md` index (loaded into every session, kept short by design) and separate topic files Claude reads on demand rather than up front.

The practical difference from a `CLAUDE.md` rule: you don't have to remember to write it down. If you tell Claude "always use pnpm, not npm, remember that," it saves the note itself instead of you editing a file. The trade-off is the inverse of a rule's reliability — auto memory is Claude's own judgment about what's worth keeping, so it's worth occasionally auditing (Claude Code exposes a command to browse what's been saved) the same way you'd review any other standing instruction. See the [official memory guide](https://code.claude.com/docs/en/memory#auto-memory) for the full mechanics.

## 1.1b Path-Scoped Rules — Always-On, But Only For Matching Files

Between "loaded every turn no matter what" (`CLAUDE.md`) and "loaded only when explicitly invoked" (a skill, below) sits a third option: rules that live in a project's rules directory, one topic per file, optionally scoped to a set of file paths. A rule with no path scoping behaves like an extra `CLAUDE.md` file. A rule scoped to, say, every file under an API directory only enters context when Claude actually reads or edits a file matching that pattern — so a constraint that's near-universal *for one part of the codebase* doesn't have to cost every session in every other part of it.

**Realistic example** — a rule that only matters when Claude is touching API handlers:

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Handler Conventions
- Every handler validates its input with the project's schema library before
  touching the database.
- Errors return the standard `{ error: { code, message } }` shape.
```

This is the mechanism for team knowledge that's genuinely a hard rule, but only within a scope narrower than "the whole repo" — exactly the case that doesn't fit cleanly into either "put it in `CLAUDE.md`" or "make it a skill."

## 1.2 Skills — Packaged, On-Demand Procedures

A skill is a named, packaged instruction set — typically a `SKILL.md` file with frontmatter declaring a name and a description — that sits dormant until something invokes it: either the engineer names it directly, or Claude Code recognizes that the current task matches the skill's description and pulls it in on its own. Unlike `CLAUDE.md`, a skill costs nothing in every other session; it only enters the context window for the session that actually needs it.

**Realistic example** — a skill for a specific, occasionally-needed procedure:

```markdown
---
name: generate-changelog-entry
description: Use when the user asks to add a changelog entry for a merged PR
  or an upcoming release. Reads the diff and commit messages, then adds an
  entry under the "## Unreleased" heading in CHANGELOG.md, following the
  existing Added/Changed/Fixed grouping.
---

1. Read the diff for the PR or range of commits being described.
2. Classify the change as Added, Changed, or Fixed.
3. Write a single-line, user-facing entry — not a restatement of the commit
   message — under the matching heading in CHANGELOG.md.
4. If no "## Unreleased" section exists yet, create one above the latest
   version heading.
```

Skills trade the "always loaded" property for a "must be discovered" property: this one only ever fires if the description is specific enough to match the moment someone actually asks for a changelog entry, or someone invokes it directly by name (`/generate-changelog-entry`). A vague description ("helps with release stuff") is a skill that exists but never gets found — functionally identical to not having written it. Module 2 covers what makes a description discoverable in detail.

## 1.3 Settings & Permissions — What Runs Without Asking

Project-level settings configure which tools and commands Claude Code is allowed to run without stopping to ask, and which ones always require explicit approval first. This isn't about *what* Claude Code knows or does — it's about the friction (or lack of it) around actually executing an action.

**Realistic example:** a project might auto-allow read-only and low-risk commands — `git status`, `git diff`, `npm test`, `npm run lint` — so routine verification doesn't interrupt the flow of a session, while requiring explicit approval for anything that mutates shared state or is hard to undo: `git push`, `git commit` on a protected branch, running a database migration, or a deploy script. The heuristic underneath most permission configs is reversibility: cheap-to-undo, read-only, or already-sandboxed actions are good candidates for auto-allow; anything that touches something outside the local working copy generally isn't.

## 1.4 Subagents — Scoped Delegates

A subagent is a separate agent configuration — its own system prompt, its own restricted set of available tools — that the main session can delegate a specific kind of task to. The point isn't just parallelism; it's **containment**. A subagent's exploration, false starts, and intermediate output happen in its own context, and only the result comes back to the main session — keeping the main thread's window free of work that isn't the main thread's concern (the same context-pollution problem Module 5 covers, solved structurally instead of by discipline).

**Realistic example:** a `test-writer` subagent scoped to `Read`, `Write`, and a test-runner `Bash` command only — no access to deployment tooling, no access to unrelated parts of the repo. The main session delegates "write tests for this hook" to it; the subagent reads the hook, writes the test file, runs the tests, and reports back pass/fail — without the main session's window absorbing every intermediate test run and false start along the way.

## 1.5 Hooks — Automation That Doesn't Depend on the Model Remembering

A hook is a shell command that fires automatically on a defined event — after a tool call, before a commit, at session end — independent of whether the model "remembers" to do it. This is the mechanism for anything where the cost of a missed step is too high to leave to instruction-following alone.

**Realistic example:** a `pre-commit`-style hook that runs the type-checker and linter automatically every time Claude Code attempts a `git commit`, blocking the commit outright if either fails — and a second hook that rejects any commit whose target branch is `main`. Both enforce something Rule 1/Rule 2-style `CLAUDE.md` instructions could *ask* for, but a hook enforces it regardless of whether the instruction was loaded, read carefully, or quietly deprioritized by recency bias (Module 1.3) forty turns into a long session.

## 1.6 Which Mechanism Does This Belong In?

These mechanisms solve different problems, and most "where should this live" confusion comes from treating them as interchangeable. A simple heuristic, in the order to check it:

1. **Does this absolutely have to happen, no exceptions, regardless of whether the model reads or prioritizes an instruction?** → **Hook.** Enforcement that can't be allowed to depend on the model remembering.
2. **Is this about which actions require a human to approve them before running?** → **Settings/permissions.** Not knowledge — friction control.
3. **Does this apply to nearly every task in the repo, regardless of what the task is?** → **CLAUDE.md.** Pay the per-turn cost because the relevance is near-constant.
4. **Does this only apply when Claude is touching a specific subset of files (a directory, an extension), but is a hard rule whenever it does?** → **Path-scoped rule.** Same reliability as `CLAUDE.md`, scoped cost.
5. **Is this a specific, occasionally-needed procedure that a well-written description can reliably recognize?** → **Skill.** Zero cost until the moment it's actually needed.
6. **Does this need its own contained exploration, or its own restricted tool access, separate from the main session's concerns?** → **Subagent.** Delegate it and let only the result return.
7. **Is this a pattern Claude noticed on its own — a build quirk, a debugging insight — rather than a rule anyone decided to write down?** → **Auto memory.** Let it accumulate, and audit it occasionally rather than authoring it yourself.

A single piece of team knowledge often touches more than one of these. "Never commit to `main`" is both a `CLAUDE.md` rule (so the model doesn't propose it) *and* a hook (so it's blocked even if the model does anyway) — the rule sets the expectation, the hook makes it actually true.

---

## Key Takeaways

* Individual session skill (Modules 0–19) makes *your* sessions better; it does nothing for the next engineer's session unless something is actually shared between them. `CLAUDE.md`, auto memory, path-scoped rules, skills, settings, subagents, and hooks are the mechanisms Claude Code gives a project for that sharing.
* `CLAUDE.md` is always loaded, every session, every turn — good for near-universal constraints, wasteful for occasional ones. Auto memory is the same "always loaded" deal, but Claude writes it, not you.
* A path-scoped rule gets `CLAUDE.md`-level reliability for a subset of files, without the whole-repo always-on cost.
* A skill costs nothing until invoked, but only fires if its description actually matches the task (or someone names it directly) — a badly-described skill is functionally unwritten.
* Settings/permissions control friction (what runs without approval), not knowledge.
* Subagents contain a task's exploration in its own scoped context, keeping the main session's window clean.
* Hooks enforce something regardless of whether the model remembers to — use them when the cost of a missed step is too high to leave to instruction-following.
* The decision heuristic runs in order: enforcement-critical → hook; approval friction → settings; near-universal → CLAUDE.md; universal-but-scoped-to-some-files → path-scoped rule; occasional and describable → skill; needs containment → subagent; self-discovered pattern → auto memory. The same piece of knowledge can legitimately need more than one.

## Try It Yourself

1. Pick one convention your team currently only enforces by word of mouth or by catching it in review (a naming convention, a "never do X" rule, a required step before merging). Run it through the 1.6 heuristic and decide which mechanism it actually belongs in — then check whether your project currently has that mechanism in place for it at all.
2. Look at your own `CLAUDE.md` (or your team's, if one exists). For each line in it, ask: does this apply to nearly every task, or only some? Flag any line that's actually a skill's worth of occasional, specific procedure that's been paying the always-on cost unnecessarily — Module 2 picks this exact question back up in depth.

---

*[← Previous: Advanced Track Overview](./README.md) | [Next: Skills vs. Rules →](./02-skills-and-rules.md)*
