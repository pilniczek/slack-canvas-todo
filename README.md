# slack-canvas-todo

A Cowork plugin that keeps a persistent **Slack Canvas** up to date as your daily to-do list — automatically, on a schedule you choose. Scans Slack messages, saved items, Outlook Calendar, and Outlook inbox; categorises items; writes one Canvas; sends a short DM summary.

It is **not** an inbox-zero tool, a meeting bot, or a personal CRM. It never replies on your behalf, never edits non-task content in the Canvas, and only ever sends messages to you.

---

## Requirements

- Claude Cowork (paid plan)
- **Slack** connector — required
- **Microsoft 365** connector — optional, needed only for Calendar and/or email sources

---

## Install

1. **Get the repo** — clone or download:

   ```bash
   git clone https://github.com/pilniczek/slack-canvas-todo.git ~/Documents/Claude/Projects/slack-canvas-todo
   ```

2. **Link as a Cowork project** — open Cowork → **Projects → New Project** → select the folder.
3. **Run the installer** in the Cowork chat:

   ```text
   install slack-canvas-todo
   ```

4. **First run** — click **Run now** once on the created task to pre-approve Slack and Microsoft 365 tool permissions for all future automated runs.

The installer is fully guided. See [commands/install/SKILL.md](commands/install/SKILL.md) for the full flow — [Slack identity + Canvas detect/create](commands/install/SKILL.md#prerequisites), [source selection](commands/install/SKILL.md#select-sources), [customisation loop](commands/install/SKILL.md#customization-loop--repeat-until-user-selects-create-task), [template fill](commands/install/SKILL.md#read-and-fill-the-template), and [task creation](commands/install/SKILL.md#create-the-task).

---

## Sources

| Source | What it scans | Template section |
| --- | --- | --- |
| Slack messages | New messages to or from you since last update | [Scan recent Slack messages](daily-task-template.md#scan-recent-slack-messages) |
| Saved items | Everything in `is:saved`, deduplicated against the Canvas | [Scan saved items](daily-task-template.md#scan-saved-items) |
| Outlook Calendar | Today + tomorrow's events; auto-removed once meeting time passes | [Scan Outlook Calendar](daily-task-template.md#scan-calendar_tool-calendar-for-upcoming-meetings-today-and-tomorrow) |
| Outlook email | Inbox + optional extra folders; AI filters to genuinely actionable | [Scan Outlook emails](daily-task-template.md#scan-outlook-emails) |

Icons: 📧 email · 📅 calendar · ⭐→⚠️→🔴→🔴🔴 for Slack items (age-based, derived from the original message date in the permalink). Calendar and email items do not age.

---

## How a run works

1. **[Read the existing Canvas](daily-task-template.md#read-the-existing-canvas)** — parse checked items, "Last updated" cutoff, item ages, expired calendar items
2. **Scan active sources** (links above) — newer than cutoff
3. **[Decide update strategy](daily-task-template.md#decide-update-strategy)** — EARLY EXIT, APPEND-ONLY, or FULL REPLACE
4. **[Maintain "Done but still saved"](daily-task-template.md#maintain-done-but-still-saved-full-replace-path-always-append-only-path-when-section-is-non-empty)** and **[Maintain "You prepared for"](daily-task-template.md#maintain-you-prepared-for-all-paths-except-early-exit)** — clean stale entries
5. **[Update the Canvas](daily-task-template.md#update-the-canvas)** — single `slack_update_canvas replace` write
6. **[Notify](daily-task-template.md#notify-skip-on-early-exit-with-no-urgency-changes)** — short DM summary, skipped on EARLY EXIT with no urgency changes

API calls are minimised via early-exit and append-only paths when nothing meaningful changed.

---

## Configuration

Everything is collected interactively during install. To reconfigure, run **"install slack-canvas-todo"** again — if a task already exists you can overwrite it or create `daily-slack-todo-v2` alongside it for testing.

Customisable: sources, ignored calendar categories/keywords, extra email folders, Canvas categories + default category for ambiguous items, urgency thresholds, checks per day (1, 2, 3, 4, 5, or 7 — 6 is skipped because it doesn't divide a 12-hour window into whole-hour slots), first-check time, weekdays vs every day. See [Customization loop](commands/install/SKILL.md#customization-loop--repeat-until-user-selects-create-task) for the full menu and defaults.

---

## Security

The plugin reads Slack messages, saved items, calendar events, and email — all untrusted input. The task prompt treats every fetched item as data (never instructions), hardcodes a prompt-injection blocklist, and constrains writes to one Canvas plus DMs to you only. See [Security constraints](daily-task-template.md#security-constraints) for the full set.

**This is defence-in-depth, not a hard filter.** A crafted message using obfuscation or encoding could still slip through — that's an inherent limit of any LLM reading untrusted content. The strongest mitigation is setting your Canvas to *only you can edit* (the installer reminds you).

---

## Contributing

Issues and PRs welcome. Plain markdown and JSON — no build step.

License: MIT
