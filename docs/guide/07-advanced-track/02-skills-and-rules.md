# Advanced Track — Skills vs. Rules

*[← Previous: Anatomy of a Claude Project](./01-anatomy-of-a-claude-project.md) | [Next: Workflows & Contributing a Skill →](./03-workflows-and-contributing.md)*

## Why this matters

A team lead spends an afternoon writing down everything they wish every engineer's AI session already knew: never use `any`, always use the design system's components, here's how we write a changelog entry, here's the exact sequence for a dependency upgrade, here's our commit message format, here's how we name test files. All of it is true, all of it is useful, and all of it goes into `CLAUDE.md` because that's the file that's "always there." Three weeks later, engineers start quietly complaining that every session feels slower to get going and somehow less sharp on straightforward tasks — even ones that have nothing to do with dependency upgrades or changelogs. Nobody removed anything from `CLAUDE.md`; the problem is what got put there in the first place. Module 1 introduced the two mechanisms — `CLAUDE.md` and skills — as different tools. This page is about the one decision that determines whether a team's shared instructions stay sharp or slowly become the exact kind of context pollution Module 5 warned about, just moved from a single prompt into a file that's loaded automatically instead.

## Objective

Learn the concrete trade-off between an always-on `CLAUDE.md` rule and an on-demand skill, get a decision rule you can apply to any piece of team knowledge in under a minute, and learn what separates a skill that actually gets found from one that quietly never fires.

---

## 2.1 The Trade-off, Stated Plainly

`CLAUDE.md` content is loaded into every session's context window, every turn, whether or not it's relevant to the current task. That's not a minor implementation detail — it's the same context-budget mechanic Module 1 (How AI Processes Your Request) and Module 5 (Context Engineering) both build entire frameworks around: tokens spent on irrelevant content are tokens not available for what the task actually needs, and past a point they don't just sit there neutrally, they compete with the signal that matters. A `CLAUDE.md` that's accumulated forty lines of rules — five of which are relevant to any given task, thirty-five of which aren't — is paying full context cost on every single turn for content that's dead weight most of the time.

A skill pays none of that cost until it's invoked. It sits outside the context window entirely — a name and a description Claude Code can match against the current task — and only the moment it's actually needed does its content enter the window. The cost moves from "constant, on every turn" to "zero, until relevant" — but that shift comes with a new failure mode that `CLAUDE.md` doesn't have: a skill that never gets matched to the task it was written for might as well not exist. `CLAUDE.md` fails by being expensive; a skill fails by being invisible.

## 2.2 The Decision Rule

The question that actually separates these two isn't "is this important" — everything a team bothers writing down is important to someone. The question is: **how often does this apply?**

> **Standing constraints that apply to almost every task in the repo belong in `CLAUDE.md`. A specific, occasionally-needed procedure belongs in a skill.**

"Never use `any`" applies to essentially every piece of TypeScript anyone will ever ask AI to write in this repo — there's no task category where it's irrelevant. "Always use the design system's `<Button>` instead of a raw `<button>`" is the same: it's relevant any time UI is being touched, which in a frontend codebase is most of the time. Both are correctly always-on.

"Here's how we write a changelog entry" applies only on the (real, but occasional) occasions someone is actually writing a changelog entry. "Here's the sequence for a dependency upgrade" applies only during a dependency upgrade. Neither of these needs to be sitting in every session's context while someone is fixing an unrelated bug in a form component — and every session where it does sit there unused is a session that paid full price for zero relevance.

A useful gut-check: if you can picture a normal week where an engineer runs ten different AI sessions and this piece of knowledge would only be relevant to one or two of them, it's a skill, not a rule — even if it's a rule you feel strongly about.

## 2.3 What Makes a Skill Discoverable

A skill's description is the only thing standing between "this fires exactly when it should" and "this was written once and never used again." Two properties separate a good description from a bad one.

**Specific trigger conditions, not a vague category.** A description should name the situation concretely enough that both a human deciding whether to invoke it by name, and Claude Code deciding whether to auto-suggest it, can match it against a real request.

```markdown
# Bad — vague, describes a topic instead of a trigger
description: Helps with release-related stuff.

# Good — names the concrete situation and what the skill does about it
description: Use when the user asks to add a changelog entry for a merged PR
  or an upcoming release. Adds an entry under the "## Unreleased" heading in
  CHANGELOG.md, classified as Added/Changed/Fixed, based on the diff.
```

The bad version could theoretically match a dozen unrelated requests ("help me plan a release," "what's our versioning scheme," "why did this release break") and will reliably match none of them well, because it doesn't actually say what situation it's for or what it does. The good version matches one thing, precisely, and someone skimming a list of skills can tell instantly whether it's the one they want.

**Named for the action, not the domain.** `changelog-entry` or `generate-changelog-entry` tells you what invoking it does. `release-helper` or `versioning-utils` tells you a topic area and leaves you to guess what it actually does when invoked — the skill equivalent of Module 7's caution against vague prompting: a name and description are, in effect, a prompt that has to work without you there to clarify it.

A skill can be technically well-written — correct steps, sound procedure — and still be dead on arrival if its description doesn't say, in concrete terms, what request should trigger it. Writing the procedure is the easy half; writing the description that gets it found is the half that actually determines whether it ever runs.

