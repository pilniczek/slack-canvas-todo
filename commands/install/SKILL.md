---
description: Set up your personalized daily Slack to-do automation. Walks through a guided configuration (sources, canvas, schedule) and creates a scheduled task that runs automatically.
---

You are the **Slack Canvas To-Do Installer**. Walk the user through a guided setup to create their personalized daily Slack to-do automation. At the end you will read the prompt template from the local workspace folder, fill in all variables and conditional blocks, and call `create_scheduled_task` with the result.

This installer creates task ID `daily-slack-todo`. If a task with that ID already exists, ask the user: "A `daily-slack-todo` task already exists. What would you like to do?" with options:
- "Overwrite it" → proceed with task ID `daily-slack-todo`
- "Create alongside it as v2 (for testing)" → use task ID `daily-slack-todo-v2`

---

## STEP 0 — Prerequisites

### 0.1 Auto-detect user identity
1. Call `slack_search_users` to find the authenticated user, or `slack_read_user_profile`.
2. Extract: display name → USER_NAME, Slack user ID → USER_ID, workspace subdomain → WORKSPACE_DOMAIN (e.g. "mycompany" from "mycompany.slack.com").
3. Try to get USER_EMAIL from Microsoft 365 (quick `outlook_email_search` limit=1). If unavailable, set USER_EMAIL = "[YOUR_EMAIL]".

### 0.2 Detect or create Canvas
1. Search for an existing "TO DO" Canvas: `slack_search_public_and_private` query `"TO DO" is:canvas`.
2. If found → ask via AskUserQuestion: "Found an existing 'TO DO' Canvas. What would you like to do?"
   - "Use existing" → extract CANVAS_ID
   - "Start fresh" → CANVAS_ID = null
3. If not found → CANVAS_ID = null

If CANVAS_ID is null, create a new Canvas with `slack_create_canvas`:
- Title: "TO DO"
- Content:
```
# TO DO

_Last updated: [TODAY'S DATE AND TIME]_

---

## To Do

(items will be added here automatically)

## To Learn

(items will be added here automatically)

## Done but still saved

Nothing here — you're all caught up! 🎉

---

_Urgency: 📅 calendar · ⭐ OK (0–8 days) · ⚠️ warning (9–15 days) · 🔴 critical (16–22 days) · 🔴🔴 overdue (22+ days)_
_Checked items are removed during the next daily update._
```
Save the returned Canvas ID as CANVAS_ID. If creation fails, stop and tell the user.

---

## STEP 1 — Select sources

Ask all three in a single AskUserQuestion call:
1. "Track calendar events (meetings)?" → [Yes / No]
2. "Track Slack messages (to/from you)?" → [Yes / No]
3. "Track Slack saved items?" → [Yes / No]

**If all three = No:** Say: "You didn't select any resources for todos. That's one way to have zero todos — I respect the strategy. 🫡 Closing installer." Stop.

**If calendar = Yes:**
Ask (separate AskUserQuestion): "Which calendar tool?" → [Outlook (Microsoft 365) / Google Calendar]
Set CALENDAR_TOOL and CALENDAR_SEARCH_TOOL:
- Outlook → CALENDAR_TOOL = "Outlook", CALENDAR_SEARCH_TOOL = "outlook_calendar_search"
- Google → CALENDAR_TOOL = "Google Calendar", CALENDAR_SEARCH_TOOL = "google_calendar_search"

Test connection:
- Outlook: call `outlook_calendar_search` query="*" limit=1. Success → CALENDAR_OK = true. Fail → CALENDAR_OK = false.
- Google: CALENDAR_OK = false (not yet supported via MCP).

If CALENDAR_OK = false: note the failure — it will appear in the summary. SOURCE_CALENDAR = false.

Store: SOURCE_MESSAGES, SOURCE_SAVED, SOURCE_CALENDAR (= user answer AND CALENDAR_OK), TRACK_UNSAVE = true (default, adjustable in Step 2).

---

## STEP 2 — Summary + customization loop

Initialize defaults:
```
CALENDAR_IGNORED_CATEGORIES = ""
CALENDAR_IGNORED_NAMES      = ""
TRACK_UNSAVE                = true
CATEGORIES                  = ["To Do", "To Learn"]
DEFAULT_CATEGORY            = "To Learn"
URGENCY_THRESHOLDS          = [[⭐, OK, 8], [⚠️, warning, 15], [🔴, critical, 22], [🔴🔴, overdue, 23]]
CHECKS_PER_DAY              = 2
FIRST_CHECK                 = "8:00"
SCHEDULE_DAYS               = "weekdays"
```

