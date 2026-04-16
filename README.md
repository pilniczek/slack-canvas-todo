# slack-canvas-todo

A Cowork plugin that keeps a persistent **Slack Canvas** up to date as your daily to-do list — automatically, on a schedule you choose.

---

## What it does

- Scans **Slack messages** (to/from you) and **saved items** for action items and things to learn
- Pulls **calendar events** (Outlook via Microsoft 365) and adds meeting prep reminders; automatically removes them once the meeting time has passed — even if you never checked them off
- Scans your **Outlook inbox** (and optional extra folders) for emails that require action — AI classifies each one so only genuinely actionable emails land on the list, not your whole inbox
- Writes everything to a **Slack Canvas** — Slack items get urgency icons that age over time (⭐ → ⚠️ → 🔴 → 🔴🔴), calendar items use 📅, and email items use 📧; neither calendar nor email items carry age-based urgency
- Tracks **"Done but still saved"** — items you checked off but haven't unsaved yet
- Runs on a **cron schedule** you configure (default: twice daily, weekdays)
- Uses **early exit and append-only paths** to minimise API calls when nothing changed

---

## Requirements

- Claude Cowork (paid plan)
- **Slack** connector installed in Cowork
- **Microsoft 365** connector (optional — needed for Outlook Calendar and/or Outlook email)

---

## Installation

### 1. Get the repo

**Option A — clone:**
```bash
git clone https://github.com/pilniczek/slack-canvas-todo.git ~/Documents/Claude/Projects/slack-canvas-todo
```

**Option B — download:** click **Code → Download ZIP** on GitHub, extract it, and place the `slack-canvas-todo` folder in your preferred projects directory.

### 2. Link the folder as a Cowork project

Open **Claude Cowork**, go to **Projects → New Project**, and select the cloned `slack-canvas-todo` folder. This properly links the folder so the plugin and its skills are available in your session.

### 3. Run the installer

Type in the Cowork chat:

```
install slack-canvas-todo
```

### 4. Follow the guided prompts

The installer walks you through everything interactively:

1. Auto-detects your Slack identity (name, user ID, workspace)
2. Finds or creates your **TO DO** Canvas
3. Asks which sources to enable (messages / saved items / calendar / email)
4. For email: auto-detects your inbox folder name and asks about any extra folders to include
5. Shows a summary with all defaults pre-filled — customise only what you want
6. Creates a scheduled task that runs automatically

### 5. Approve permissions on first run

After setup, click **Run now** once on the created task to pre-approve Slack and Microsoft 365 tool permissions for all future automated runs.

---

## Configuration

All settings are collected interactively during install. You can customise:

| Setting | Default |
|---|---|
| Sources | Messages + Saved items + Calendar + Emails |
| Calendar tool | Outlook (Microsoft 365 connector) |
| Ignored calendar categories | _(none)_ |
| Ignored meeting keywords | _(none)_ |
| Email inbox | Auto-detected from your Outlook locale |
| Extra email folders | _(none — inbox only)_ |
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

- [ ] 📅 Prepare for: Product Sync (Apr 16, 14:00) — _Calendar, Alice Brown, 5 attendees_
- [ ] ⚠️ Fix login bug — review ticket and test locally — _Bob K., #dev, [Mar 30](https://...)_
- [ ] 📧 Re: Feature Hub — accessories export — _ondrej.pelouch@avenga.com, email, [Apr 10](https://outlook.office365.com/...)_
- [ ] ⭐ Log time for this week — _DM, [Apr 16](https://...)_

## To Learn

- [ ] ⭐ New feature demo — watch recording — _Carol M., #general, [Apr 1](https://...)_

## Done but still saved

- ✅ Check API response format — _[Dec 16](https://...)_ — unsave in Slack to clear

## You prepared for

Nothing yet — check off a meeting prep item to see it here.

---

_Urgency: 📧 email · 📅 calendar · ⭐ OK (0–8 days) · ⚠️ warning (9–15 days) · 🔴 critical (16–22 days) · 🔴🔴 overdue (23+ days, based on original message date)_
_Checked items are removed during the next daily update. Past calendar events (📅) are removed automatically once their meeting time has passed, even if unchecked._
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
├── CLAUDE.md                    # Codebase instructions for Cowork
├── daily-task-template.md       # Parametrized prompt — filled in by the installer
├── .mcp.json                    # Required connector declarations
├── LICENSE
└── README.md
```

**`daily-task-template.md`** is the source of truth for the scheduled task prompt. The installer reads it from your local workspace folder, fills in all `{{VARIABLES}}` and evaluates `[INCLUDE IF: ...]` blocks based on your choices, then registers the result as a scheduled task. To change the prompt logic, edit this file and re-run the installer.

---

## Security

The plugin reads content from Slack messages, saved items, calendar events, and email — all of which are untrusted input. The task prompt includes a hardcoded blocklist of known prompt-injection patterns (e.g. "ignore previous instructions", "SYSTEM:") and treats all fetched content as opaque data, never as instructions.

**Residual risk:** this is defence-in-depth, not a hard filter. A crafted message using obfuscation, encoding tricks, or a foreign language could still slip through. There is no perfect fix for this at the prompt level — it is an inherent limitation of any LLM-based automation that reads untrusted content. The most effective mitigation is setting your Canvas to **only you can edit** (prompted during installation), which prevents external parties from injecting content directly into the Canvas that the task reads on every run.

---

## Contributing

Issues and PRs welcome. The plugin is plain markdown and JSON — no build step needed.

License: MIT