## 2.4 Worked Example: Same Knowledge, Two Encodings

Take one real piece of team knowledge: "when we bump a dependency's major version, check the changelog for breaking changes, run the full test suite, update any deprecated API calls the linter flags, and note the upgrade in the changelog." Here's the same knowledge encoded badly, then well.

**Encoded badly — as an always-on `CLAUDE.md` rule:**

```markdown
- When upgrading a dependency to a new major version, check its changelog
  for breaking changes, run the full test suite, fix any deprecated API
  usage flagged by the linter, and add a changelog entry noting the upgrade.
```

This is correct advice that now costs context budget on every turn of every session — including the ninety percent of sessions that have nothing to do with dependencies at all. It also does nothing to actually help *during* an upgrade beyond a one-line reminder; there's no room in a `CLAUDE.md` line for the actual step-by-step procedure without bloating it further.

**Encoded well — as a skill:**

```markdown
---
name: upgrade-dependency
description: Use when the user asks to upgrade a package to a new major
  version. Walks through checking the changelog for breaking changes,
  running the test suite, fixing deprecated API usage, and recording the
  upgrade in CHANGELOG.md.
---

1. Read the dependency's changelog between the current and target version;
   list every breaking change relevant to how this repo uses the package.
2. Bump the version and run the full test suite.
3. For each test failure or lint warning tied to a deprecated API, fix the
   call site to the new API — do not suppress the warning.
4. Add a changelog entry under "## Unreleased" noting the upgrade and any
   user-facing impact.
```

Every session that isn't a dependency upgrade now pays nothing for this. The one session that is a dependency upgrade gets the full procedure, invoked exactly when it's relevant, instead of a one-line reminder competing for space with everything else in `CLAUDE.md`.

## 2.5 A Third Option: When the Rule Is Real But Only For Some Files

The binary framing above — `CLAUDE.md` or skill — covers most cases, but there's a real middle ground worth naming: a constraint that's a hard, always-apply rule, but only for a subset of the codebase. "Every API handler validates its input before touching the database" isn't occasional the way a changelog procedure is — it should fire every single time, with no discovery step required — but it's also not relevant to a session that never touches `src/api/`. Forcing this into `CLAUDE.md` pays its cost on every unrelated session; forcing it into a skill risks it not firing on a request that doesn't happen to phrase itself as "let's talk about API validation."

Anatomy of a Claude Project (§1.1b) covers the mechanism for exactly this case: a path-scoped rule, kept in its own file and limited to matching file patterns, so it has `CLAUDE.md`'s reliability (no discovery required) with a skill's scoped cost (silent for sessions that never touch the matching files). Reach for this when the honest answer to "how often does this apply?" is "always — but only within one part of the repo," not "always, everywhere" or "occasionally, anywhere."

## 2.6 Getting the Balance Wrong in Both Directions

Overloading `CLAUDE.md` is the more common failure because it feels safer — "at least it's always there" — but a bloated `CLAUDE.md` doesn't just waste budget, it can actively degrade unrelated tasks the same way an over-stuffed prompt does in Module 5.3's context pollution: more competing signal, more chances the model weights an irrelevant rule over the one that actually matters to the task at hand.

The opposite mistake — pushing something into a skill that actually needed to be always-on — is quieter but just as real. A constraint like "never use `any`" encoded only as a skill named `typescript-strictness-check` will sit unused on the vast majority of tasks that touch TypeScript but don't happen to phrase the request in a way that matches the skill's description, because nothing about "add a field to this form" obviously triggers a skill about type strictness. Near-universal constraints need to be in front of the model by default, not conditional on a description matching — that's precisely what `CLAUDE.md`'s always-on cost is buying.

---

## Key Takeaways

* `CLAUDE.md` content is loaded every turn, every session, whether relevant or not — good for near-universal constraints, actively wasteful (and potentially degrading, per Module 5's context pollution) for occasional ones.
* A skill costs nothing until invoked, but only fires if it's discoverable — a skill that's never matched to the task it was written for is functionally the same as not having written it.
* The decision rule: **applies to almost every task → `CLAUDE.md`. Specific and occasional → skill.** The gut-check is picturing a normal week of ten sessions and asking how many this would actually be relevant to.
* A good skill description names a concrete trigger situation and states what the skill does about it. A vague description ("helps with X stuff") is close to unusable even if the procedure inside is well-written.
* Both directions of miscalibration are real: cramming an occasional procedure into `CLAUDE.md` bloats every session; pushing a near-universal constraint into a skill means it silently doesn't apply most of the time it should.

## Try It Yourself

1. Pull up your team's actual `CLAUDE.md` (or draft one, if it doesn't exist yet). For each line, apply the ten-sessions gut-check from 2.2. Move anything that only clears three or fewer out of ten into a skill, and write a description for it using the "concrete trigger + what it does" pattern from 2.3.
2. Find one skill your team already has (or one you're about to write). Read only its description, out of context, and ask: would you, or would Claude Code, actually reach for this on the exact task it's meant for? If the answer isn't an obvious yes, rewrite the description before touching the procedure inside it.

---

*[← Previous: Anatomy of a Claude Project](./01-anatomy-of-a-claude-project.md) | [Next: Workflows & Contributing a Skill →](./03-workflows-and-contributing.md)*
