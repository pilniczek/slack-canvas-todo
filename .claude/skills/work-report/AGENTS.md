# AGENTS.md - `work-report`

Contract pins and vocabulary unique to this skill; the four every skill carries are in the root [AGENTS.md](../../AGENTS.md#contract-pins-every-skill-carries). Changing anything below breaks consumers: be deliberate, mention it in the commit.

## Skill-specific pins

- Distribution vendors [SKILL.md](SKILL.md) as-is. Nothing outside it configures the skill: no install step, no env overrides, no files expected on disk.
- The `.gitignore` line lives in the skill: [SKILL.md](SKILL.md) Step 3 offers it on first run in a repo. A decline never blocks the report.
- `/WORK-REPORT.md` keeps its leading slash; a bare `WORK-REPORT.md` would match by basename at any depth.
- Frontmatter `allowed-tools` is a pre-approval, not a restriction: it drops permission prompts for listed tools during the invoking turn and cannot stop an unlisted tool (that is `disallowed-tools`, unused). Every command the body runs must be listed or the run stalls on a prompt (`grep` for the gitignore check, the exact `printenv CLAUDE_CODE_SESSION_ID` - never a bare `printenv`, which would pre-approve dumping every environment secret). Widening the list widens auto-approval, which the pre-release Snyk scan flags. Safety comes from the offer-first gates.
- Shipping steps (commit, push, PR, merge, tag, release) are never a *Follow-ups* item.
- Skill invocations live only in *Suggested skills*. The `only if it advances an [action] item` rule maps means to outcome: the follow-up states the outcome, the *Suggested skills* line names the tool.
- *Problem → Solution*'s entry rule governs what the report records anywhere: **a course changed after a first attempt failed**, **a real fork (≥2 viable options) taken**, or **a follow-up the report already listed, once finished and not recoverable from the diff** - whoever closed it. The same test dispositions every carried follow-up in [Step 2](SKILL.md#step-2---read-the-existing-report-and-build-on-it): passing becomes an entry, failing leaves no trace, which bounds the report across resumed sessions. Never add a rule recording work for its own sake; git is the changelog.
- Verification is content, not a section: an exercised check is a *Problem → Solution* entry or a clause in the **Solution** it verifies; an unexercised one is an `[action]` naming it.
- An `[action]` with no owner is valid. A gap already decided against (accepted risk, platform out of scope) is *Abandoned* `[considered]`, or `[ignored]` when the user or a rule the user set dismissed it; keeping it in *Follow-ups* would re-offer refused work.
- `[ignored]` carries the user's authority: it binds every later run as a veto, so one without that authority would bury work the user never agreed to drop. The authority comes directly (an explicit dismissal) or through a rule the user stated or confirmed this session, or wrote in the repo's instruction files, cited in the entry. The skill recognises the kind of decision, never the tool that applied it, so no other skill is named and none needs to know this one. An agent's own rejection or an unconfirmed skill default is `[considered]`; silence, an unanswered question, or a deferral stays in *Follow-ups*. The user lifts it by asking for the work again or by changing the rule. The veto is questioned, never overturned: a defect this session sharing a root cause or failing scenario with an entry adds a reopen `[question]`, and the entry stays.
- An existing report is extended by default, since another agent's report is usually a handoff. [Step 2](SKILL.md#step-2---read-the-existing-report-and-build-on-it) asks first when the *Goal* is unrelated, or when both branch and session id differ; the question names what a fresh start drops. A matching session id waives only the branch test, so one session can build variants on several branches yet still gets asked on unrelated work.
- Why a purpose-built report rather than an existing tool: [docs/other-options.md](../../docs/other-options.md).
- The offer-first guarantee is stated three times - frontmatter `description`, the SKILL.md body gate, the README's Auto-activation section - because it must hold at activation time, before the body loads. All three must agree: proactive activation offers and waits, an explicit invocation is the go-ahead.

## Vocabulary is deliberate

Other agents read the report, so its shape is an interface. Every term below appears verbatim in [SKILL.md](SKILL.md); renaming one is a multi-file change. **Goal**, **Problem → Solution**, **Follow-ups**, **Suggested skills**, **Abandoned** are also the report's headings, in that order, always present, `- none` under an empty one. [Step 4](SKILL.md#step-4---compose-the-report) defines their content; this table pins naming and the *Avoid* list.

| Term | What it names | Avoid |
| ---- | ------------- | ----- |
| **Work report** | `WORK-REPORT.md` at a working tree's root. One per tree, overwritten in place, gitignored by default. | summary, handoff doc, session log, journal |
| **Self-report** | The stance: what the author *did and intended* this chat session, never reconstructed from git history. | audit, proof, reconstruction |
| **The work** | Everything done in the current chat session, not the branch's whole history. | the diff, the changes, the branch |
| **Goal** | What the work set out to do. | objective, purpose |
| **Problem → Solution** | Each problem and the approach that worked, plus any choice or check not recoverable from the diff. | challenges, issues, notes, changelog, verification |
| **Follow-ups** | What the next session picks up, tagged `[question]` or `[action]`. Excludes decided-against gaps, shipping steps, skill invocations. | open questions, next actions, TODOs |
| **Suggested skills** | Skills the next agent should invoke. | recommendations, tooling |
| **Abandoned** | What the next agent should not re-walk, tagged `[tried]`, `[considered]`, or `[ignored]` (the user's authority, direct or through a rule the user set). | abandoned approaches, discarded, dead ends, rejected |
| **Session id** | The session's UUID, the transcript filename under `~/.claude/projects/`. A match waives the Step 2 branch test. | session, chat id |

Fixed strings: the heading `# Work report`; the identifier lines `branch:` (where the work started, carried when extending) and `session id:` (last writer), one fact per line, never merged; the fallback values `(detached)`, `(no git)`, `unknown`; the empty-section marker `- none`. Readers grep for all of them.