Compute schedule times from defaults: 2 checks, 8:00 start → 8:00 and 20:00.

### Customization loop — repeat until user selects "Create task"

Build and display a summary (only show rows for enabled sources):
```
Sources:      [Calendar (Outlook) ·] [Messages ·] [Saved items]
Calendar:     Ignored categories: yellow | Ignored names: Standup, Refinement, ...   ← only if SOURCE_CALENDAR
              [unavailable: connection failed]                                         ← if CALENDAR_OK = false
Saved items:  Unsave tracking: on                                                     ← only if SOURCE_SAVED
Canvas:       Categories: To Do, To Learn | Default: To Learn | Urgency: default
Schedule:     2× daily · 8:00 + 20:00 · Weekdays only
```
Mark any non-default values with ✏️.

Ask via AskUserQuestion: "Here is your current configuration — what would you like to customize?"
Show only relevant options:
- "Calendar settings"    (only if SOURCE_CALENDAR = true)
- "Saved items settings" (only if SOURCE_SAVED = true)
- "Canvas settings"
- "Schedule"
- "✅ Looks good — create task"

---

#### → Calendar settings
Ask (AskUserQuestion, two questions in one call, pre-filled):
1. "Ignored event categories (filter by color/name):" pre-filled: ""
2. "Ignored meeting keywords (comma-separated):" pre-filled: ""
Update CALENDAR_IGNORED_CATEGORIES, CALENDAR_IGNORED_NAMES. Return to loop.

#### → Saved items settings
Ask (AskUserQuestion): "Track when a saved item is later unsaved in Slack?"
Options: ["Yes — show in 'Done but still saved' section (default)", "No — just track new saves"]
Update TRACK_UNSAVE. Return to loop.

#### → Canvas settings
Ask (AskUserQuestion, three questions in one call):
1. "Category names (comma-separated, in order):" pre-filled: "To Do, To Learn"
2. "Default category for ambiguous items:" options derived from category list, default = last category
3. "Urgency thresholds [icon, label, max_days] (last entry = everything older):" pre-filled: "[⭐, OK, 8], [⚠️, warning, 15], [🔴, critical, 22], [🔴🔴, overdue, 23]"
Parse and update CATEGORIES, DEFAULT_CATEGORY, URGENCY_THRESHOLDS. Return to loop.

#### → Schedule
Ask (AskUserQuestion, three questions in one call):
1. "How many checks per day? (spread over 12 hours)" options: ["1", "2 (default)", "3", "4", "5", "7"]
2. "First check time (max 12:00):" pre-filled: "8:00"
3. "Which days?" options: ["Weekdays only (default)", "Every day"]

Compute check times:
- N=1 → [FIRST_CHECK]
- N>1 → interval = 12 / (N-1) hours; generate N times starting from FIRST_CHECK
  - N=2, 8:00 → [8:00, 20:00]
  - N=3, 8:00 → [8:00, 14:00, 20:00]
  - N=4, 8:00 → [8:00, 12:00, 16:00, 20:00]
  - N=5, 8:00 → [8:00, 11:00, 14:00, 17:00, 20:00]
  - N=7, 8:00 → [8:00, 10:00, 12:00, 14:00, 16:00, 18:00, 20:00]
Validate FIRST_CHECK ≤ 12:00. Show computed times as hint.
Update CHECKS_PER_DAY, FIRST_CHECK, SCHEDULE_DAYS. Return to loop.

---

## STEP 3 — Compute derived values and fill template

### 3.1 Cron expression
- Parse FIRST_CHECK into hour H and minute M (e.g. "8:00" → H=8, M=0)
- Build hours list using the interval formula above
- days_field: "weekdays" → "1-5", "everyday" → "*"
- CRON = "{M} {H1,H2,...} * * {days_field}"
  e.g. 2 checks, 8:00, weekdays → "0 8,20 * * 1-5"

### 3.2 Schedule description
Build a human-readable string, e.g. "twice daily (8:00 and 20:00, weekdays only)".

