# AGENTS.md - `subtract-first`

Contract pins and vocabulary unique to this skill; the four every skill carries (path, frontmatter `name`, frontmatter `description`, README to SKILL.md) are in the root [AGENTS.md](../../AGENTS.md#contract-pins-every-skill-carries) and apply here too. Everything below breaks consumers the same way: be deliberate, mention it in the commit.

## Skill-specific pins

- **The gate runs before the code exists.** The whole boundary against `/simplify` and the code review skills, which operate on finished code. Letting this skill sweep existing code for removable parts makes two skills do one job.
- **Three searches, fixed order and fixed names: reuse, extend, subtract.** Subtract is last because it is the one never generated unprompted, and it must be reached even when the first two already answered. The output carries one line per search for the same reason: a search with no slot in the report is a search nothing proves ran.
- **The trigger is a new name or a new layer**, not a line count. A line threshold fires on a long flat edit and misses a three-line wrapper, which is the change that actually costs later.
- **Three trigger modes, not a rule plus exceptions: full gate, reuse only, skip entirely.** The middle one is the already-decided case - the decision is not reopened, but duplicating something that exists is still worth catching. Folded back into the skip list it becomes a skip that does not skip.
- **The skip mode is load-bearing, not politeness.** Firing on edits inside an existing unit trains the user to ignore the gate, at which point the skill is worse than absent.
- **The output is the four labelled lines `Reuse` / `Extend` / `Subtract` / `Taking`, in chat, before the code.** Never a file. The labels are the contract: they make the gate's completeness readable from its output, and free prose lets `Extend` vanish silently. A written artifact arrives too late to overrule cheaply and duplicates `/work-report`.
- **"Nothing was removable" is a valid stated outcome.** A gate that must produce a subtraction every time would manufacture them, and the replication numbers in [README.md](README.md#why-this-exists) say the additive option legitimately wins most of the time.
- **Proposed removals carry their blast radius.** Hyrum's Law and Chesterton's fence are named in the skill so deletion reads as a change with consequences; the link and the debloating figure live in [README.md](README.md). Removing that section turns a bias correction into a licence.
- **[README.md](README.md) anchors into three SKILL.md headings:** `#the-gate`, `#when-to-run-it`, `#removal-is-the-permanent-cut`. Renaming any is a two-file change.
- **Frontmatter `allowed-tools`: `Grep, Glob, Read`.** A pre-approval, not a restriction - it drops the permission prompt for the three searches during the invoking turn. Read-only by design: the skill recommends, the surrounding session writes the code. Widening it to write or execute tools changes the security surface and is flagged by the pre-release Snyk scan.
- **The research citations are load-bearing.** The evidence that the additive default is a generation failure rather than a preference is what stops the instruction reading as ritual. [README.md](README.md#why-this-exists) holds the single copy of every citation and figure; [SKILL.md](SKILL.md) states the findings in words, because the model acts on the reason, not the provenance, and every figure there costs context on each activation. This file links, never restates.

## Vocabulary is deliberate

Every term below appears verbatim in [SKILL.md](SKILL.md). Renaming one is a multi-file change: the labels are the output contract, and the headings are what README anchors into.

| Term | What it names | Avoid |
| ---- | ------------- | ----- |
| **Subtraction gate** | The three searches plus the stated decision, run before new code is written. | check, review, audit, pass |
| **Reuse** | First search: does this already exist in the repo or a declared dependency, searched by concept rather than by the intended name. | find, lookup, dedupe |
| **Extend** | Second search: is widening something close cheaper than adding a sibling. | modify, generalize |
| **Subtract** | Third search: can the outcome come from removing something instead. | delete, simplify, refactor |
| **Taking** | The fourth output line: the option chosen, and what it deletes. | decision, result |
| **New named thing / new layer** | The trigger - a new file, symbol, abstraction, dependency, flag, or an indirection between existing parts. | big change, large diff |
| **Blast radius** | What a proposed removal could break beyond its callers, including behaviour depended on rather than promised. | risk, impact |
