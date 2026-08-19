# Playbook: Migrating Code

*[← Previous: Starting a New Feature](./01-starting-a-new-feature.md) | [Next: Debugging a Bug →](./03-debugging-a-bug.md)*

## Situation

You're moving code from one pattern, library, or API to another — Options API to Composition API, one HTTP client to another, a deprecated component to its replacement. Correct behavior already exists. The job is to preserve it in a new shape, not to redesign it. This is a different situation from [Starting a New Feature](./01-starting-a-new-feature.md), where nothing exists yet — here, everything exists, and the risk is losing it.

## Steps

1. **Establish the explicit mapping before touching any code.** What maps to what, field by field, method by method? What has no equivalent in the target shape, and what do you do about that gap? Write this down the same way Module 4 treats a contract — it's the thing you're implementing against, not something to discover mid-migration.
2. **Migrate one small vertical slice first.** Pick the smallest self-contained piece — one component, one endpoint call — and port it as a proof of the mapping. Review that one slice on its own before scaling the approach out to the rest.
3. **Verify behavior parity, not just "it compiles."** A migrated component that type-checks and renders is not the same as one that behaves identically to the original — check the actual outputs, edge cases, and error paths against the old version. See [Verification](../03-workflow/06-verification.md).
4. **Repeat slice by slice, keeping each diff reviewable.** Don't batch the rest of the migration into one pass once the first slice looks right — the same incremental-implementation discipline from [Plan Before Code](../03-workflow/01-plan-before-code.md) applies here: one step, reviewed, then the next.

## Quick example

Migrating a class-based `OrderList` component to hooks. The mapping comes first, on paper:

```text
this.state.orders          → const [orders, setOrders] = useState(...)
componentDidMount           → useEffect(() => { fetch... }, [])
componentDidUpdate(prevProps.filter) → useEffect([filter])
this.setState callback form → no direct equivalent — needs a
                               functional update or a ref, decide which
```

Then the migration prompt for the first slice, with the constraint stated explicitly:

```text
Migrate OrderList's componentDidMount/componentDidUpdate pair
to useEffect, following this mapping:

componentDidMount -> effect with [] deps
componentDidUpdate(prevProps.filter) -> effect with [filter] deps

Preserve behavior exactly. Do not rename props, do not change
the render output, do not introduce new dependencies. This is
a mechanical port, not a cleanup.
```

Review that one effect against the original lifecycle behavior before touching `componentWillUnmount` or anything else in the file.

## Watch-outs

* **AI silently "improving" code mid-migration.** This is the single biggest failure mode in a migration: a rename here, a restructured function there, a new dependency pulled in because it made the new shape "nicer." Each of those is a refactor wearing a migration's clothes, and it's exactly what [AI Refactoring](../03-workflow/04-ai-refactoring.md) warns against for refactors generally — except here it's worse, because it makes behavior-parity impossible to verify. You can no longer tell whether a difference in output is the migration or the "improvement."
* **The fix is an explicit constraint, stated up front and restated at each slice:** "preserve behavior exactly, do not improve along the way." If a real improvement is worth making, it's a separate, explicit task that happens after the migration is verified — not something that rides along inside it.
* **Treating "it compiles" as done.** A migration between typed shapes can compile cleanly while silently changing behavior — a wider type, a dropped null check, a default parameter that used to be required. Compilation is the floor, not the finish line.

## Related reading

* [AI Refactoring](../03-workflow/04-ai-refactoring.md) — the same "preserve behavior, don't improve along the way" discipline, applied to refactors; the reasoning transfers directly.
* [Plan Before Code](../03-workflow/01-plan-before-code.md) — incremental implementation, one reviewable step at a time.
* [Verification](../03-workflow/06-verification.md) — why "it compiles" is not "it's correct," and what checking behavior parity actually requires.

## Try it

Before your next migration, write the old-shape-to-new-shape mapping down explicitly — including anything with no equivalent — and state "preserve behavior exactly, do not improve along the way" as its own line in your first prompt. Migrate one slice, verify parity against the original, and only then decide whether to continue.

---

*[← Previous: Starting a New Feature](./01-starting-a-new-feature.md) | [Next: Debugging a Bug →](./03-debugging-a-bug.md)*
