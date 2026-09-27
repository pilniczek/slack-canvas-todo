# docs-consistency-check

A Claude skill that audits the prose layer of a project (READMEs, SKILL.md, CLAUDE.md, AGENTS.md, templates, manifests, changelogs, installers) for drift: one file updated and a related one not, or two files contradicting each other. It also flags content restated in several places that should live in one and be referenced, the usual source of the next drift.

It does not check code logic, verify external links, enforce formatting, or run in CI. Installers and manifests are read for the concepts they document, never analysed as programs.

## Install

As a Claude Code plugin (from the [dev-skills](https://github.com/pilniczek/dev-skills) marketplace):

```text
/plugin marketplace add pilniczek/dev-skills
/plugin install docs-consistency-check@dev-skills
```

Or vendor it into your repo with skills.sh:

```bash
npx skills add https://github.com/pilniczek/dev-skills --skill docs-consistency-check
```

[skills.sh/pilniczek/dev-skills](https://skills.sh/pilniczek/dev-skills/docs-consistency-check)

## What it catches

Every finding lands in one of five severity tiers:

- [🔴 Conflict](SKILL.md#-conflict)
- [⚠️ Outdated](SKILL.md#️-outdated)
- [↩️ Orphaned](SKILL.md#️-orphaned)
- [❓ Unverifiable](SKILL.md#-unverifiable)
- [🔁 Restated](SKILL.md#-restated)

## How it works

1. [Apply security guards, identify the file set](SKILL.md#step-1---apply-security-guards-then-identify-the-file-set)
2. [Load intentional variations](SKILL.md#step-2---load-intentional-variations): suppress pre-marked findings
3. [Count heuristic](SKILL.md#step-3---count-heuristic-fast-first-pass): declared invariants first, then list sizes
4. [Build the concept inventory](SKILL.md#step-4---build-the-concept-inventory): shared vocabulary, restatements tagged
5. [Cross-reference and classify](SKILL.md#step-5---cross-reference-and-classify) into the severity tiers
6. [Report findings](SKILL.md#step-6---report-findings), ordered by severity
7. [Apply fixes](SKILL.md#step-7---apply-fixes-and-manage-intentional-variations), or mark a ❓ or 🔁 finding intentional

Credential and secret files are excluded before any read, and findings paraphrase rather than quote. See [Security invariants](SKILL.md#security-invariants).

## Ignoring files

On first run the skill offers to create `.docs-consistency-check-ignore` (gitignore syntax) at your project root, seeded with the usual vendored skill directories. [Step 1](SKILL.md#step-1---apply-security-guards-then-identify-the-file-set) holds the template.

## Auto-activation

Claude offers this skill, asking first, when you mention updating or creating a README, SKILL.md, CLAUDE.md, AGENTS.md, template, plugin.json, changelog, or installer, or use keywords like "docs", "sync", "feature added", or "I just updated". Phrases like "check consistency", "in sync", "find inconsistencies", "verify everything is updated", "single source of truth", or "restated in multiple places" run it directly. Once invoked it [stays armed](SKILL.md#stay-armed-for-the-rest-of-the-session) for the rest of the conversation.

## Suppressing a finding permanently

An intentional ❓ or 🔁 finding can be recorded in `intentional-variations.md` and skipped on later runs: see [Step 7](SKILL.md#step-7---apply-fixes-and-manage-intentional-variations) to mark one and [Review intentional mode](SKILL.md#review-intentional-mode) to revisit it.

## Contributing

See [CONTRIBUTING.md](https://github.com/pilniczek/dev-skills/blob/master/CONTRIBUTING.md) for local setup and the pre-release security scan workflow.
