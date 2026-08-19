# Playbook: Prompt Quick Reference

*[← Previous: Managing a Session](./08-managing-a-session.md) | [Next: Standard AI Workflow for the Team →](../06-team-standard/01-standard-ai-workflow.md)*

## Situation

You know which playbook applies, but you want a real, copy-paste starting prompt instead of writing one from scratch. Every prompt below is pulled directly from Anthropic's own [prompt library](https://code.claude.com/docs/en/prompt-library) — not paraphrased, not invented for this guide — grouped by which playbook it backs. `{curly braces}` mark the slots you fill in for your own task.

## Steps

1. **Find the row that matches your situation** in the table below.
2. **Fill in the slots**, don't send the template as-is — a prompt with `{path}` still in it tells AI nothing.
3. **Read the "why it works" column before you copy it blindly.** The point of each prompt is a pattern, not the exact wording — once you see why it works, you can write your own variant for a situation the table doesn't cover.

## Reference table

| Playbook | Prompt | Why it works |
|---|---|---|
| [Starting a New Feature](./01-starting-a-new-feature.md) | `I want to build {feature}. interview me about implementation, UX, edge cases, and tradeoffs until we have covered everything, then write the spec to SPEC.md` | Ask to be interviewed instead of writing the spec yourself — AI asks structured questions until the requirements are complete, instead of you guessing what's missing. |
| [Starting a New Feature](./01-starting-a-new-feature.md) | `plan how to refactor the {target} to {goal}. list the files you would change, but don't edit anything yet` | "Don't edit yet" separates exploration from changes, so you see the approach before any code moves — the literal Analyze → Plan split from [Plan Before Code](../03-workflow/01-plan-before-code.md). |
| [From Docs to Code](./05-from-docs-to-code.md) | `read {input} and write up the action items, then create a {tracker} ticket for each with acceptance criteria` | Turns an unstructured doc or meeting note into a reviewable contract (tickets with acceptance criteria) instead of leaving the gaps implicit. |
| [Migrating Code](./02-migrating-code.md) | `migrate everything from {from} to {to}: identify every place that needs to change, then make the changes` | Asking to identify every call site *first* means the list comes back in the response, so you can check nothing was missed before trusting the migration is complete. |
| [Migrating Code](./02-migrating-code.md) | `port {source} to {target}, keeping the same {keep}` | Naming what must stay the same (API shape, test behavior) gives AI a contract to check the port against — exactly the "preserve behavior exactly" constraint this playbook's Watch-outs section warns you to state explicitly. |
| [Updating Existing Code](./04-updating-existing-code.md) | `look at how {example} is implemented to understand the pattern, then build {new} the same way` | Point at code you already like. Without a reference, AI defaults to generic best practices; with one, it matches the conventions your codebase actually uses. |
| [Updating Existing Code](./04-updating-existing-code.md) | `add a {endpoint} endpoint that returns {payload}` | States inputs and outputs, not implementation steps — AI finds where similar code lives and adds yours alongside it instead of inventing a new pattern. |
| [Debugging a Bug](./03-debugging-a-bug.md) | `the {test} test is failing, find out why and fix it` | Describes the symptom, not the file — AI runs the test to see the actual failure and traces it into source, instead of guessing from a description. |
| [Debugging a Bug](./03-debugging-a-bug.md) | `here is a build error. fix the root cause and verify the build succeeds` *(paste the error)* | "Root cause" plus "verify" in the same prompt prevents a surface-level patch that suppresses the error without actually fixing it — see [Verification](../03-workflow/06-verification.md). |
| [Debugging a Bug](./03-debugging-a-bug.md) | `users are seeing {symptom} on {where}. investigate and tell me what is going on` | Gives a symptom and a location, not a diagnosis — AI reads the actual code path and traces likely causes instead of you pre-guessing the mechanism. |
| [Dividing Work with Sub-Agents](./07-dividing-work-with-subagents.md) | `use a subagent to review {path} for security issues and report what it finds` | A subagent runs the review in its own context window and reports back a summary, so a long investigation doesn't fill up your main session — the containment payoff this playbook's Steps section describes. |
| [Using a Skill](./06-using-a-skill.md) | `create a /{name} skill for this project that {steps}` | Name the steps once, reuse them as a command — this is the literal mechanic behind the Advanced Track's [contribution loop](../07-advanced-track/03-workflows-and-contributing.md). |
| [Using a Skill](./06-using-a-skill.md) | `write a hook that {action} after every {event}` | For behavior that must happen with zero exceptions (see the Advanced Track's [hooks section](../07-advanced-track/01-anatomy-of-a-claude-project.md)), a hook doesn't depend on the model remembering to do it — a skill or a rule only asks. |
| [Managing a Session](./08-managing-a-session.md) | `that is not right: {feedback}. try a different approach` | Naming the specific constraint that was missed, not just "that's wrong," gives AI something concrete to satisfy on the retry instead of guessing again in a different wrong direction. |
| [Managing a Session](./08-managing-a-session.md) | `that is too much. keep only the changes to {scope} and undo your other edits` | For when the direction was right but the change went too broad — narrows scope without discarding the part that was correct. |
| [Managing a Session](./08-managing-a-session.md) | `you keep {mistake}. add a rule to CLAUDE.md so this stops happening` | A correction typed into chat isn't shared with anyone else's session. The same correction written into `CLAUDE.md` is — see the Advanced Track's [Skills vs. Rules](../07-advanced-track/02-skills-and-rules.md). |
| [Managing a Session](./08-managing-a-session.md) | `summarize what we did this session and suggest what to add to CLAUDE.md` | Ask before you forget. AI knows what it had to figure out this session and can propose the entries so the next session — yours or a teammate's — starts with that context already loaded. |

## Watch-outs

* **Copying a template without filling every slot.** A `{path}` or `{feature}` left in the prompt is a prompt that says less than a plain sentence would have.
* **Treating this table as exhaustive.** It's a starting set, not the full library — the [live prompt library](https://code.claude.com/docs/en/prompt-library) is searchable and has roughly 45 prompts across the full software lifecycle, including onboarding, data analysis, and incident response that don't have a dedicated playbook here.

## Related reading

* [Official prompt library](https://code.claude.com/docs/en/prompt-library) — the full, searchable source these are pulled from, with a "why this works" note under every entry.
* [Official common workflows](https://code.claude.com/docs/en/common-workflows) — the step-by-step recipes several of these prompts are drawn from.

## Try it

Next time you're about to write a prompt from scratch for a task that fits one of the eight playbooks, check this table first. If nothing fits, that's a real gap — consider whether it's worth its own future playbook entry.

---

*[← Previous: Managing a Session](./08-managing-a-session.md) | [Next: Standard AI Workflow for the Team →](../06-team-standard/01-standard-ai-workflow.md)*
