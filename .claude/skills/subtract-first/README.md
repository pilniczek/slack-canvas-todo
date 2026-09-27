# subtract-first

A Claude skill that makes the subtractive option get considered before code is added. It runs at implementation time, not afterwards: before a new file, component, hook, abstraction, config flag or dependency appears, and again mid-change when a change has grown past what was predicted.

## Why this exists

It corrects a measured cognitive bias, not a style preference. [Adams et al., Nature 592:258-261 (2021)](https://doi.org/10.1038/s41586-021-03380-y) ran eight experiments and found people "systematically default to searching for additive changes, and consequently overlook subtractive transformations", worse under cognitive load. Subtractive options are not weighed and rejected, they are never generated - and a developer mid-task under deadline is exactly the loaded condition.

A [preregistered replication](https://doi.org/10.1002/jocb.1535) (N = 477) reproduced it - 1155 additive ideas against 297 subtractive - and showed a plain verbal cue raises the share of people generating at least one subtractive idea (OR = 2.52). The intervention is being asked, and this skill is that ask, while it still changes the code. The same numbers set its calibration: even cued, additive won most of the time.

## Install

As a Claude Code plugin (from the [dev-skills](https://github.com/pilniczek/dev-skills) marketplace):

```text
/plugin marketplace add pilniczek/dev-skills
/plugin install subtract-first@dev-skills
```

Or vendor it into your repo with skills.sh:

```bash
npx skills add https://github.com/pilniczek/dev-skills --skill subtract-first
```

[skills.sh/pilniczek/dev-skills](https://skills.sh/pilniczek/dev-skills/subtract-first)

## The gate

Three searches over the repo: **reuse** (does this already exist, searched by concept rather than by the name you were about to use), **extend** (is widening something close cheaper than standing up a sibling), **subtract** (can the same outcome come from removing something instead). What each looks for, and the recurring shapes a subtraction takes, is in [The gate in SKILL.md](SKILL.md#the-gate).

The output is four labelled lines - `Reuse`, `Extend`, `Subtract`, `Taking` - stated in chat before the code is written, so the option taken and the subtractive option not taken are both visible and overruling either is cheap. When adding wins, which it often should, `Taking` says what the addition makes removable, and "nothing" is an acceptable answer.

## When it fires

On a new named thing or a new layer: a file, module, component, hook, exported symbol, abstraction, wrapper, dependency, script, config flag or feature toggle. Not on edits inside an existing unit that introduce no new name and no new layer, because ceremony on a one-line fix trains you to ignore the gate. [When to run it in SKILL.md](SKILL.md#when-to-run-it) draws the line.

## It is not a cleanup pass

`/simplify` and the code review skills work on code that exists. This one works on code that does not exist yet, where the cheapest subtraction is the one that stops it being written. Running both on the same change duplicates the work.

Nor does it license reckless deletion. A proposed removal carries its blast radius - [Hyrum's Law](https://www.hyrumslaw.com/), Chesterton's fence, and the measured failure rate of even automated, test-verified removal: [coverage-based debloating](https://arxiv.org/abs/2008.08401) stripped 68.3% of library bytecode and 81.5% of client projects still passed their tests, so roughly one client in five broke. So the skill finds the callers first and treats anything exported past the repo boundary as needing your explicit agreement. See [Removal is the permanent cut](SKILL.md#removal-is-the-permanent-cut).

## Contributing

See [CONTRIBUTING.md](https://github.com/pilniczek/dev-skills/blob/master/CONTRIBUTING.md) for local setup and the pre-release security scan workflow.
