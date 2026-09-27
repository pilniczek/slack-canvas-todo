---
name: subtract-first
description: >
  Forces the subtractive option to be considered before code is added. Fires at implementation
  time - before creating a new file, component, hook, abstraction, config flag or dependency,
  and again mid-change when a change outgrows its estimate. Runs three searches (reuse, extend,
  subtract), names what the addition makes removable, and states the one it picked. Use whenever
  about to write new code, "add a component", "create a helper", "install a package", "new
  abstraction", "wrap this", or when a diff is growing faster than expected. Not a cleanup pass
  on finished code - that is /simplify.
allowed-tools: Grep, Glob, Read
---

Run the **subtraction gate** before adding code, and again when a change outgrows what you
expected. Not writing less code as a virtue: people default to searching for additive changes, and
the option to *remove* something usually never gets generated at all, so it never gets compared
against the option to add. Cognitive load makes it worse, and a developer mid-task under deadline
is exactly that condition. What corrects it is being asked - this gate is that ask, while it still
changes the code.

Calibrate: even when asked, the additive option wins most of the time, and often deserves to. A
gate that always concludes "delete something" is broken the other way.

## When to run it

Run the **full gate** when the next step introduces a new **named thing** or a new **layer**:

- a new file, module, component, hook, class or exported symbol
- a new abstraction, wrapper, indirection or generic parameter
- a new dependency, script, config flag, environment variable or feature toggle
- a change that has grown well past what you predicted, mid-flight

Narrow to **reuse only** when the user has already decided. "Create a `UserAvatar` component in
`src/components`" is an instruction, not an open question: do not reopen it, but duplicating
something that exists is still the failure they will care about. Run reuse, report it in one line,
build what was asked.

**Skip entirely** for edits inside an existing unit that add no new name and no new layer: fixing
a condition, renaming, styles, values, testing code that exists. Ceremony on a one-line fix trains
the user to ignore the gate.

## The gate

Three searches, this order, cheap and parallel - Grep and Glob over the repo, Read on the one or
two candidates worth opening.

**1. Reuse.** Does this already exist here? Search by concept, not by the name you were about to
use - the existing thing is rarely called what you would have called it. Look for the domain noun,
the verb, the prop shape, the import that would already pull it in. Check the shared locations
(`components/`, `hooks/`, `utils/`, `lib/`) and the declared dependencies: a library already in
`package.json` beats both writing it and installing one.

**2. Extend.** If something close exists, is widening it cheaper than standing up a sibling? Two
near-identical components that drift apart cost more than one with a prop. The counter case is
real: extending something already carrying several flags is how a component becomes
unmaintainable. Say which of the two this is.

**3. Subtract.** Can the same outcome come from removing something instead? This one never gets
generated on its own, so generate it explicitly even when it feels unlikely. Recurring shapes:

- the special case disappears if the upstream data shape is fixed
- the wrapper disappears if the thing it wraps is called directly
- the flag disappears if the two branches were never both needed
- the state disappears if it can be derived
- the abstraction disappears if it has exactly one caller
- the guard disappears if the invalid state cannot be represented

## Then state the choice

Four labelled lines in chat, before writing the code. `Subtract` and `Taking` always appear;
`Reuse` and `Extend` get one line each, whether or not they found anything.

```text
Reuse: nothing matches, closest is `useDeviceState` (different lifecycle).
Extend: no near sibling worth widening.
Subtract: drop the `isPending` flag and derive it from `status`.
Taking: derive from `status`, no new hook.
```

Naming the subtractive option you did not take is what makes overruling you cheap. When adding
wins, which is often, `Taking` says what the addition makes removable:

```text
Reuse: three inline URL builds, no shared helper.
Extend: nothing close enough to widen.
Subtract: nothing removable on its own.
Taking: add `parseFeedUrl` in src/api, deleting the three inline builds in
useDeviceHeartbeat, useGetCars and lauraConfig in the same change.
```

"Nothing removable" is a valid answer - say it plainly rather than inventing a justification. The
report records that the trade was looked at, not that one was found.

## Removal is the permanent cut

The gate proposes removals, it does not authorise reckless ones. Hyrum's Law and Chesterton's
fence apply: code can be depended on for behaviour its contract never promised.

A proposed removal carries its blast radius. Before deleting, find the callers, say what could
depend on the behaviour rather than the contract, and treat anything exported past the repo
boundary as needing the user's explicit agreement. "The tests pass" is not "this was unused".

## What this is not

Not a cleanup pass over finished code - that is `/simplify` and the code review skills, and
running both on one change wastes the user's time. The distinguishing question is whether the
code exists yet.

Not a stall either. The gate is three searches and four lines. If it turns into a design
discussion you have overrun its purpose - state the trade, decide, keep going, let the user
redirect.
