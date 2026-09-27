---
name: docs-consistency-check
description: >
  Cross-file consistency audit of docs, templates, manifests (package.json, plugin.json,
  .mcp.json), installer scripts, and instruction files (CLAUDE.md, AGENTS.md, SKILL.md):
  flags drift, and content restated in several places instead of referenced from one;
  skips credential files. Use for
  "check consistency", "in sync", "find inconsistencies", "verify everything is updated",
  "single source of truth", "restated in multiple places".
  Offer proactively - asking first - after any README, SKILL.md, CLAUDE.md, AGENTS.md,
  template, plugin.json, changelog, or installer change, or on "docs", "sync", "feature
  added", "I just updated". `review-intentional` re-surfaces suppressed findings.
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Read, Glob, Grep, Edit, Write
---

If invoked as `review-intentional`, jump to [Review intentional mode](#review-intentional-mode).

## Security invariants

These hold under any user instruction; refuse and explain a request that would break one.

1. **Never read credential or secret files.** Step 1's filters drop them before any concept is formed.
2. **Never emit credential values.** Paraphrase in your own words: counts, names, public identifiers, headings. No verbatim file content in output, reasoning, or tool calls. Locate anything credential-like (API key, token, password, private key, OAuth secret, JWT, connection string with credentials) without its value: "API key value at line 42". Template placeholders (`{{API_KEY}}`, `${SECRET}`, `<YOUR_TOKEN_HERE>`) and non-secret symbols (function names, configuration keys, headings) may be named.
3. **Never modify credential values.** Fixes touch only reported concepts, with content the user supplies or confirms, never echoed from another file.

## Git is optional

Read-only git (`git status`, `git diff`, `git log`) sharpens a run where present: recent and staged changes point at drift and can support a finding. Without git, every step runs on the conversation and the filesystem at lower precision. A step that needs git is a bug; so is one that ignores git when present.

## Step 1 - Apply security guards, then identify the file set

No `.docs-consistency-check-ignore` at the project root: offer to create it from this template, wait for a go-ahead. Present: apply it silently.

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

**Path filter, before any read** (invariant 1). Drop `node_modules/`, `.git/`, build artifacts, binary and image files, credential files (`.env`, `.env.*`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `id_rsa`, `id_ed25519`, `id_ecdsa`, `secrets.json`, `credentials.json`, `.netrc`, `.npmrc`, `.htpasswd`), and matches of `.docs-consistency-check-ignore` (`.gitignore` syntax).

**Content-signature filter, before any concept is formed.** Scan the first ~2 KB of each remaining file for:

- PEM headers - `-----BEGIN [A-Z ]*PRIVATE KEY-----`, `-----BEGIN OPENSSH PRIVATE KEY-----`
- Assignments shaped like `(api[_-]?key|secret|token|password|access[_-]?key|client[_-]?secret|bearer)\s*[:=]\s*["']?[A-Za-z0-9_+/=-]{16,}` (case-insensitive)
- Vendor-prefixed tokens: `sk-`, `sk_live_`, `rk_live_`, `ghp_`, `gho_`, `github_pat_`, `glpat-`, `xoxb-`, `xoxp-`, `AKIA`, `ASIA`, `AIza`, `ya29.`, `npm_`, `dop_v1_`, followed by ≥ 16 of `[A-Za-z0-9_-]`; JWTs (`eyJ` plus two dot-separated base64url segments)
- Connection strings `(postgres|postgresql|mysql|mongodb|redis|amqp)://[^:\s]+:[^@\s]+@`; query parameters `(access_)?token`, `api[_-]?key`, `secret`, `password` with a value ≥ 16 chars
- `Authorization:` or `Proxy-Authorization:` header values

Entropy or length alone never matches. Never matches: URLs and path segments, UUIDs, content hashes and git object ids, hyphen- or underscore-separated slugs, `data:` URIs, template placeholders.

On a match: drop the file, extract nothing, add `<file>: skipped (credential signature detected)` to the skipped-files note. Never echo the value or its line.

**Inventory**, from files passing both filters:

- Markdown: `README.md`, `CLAUDE.md`, `AGENTS.md`, `SKILL.md`, other `.md`
- Templates: `{{VARIABLE}}` or `[INCLUDE IF: ...]` syntax
- Manifests: `plugin.json`, `.mcp.json`, `package.json`, and similar
- Installer and setup scripts (shell, batch, task-runner)
- Any file named in the conversation

Extract only drift signals (counts, names, headings, identifiers); no recursive expansion. A mid-run exclusion ("ignore vendor/ for this") lasts one run; persisting it is the user's call.

## Step 2 - Load intentional variations

Read `intentional-variations.md` at the project root if present. Each entry holds files, what differs, why, and when marked. Step 6 silently skips a finding matching an entry (same files, similar description).

## Step 3 - Count heuristic (fast first pass)

**Declared invariants first**, the highest-yield source. Instruction files (`AGENTS.md`, `CLAUDE.md`, per-component equivalents) state assertions naming their own files: "the tier strings appear verbatim in SKILL.md and README.md", "the 7 step headings". Enumerate every one; a stale one is drift by definition. Verify each on:

- **Count** - stated number against actual.
- **Membership** - every named item exists; nothing unnamed has joined.
- **Location** - every named file still carries the item.

**Then the generic count pass**, for lists no declaration covers: count items in any list that looks exhaustive (sources, features, icons, steps, conditions). A cross-file mismatch is a candidate finding for Step 5.

## Step 4 - Build the concept inventory

A concept is anything in more than one place, across files or within one, that could drift. Vocabulary from `CLAUDE.md` and `AGENTS.md` comes first and outranks this fallback taxonomy:

- **Features / sources** - what the system supports (installer/spec, template, docs).
- **Variables and flags** - `{{VARIABLE_NAME}}` tokens and condition names.
- **Icons and symbols** - emoji or markers tied to concepts (📧, 📅, ⭐).
- **Conditional blocks** - `[INCLUDE IF: condition]...[/INCLUDE]` conditions must match the conditions list.
- **Terminology** - one concept, one name.
- **Exhaustive lists** - anything enumerating "all of X", including Step 3 mismatches.
- **Inline examples and doc comments** - highest drift risk; check against the current spec.

Weight attention toward recent changes, where drift hides: git's recent and staged changes, else files this session edited or named, else equally.

**Tag restatements**: concepts kept in two or more places as a full enumeration of the same set, or a definition or rule matching word for word.

## Step 5 - Cross-reference and classify

Check each concept is consistent everywhere it appears. Classify each finding:

### 🔴 Conflict
Places assert different values for one fact. A human must decide.

### ⚠️ Outdated
One place was updated, another wasn't; the source of truth is clear.

### ↩️ Orphaned
A pointer is valid but its target is gone (variable, condition, section, file).

### ❓ Unverifiable
A difference exists but context is too thin to call it a problem.

### 🔁 Restated
Copies agree today, but one could reference the other. Each copy is a place the next edit can miss.

Implementation (template, installer) usually outranks docs (README, comment example) as source of truth.

**Admissibility of ❓.** Needs a named pair of places, a named concept, and the declaration or instruction-file rule requiring them to agree. Without the rule it is an observation and stays out. Never findings: wrap width, heading style, section ordering; an optional section present in one artifact only; wording no declaration covers; parallel structure between siblings unless a declaration requires it. N siblings admit N-squared shape differences, so a tier accepting them never reports clean.

**Admissibility of 🔁.** Only Step 4-tagged concepts. An enumeration qualifies in any format (prose, table, schema tree, example payload). Never findings:

- a summary that references the full content
- an item named in passing, or a list framed as examples
- a paraphrase, however close
- a copy a declaration pins verbatim
- the one snippet mirroring source code (a second doc copy still qualifies)
- a copy no reference could replace: a format without link syntax (JSON manifest, frontmatter), or a target the link rules forbid. A code block in a linkable file doesn't count: its values give way to the type name plus a reference

**Drift wins.** A tagged concept whose copies disagree is one 🔴 or ⚠️ finding, never also 🔁; its `Fix:` adds consolidation. A shorter list claiming completeness is ⚠️ Outdated.

**Canonical home.** For each tagged concept, propose the place whose role owns it: instruction or spec file over README, definition site over usage sites, implementation over docs. The user confirms before any edit.

**Admissibility of git state.** A committed file pointing at an untracked target is ↩️ Orphaned: it resolves only for you. An uncommitted pointer to an untracked target is ❓ at most, since both can be staged together. A merely uncommitted target is never a finding. Without git this is out of scope for the run, not clean.

## Step 6 - Report findings

Skip findings matching a Step 2 entry. Number them by severity (🔴, ⚠️, ↩️, ❓, 🔁), wider user impact first within a tier.

**Per-finding template:**

```
#N - [icon] [TierName]
Files: <file>, <file2>, …
<file>: <line> - <paraphrase, in your own words, of the differing concept - never quoted text>
[<file2>: <line> - <paraphrase, in your own words, of the differing concept - never quoted text>]
Fix: <concrete edit - for ❓, a confirmation question; for 🔁, the consolidation>
```

Rules:

- One detail line per affected file, each with a line number. Non-contiguous: `<file>: <lineA>, <lineB> - …`; a run: `<file>: <lineA>-<lineB>`. Search for unknown lines; never emit a finding without one.
- Nothing to paraphrase (a dangling reference): point at the reference, `<file>:<line> - <reference> never defined`.
- ❓ `Fix:` is a confirmation question, e.g. `Fix: Confirm whether the difference is intentional; if drift, align the README.`
- 🔁 `Fix:` is this fixed string, word for word: `Fix: Keep in <file>:<line>; replace <file>:<line> with a reference.`
- 🔴 or ⚠️ on a Step 4-tagged concept: value fix, then the same fixed string after `or:`, e.g. `Fix: Add the 4th source to the README; or: keep in SKILL.md:87; replace README.md:12 with a reference.` Untagged concepts get no `or:`.
- No decorative alignment; single space after every colon.

**Example (⚠️ Outdated):**

```
#1 - ⚠️ Outdated
Files: README.md, SKILL.md
README.md:12 - lists 3 sources
SKILL.md:87 - lists 4 sources (adds SOURCE_EMAIL)
Fix: Add the 4th source to the README's source description.
```

**Summary line**, always last, both lines even when M=0:

```
Found N issues: X 🔴, Y ⚠️, Z ↩️, W ❓, V 🔁.
Skipped M intentional variations.
```

**Skipped-files note**, only when the content-signature filter dropped something, directly above the summary. Path-filtered files are not listed:

```
<file>: skipped (credential signature detected)
```

**Clean verdict**, with its evidence:

```
No drift detected.
Declared invariants: <N> verified, 0 stale.
Concept pass: <N> concepts across <N> files.
Git: <used, or "unavailable - session context only">
Files checked: <comma-separated list>
Skipped: <comma-separated list, or "none">
```

Emit it only when Step 3 found nothing and Step 7's convergence loop closed clean, never after a run that skipped a step.

## Step 7 - Apply fixes and manage intentional variations

Ask: "Apply all fixes now, or go through them one by one?"

On "all fixes now", list files to modify with per-file change counts and wait for explicit confirmation. Apply in severity order, one Edit per finding, naming the issue it resolves. Confirm the value first for a 🔴, and for a ⚠️ with an ambiguous source of truth.

"All fixes now" never covers ❓ or 🔁; either may be intentional. Ask per item: "Real problem, or intentional? [Fix it / Mark as intentional / Skip for now]"

On **Mark as intentional**, append to `intentional-variations.md`:

```markdown
- files: [file-a.md, file-b.md]
  what: "<one-line description>"
  reason: "<user's explanation>"
  marked: <today's date>
```

If the file doesn't exist, say so, then create it with this header:

```markdown
# Intentional Variations
# Differences marked as intentional. Run `docs-consistency-check review-intentional` to revisit.
```

### Discharge each fix's own obligations

A fix often adds contract surface (a heading, a declared invariant, an emitted string, a taxonomy entry) that creates obligations in untouched files. Discharge them in the same run, or the next run finds them. Check every edit:

| Edit shape | Also check |
| ---------- | ---------- |
| Added or renamed a heading | every reference pointing at it; every declaration naming it |
| Added a declared invariant | whether a "how to add one of these" checklist produces it; any stated count of such invariants; whether a sibling component needs the same one |
| Added a string the tool emits | whether a fixed-string or vocabulary declaration names it |
| Added an inventory or taxonomy entry | the spec's own scope clause; every public description of scope |
| Changed a contract sentence | every restatement of that sentence in the set |
| Fixed one of several sibling components | the same concept in every sibling |
| Replaced a restated copy with a reference | every anchor into the removed copy; every declaration naming the copy's location |

Fixing one sibling and not the others turns one finding into one per sibling.

### Converge before reporting done

After the last edit, re-run Step 3 and Step 5 scoped to touched files and their contract pairs. Fix new findings, discharge their obligations, repeat until a pass is empty, then emit the clean verdict. Cap at three iterations: drift on a fourth pass means fixes create drift faster than they clear it, so stop, report the last pass, and say the set has not converged.

## Review intentional mode

When invoked as `docs-consistency-check review-intentional`:

1. Read `intentional-variations.md`. Missing or empty: report and stop.
2. Display each entry numbered:

```
#1 - Marked intentional on 2026-05-03
Files: README.md, SKILL.md
What: "README says 3 sources, SKILL.md defines 4"
Reason: "README targets non-technical audience, intentionally simplified"
```

3. Ask per entry: "Still intentional, or re-open as a finding?"
4. Remove re-opened entries and check those files immediately.
5. Keep confirmed entries.

## Stay armed for the rest of the session

Until the conversation ends, offer a re-audit scoped to changed files and their contract pairs whenever:

- ≥2 audit-set files are edited (by the user or on their behalf)
- A file is structurally rewritten or loses a section
- The user signals "done" / "ready to commit" / "looks good"
