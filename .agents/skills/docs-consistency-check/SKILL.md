---
name: docs-consistency-check
description: >
  Performs a structured cross-file consistency audit across documentation, configuration, and
  instruction files. Use whenever the user says "check consistency", "run a consistency check",
  "make sure files are in sync", "find inconsistencies", or "verify everything is updated after
  this change". Also offer to run proactively (without waiting to be asked) when the user
  mentions updating or creating any of these file types: README, SKILL.md, CLAUDE.md,
  AGENTS.md, template, plugin.json, changelog, or installer — or uses keywords like "docs",
  "sync", "config", "feature added", or "I just updated". In proactive mode, offer first rather
  than running silently. Supports a `review-intentional` mode to surface previously suppressed
  findings for re-evaluation.
---

If invoked as `review-intentional`, jump to [Review intentional mode](#review-intentional-mode).

## Step 1 — Load the ignore list and identify the file set

If `.docs-consistency-check-ignore` doesn't exist at the project root, offer to create it with the default template below and wait for the user's go-ahead before writing. If it exists, apply its patterns silently.

**Default template:**

```
# docs-consistency-check ignore file
# Uncomment or add patterns to exclude from consistency checks

.agents/
.claude/
# .git/
# node_modules/
# dist/
# build/
```

Always skip (hardcoded): `node_modules/`, `.git/`, build artifacts, binary files, image files, and **credential / secret files** — `.env`, `.env.*`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `id_rsa`, `id_ed25519`, `id_ecdsa`, `secrets.json`, `credentials.json`, `.netrc`, `.npmrc`, `.htpasswd`. Plus anything matching a pattern from `.docs-consistency-check-ignore` (`.gitignore` syntax). Drift in those files is out of scope, and reading them risks emitting secrets in findings.

Collect within the project root:

- Markdown files: `README.md`, `CLAUDE.md`, `AGENTS.md`, `SKILL.md`, other `.md`
- Template files: anything with `{{VARIABLE}}` or `[INCLUDE IF: ...]` syntax
- Manifest / config files: `plugin.json`, `.mcp.json`, `package.json`, etc.
- Any file explicitly mentioned in the conversation

Read each file in the inventory above — the ignore list bounds the set, so no recursive expansion. Read enough to identify concepts and drift signals; don't reproduce raw file content elsewhere.

**Mid-run exclusions:** If the user says "ignore vendor/ for this", apply it for the current run only. Persistence is the user's call.

---

## Step 2 — Load intentional variations

Read `intentional-variations.md` from the project root if it exists. Build a lookup of suppressed items — each entry records the affected files, what differs, why it's intentional, and when it was marked. Any finding that matches a suppressed entry (same files, semantically similar description) is silently skipped during reporting.

If the file doesn't exist, proceed with an empty suppressions list.

---

## Step 3 — Count heuristic (fast first pass)

Before the full inventory, count items in any list that looks exhaustive — sources, features, icons, steps, conditions. Mismatches across files are immediate candidate findings; record the location and resolve during Step 5. This is the highest-yield drift signal.

---

## Step 4 — Build the concept inventory

A "concept" is anything that appears in more than one file and could drift. **Start from instruction files** — `CLAUDE.md` and `AGENTS.md` define project vocabulary; those concepts take priority over the fallback taxonomy below.

**Fallback taxonomy:**

- **Features / sources** — Things the system supports; usually appear in installer/spec, template, and docs.
- **Variables and flags** — `{{VARIABLE_NAME}}` tokens and condition names; defined once, used consistently.
- **Icons and symbols** — Emoji or markers tied to concepts (📧, 📅, ⭐).
- **Conditional blocks** — `[INCLUDE IF: condition]...[/INCLUDE]`; conditions in blocks must match the conditions list.
- **Terminology** — Same concept must use the same name everywhere.
- **Exhaustive lists** — Anything enumerating "all of X". Count mismatches from Step 3 belong here.
- **Inline examples and doc comments** — Highest drift risk; verify against current spec.

Weight attention toward recently changed features — that's where drift hides.

---

## Step 5 — Cross-reference and classify

For each concept, check whether every file mentions it consistently. Classify each finding:

### 🔴 Conflict
Two or more files actively assert different values for the same fact. A human must decide.

### ⚠️ Outdated
One file was updated, another didn't catch up — the source of truth is clear.

### ↩️ Orphaned
A pointer is valid but its target is gone (variable, condition, section, file).

### ❓ Unverifiable
A difference exists but context is insufficient to call it a problem.

When a conflict involves an implementation file (template, installer) and a doc file (README, comment example), the implementation is usually the source of truth.

---

## Step 6 — Report findings

Skip findings matching an entry from Step 2's intentional variations.

Output a numbered list ordered by severity (🔴, ⚠️, ↩️, ❓). Within a tier, wider user impact first.

**Per-finding template:**

```
#N — [icon] [TierName]
Files: <file> (line ~A)[, <file2> (line ~B), …]
<file>: <short fragment or paraphrase identifying the differing concept>
[<file2>: <short fragment or paraphrase identifying the differing concept>]
Fix: <concrete edit — for ❓, a confirmation question instead>
```

Rules:

- **Show fragments, never raw content; redact any secrets.** When pointing to a difference, show a short fragment (a phrase, a count, a list of identifiers) — never copy whole lines or paragraphs. If a fragment would include a credential value (API key, token, password, private key, OAuth secret, JWT, connection string with embedded credentials), replace the value with `<REDACTED>` while keeping just enough context to identify the finding. Template placeholders (`{{API_KEY}}`, `${SECRET}`) are not credentials; show them as-is. The same rule applies to fix instructions and to any content the apply-fixes step (Step 7) writes back into files. Goal: the report should be useful to a human reviewer without ever reproducing arbitrary file content.
- One detail line per affected file. Single-file findings (often ↩️ Orphaned) get one detail line.
- If there's no fragment to show (e.g. dangling reference with no target), describe inline: `<file>: <reference> — never defined`.
- For ❓ Unverifiable, `Fix:` becomes a confirmation question (e.g. `Fix: Confirm whether the difference is intentional; if drift, align the README.`).
- No decorative whitespace alignment — single space after every colon.

**Example (⚠️ Outdated):**

```
#1 — ⚠️ Outdated
Files: README.md (line ~12), SKILL.md (line ~87)
README.md: lists 3 sources
SKILL.md: lists 4 sources (adds SOURCE_EMAIL)
Fix: Add the 4th source to the README's source description.
```

**Summary line** (always last, icon-only counts; emit both lines even when M=0):

```
Found N issues: X 🔴, Y ⚠️, Z ↩️, W ❓.
Skipped M intentional variations.
```

If no issues are found, output `No drift detected.` followed by a comma-separated list of files checked.

---

## Step 7 — Apply fixes and manage intentional variations

After reporting, ask: "Apply all fixes now, or go through them one by one?"

If the user picks "all fixes now", first list the files that will be modified and the per-file change count, then wait for an explicit confirmation before any Edit. Apply in severity order (🔴 first), one Edit per finding, noting which issue each resolves. For 🔴 conflicts, confirm the correct value with the user before editing. For ⚠️ Outdated findings where the source of truth is ambiguous, also confirm before editing.

For ❓ findings, ask per-item: "Real problem, or intentional? [Fix it / Mark as intentional / Skip for now]"

On **Mark as intentional**, append to `intentional-variations.md`:

```markdown
- files: [file-a.md, file-b.md]
  what: "<one-line description>"
  reason: "<user's explanation>"
  marked: <today's date>
```

If `intentional-variations.md` doesn't exist, tell the user it will be created to record this entry, then create it with this header first:

```markdown
# Intentional Variations
# Differences marked as intentional. Run `docs-consistency-check review-intentional` to revisit.
```

---

## Review intentional mode

When invoked as `docs-consistency-check review-intentional`:

1. Read `intentional-variations.md`. If missing or empty, report and stop.
2. Display each entry numbered:

```
#1 — Marked intentional on 2026-05-03
Files: README.md, SKILL.md
What: "README says 3 sources, SKILL.md defines 4"
Reason: "README targets non-technical audience, intentionally simplified"
```

3. Ask per entry: "Still intentional, or re-open as a finding?"
4. Remove re-opened entries and run a targeted check on those files immediately.
5. Keep confirmed entries.

---

## Stay armed for the rest of the session

A finished audit cycle doesn't end this skill's responsibility. For the remainder of the conversation, offer a focused re-audit when any of the following happen:

- ≥2 audit-set files are edited (by user or by Claude on the user's behalf)
- A file is structurally rewritten or has a section removed
- The user signals "done" / "ready to commit" / "looks good"

Scope the re-audit to the changed files and their contract pairs — full re-audits aren't usually needed.
