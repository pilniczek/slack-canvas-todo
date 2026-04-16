# slack-canvas-todo

A Cowork plugin that keeps a persistent **Slack Canvas** up to date as your daily to-do list — automatically, on a schedule you choose.

---

## What it does

- Scans **Slack messages** (to/from you) and **saved items** for action items and things to learn
- Pulls **calendar events** (Outlook or Google) and adds meeting prep reminders
- Writes everything to a **Slack Canvas** with urgency icons that age over time: ⭐ → ⚠️ → 🔴 → 🔴🔴
- Tracks **"Done but still saved"** — items you checked off but haven't unsaved yet
- Runs on a **cron schedule** you configure (default: twice daily, weekdays)
- Uses **early exit and append-only paths** to minimise API calls when nothing changed

---

## Requirements

- Claude Cowork (paid plan)
- **Slack** connector installed in Cowork
- **Microsoft 365** connector (optional — needed for Outlook Calendar)

---

## Installation

1. Download or clone this repo
2. In Cowork, add the `slack-canvas-todo` folder to your workspace (click the folder icon to select it)
3. Say **"install slack-canvas-todo"** (or "set up my Slack to-do list") and follow the prompts

The installer will:

1. Auto-detect your Slack identity (name, user ID, workspace)
2. Find or create your **TO DO** Canvas
3. Ask which sources to enable (messages / saved items / calendar)
4. Show a summary with all defaults pre-filled — customize only what you want
5. Create a scheduled task that runs automatically

**First run:** after setup, click **Run now** once on the created task to pre-approve tool permissions for all future automated runs.

---

## Configuration

All settings are collected interactively during install. You can customise:

| Setting | Default |
|---|---|
| Sources | Messages + Saved items + Calendar |
| Calendar tool | Outlook (Microsoft 365) |
| Ignored calendar categories | _(none)_ |
| Ignored meeting keywords | _(none)_ |
| Canvas categories | To Do, To Learn |
| Default category for ambiguous items | To Learn |
| Urgency thresholds | ⭐ 0–8d · ⚠️ 9–15d · 🔴 16–22d · 🔴🔴 23+d |
| Checks per day | 2 (8:00 + 20:00) |
| Days | Weekdays only |

To reconfigure, say **"install slack-canvas-todo"** again. If a task already exists, the installer will ask whether to overwrite it or create a `daily-slack-todo-v2` alongside it for testing.

---

## How the Canvas looks

```
# TO DO

_Last updated: 2026-04-16, 20:00 CEST_

---

## To Do

- [ ] 📅 Prepare for: Product Sync (14:00) — _Calendar, Alice Brown, 5 attendees_
- [ ] ⚠️ Fix login bug — review ticket and test locally — _Bob K., #dev, [Mar 30](https://...)_
- [ ] ⭐ Log time for this week — _DM, [Apr 16](https://...)_

## To Learn

- [ ] ⭐ New feature demo — watch recording — _Carol M., #general, [Apr 1](https://...)_

## Done but still saved

- ✅ Check API response format — _[Dec 16](https://...)_ — unsave in Slack to clear

---

_Urgency: 📅 calendar · ⭐ OK (0–8 days) · ⚠️ warning (9–15 days) · 🔴 critical (16–22 days) · 🔴🔴 overdue (22+ days)_
_Checked items are removed during the next daily update._
```

---

## Files

```
slack-canvas-todo/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── commands/
│   └── install/
│       └── SKILL.md             # Install skill — triggered by "install slack-canvas-todo"
├── daily-task-template.md       # Parametrized prompt — filled in by the installer
├── .mcp.json                    # Required connector declarations
└── README.md
```

**`daily-task-template.md`** is the source of truth for the scheduled task prompt. The installer reads it from your local workspace folder, fills in all `{{VARIABLES}}` and evaluates `[INCLUDE IF: ...]` blocks based on your choices, then registers the result as a scheduled task. To change the prompt logic, edit this file and re-run the installer.

---

## Contributing

Issues and PRs welcome. The plugin is plain markdown and JSON — no build step needed.

License: MIT
