# AGENTS.md - `docs-consistency-check`

Contract pins unique to this skill, on top of the four in the root [AGENTS.md](../../AGENTS.md#contract-pins-every-skill-carries). Each is breaking for consumers: be deliberate, and mention it in the commit.

## Skill-specific pins

- Severity tier strings: `🔴 Conflict`, `⚠️ Outdated`, `↩️ Orphaned`, `❓ Unverifiable`, `🔁 Restated`. They appear verbatim in [SKILL.md](SKILL.md), [README.md](README.md), and every report the skill emits.
- Auto-activation triggers: the file-types list, the proactive keywords list, and the "Use for" phrases list appear verbatim in SKILL.md's frontmatter `description` and in the README's Auto-activation section.
- Git is optional: with a repo the audit may read git to sharpen a run; without one every step falls back to the conversation and the filesystem. A step that works only with git is as broken as one that ignores git when present.
- Report fixed strings: the per-finding template with its two consolidation `Fix:` shapes (the 🔁 keep-and-replace form and the `or:` form on 🔴 and ⚠️), the `Found N issues:` summary line, the skipped-files line, and the clean verdict's evidence lines. Other agents parse reports, so adding an evidence line means adding it here.
- Frontmatter `allowed-tools`: a pre-approval, not a restriction. Every command the body runs must appear there or the run stalls on a permission prompt. Widening it widens auto-approval and shows in the pre-release Snyk scan; the git entries are read-only by design.
- SKILL.md headings the README anchors into: the 7 step headings, the 5 tier sub-headings under Step 5, "Security invariants", "Stay armed for the rest of the session", and "Review intentional mode".

## `.docs-consistency-check-ignore` conventions

The skill's runtime config, kept at the consumer's repo root, not in the skill folder. The defaults seeded by the [Step 1](SKILL.md#step-1---apply-security-guards-then-identify-the-file-set) template are vendored dev-tool skill directories, since auditing them reports drift in other people's docs.