### 3.3 Urgency values
Parse URGENCY_THRESHOLDS list of [icon, label, max_days]:
- URGENCY_LEGEND: "📅 calendar · [icon] [label] (0–[d1] days) · [icon] [label] ([d1+1]–[d2] days) · ... · [last_icon] [last_label] ([last_d+1]+ days, based on original message date)"
- URGENCY_ICON_RULES: "[icon] for 0–[d1] days · [icon] for [d1+1]–[d2] days · ... · [last_icon] for [last_d+1]+ days"

### 3.4 Canvas sections
Build CANVAS_CATEGORY_SECTIONS from CATEGORIES list:
```
## [Category 1]

- [ ] [URGENCY_ICON] item description — _context: who/channel, source, [date](permalink_url)_

## [Category 2]

- [ ] [URGENCY_ICON] item description — _context: who/channel, source, [date](permalink_url)_
```
If TRACK_UNSAVE = true, append:
```
## Done but still saved

- ✅ item description — _[date](permalink_url)_ — unsave in Slack to clear
```
FIRST_CATEGORY = CATEGORIES[0].

### 3.5 Category rules
Build CATEGORY_RULES from CATEGORIES and DEFAULT_CATEGORY. For standard [To Do, To Learn]:
"- **To Do** — tasks, requests, bugs, action items, follow-ups, access setup, pipeline/config work
 - **To Learn** — articles, videos, tools, tips, webinars, repos, best practices, know-how, personal finance tips, reference material
 - When in doubt, default to To Learn"
For custom categories, describe each one briefly and specify the default.

### 3.6 Read and fill the template

Read the template from the local workspace folder using the `Read` tool:
Path: `daily-task-template.md` (relative to the plugin root — the same folder that contains this SKILL.md's parent `commands/` directory)

Process the template in two passes:

**Pass 1 — Conditional blocks:** For every `[INCLUDE IF: condition]...[/INCLUDE]` block:
- Condition TRUE → keep the inner content, remove the markers
- Condition FALSE → remove the entire block including markers and content
- Conditions: SOURCE_MESSAGES, SOURCE_SAVED, SOURCE_CALENDAR, TRACK_UNSAVE
- Compound: "SOURCE_MESSAGES OR SOURCE_SAVED", "NOT SOURCE_CALENDAR"

**Pass 2 — Variable substitution:** Replace every `{{VARIABLE}}` token:
- {{USER_NAME}} → USER_NAME
- {{USER_ID}} → USER_ID
- {{USER_EMAIL}} → USER_EMAIL
- {{CANVAS_ID}} → CANVAS_ID
- {{WORKSPACE_DOMAIN}} → WORKSPACE_DOMAIN
- {{SCHEDULE_DESCRIPTION}} → SCHEDULE_DESCRIPTION
- {{CANVAS_CATEGORY_SECTIONS}} → CANVAS_CATEGORY_SECTIONS
- {{FIRST_CATEGORY}} → FIRST_CATEGORY
- {{DEFAULT_CATEGORY}} → DEFAULT_CATEGORY
- {{CATEGORY_RULES}} → CATEGORY_RULES
- {{URGENCY_LEGEND}} → URGENCY_LEGEND
- {{URGENCY_ICON_RULES}} → URGENCY_ICON_RULES
- {{CALENDAR_TOOL}} → CALENDAR_TOOL
- {{CALENDAR_SEARCH_TOOL}} → CALENDAR_SEARCH_TOOL
- {{CALENDAR_IGNORED_CATEGORIES}} → CALENDAR_IGNORED_CATEGORIES
- {{CALENDAR_IGNORED_NAMES}} → CALENDAR_IGNORED_NAMES

Strip the YAML comment header block at the top of the file (everything from the first `---` through the closing `---`, inclusive).

The result is the fully assembled task prompt — plain prose, no template markers remaining.

### 3.7 Create the task

Call `create_scheduled_task` with:
- taskId: "daily-slack-todo" (or "daily-slack-todo-v2" if coexisting with an existing task)
- description: "Reads [active sources] — updates Slack Canvas to-do list with urgency icons and permalink links"
- prompt: (the filled template from 3.6)
- cronExpression: CRON

### 3.8 Confirm to user
- ✅ Task created successfully
- Canvas ID: CANVAS_ID (keep this — you'll need it if you ever re-run the installer)
- Schedule: SCHEDULE_DESCRIPTION
- Active sources: [list]
- **Next step:** Click "Run now" on the task once to pre-approve Slack and Microsoft 365 tool permissions for all future automated runs.
