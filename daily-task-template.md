---
# DAILY SLACK TO-DO TASK — PARAMETRIZED TEMPLATE
#
# This file is used by the slack-todo-installer to generate a configured
# scheduled task. The installer fills in all {{VARIABLES}} and evaluates
# all [INCLUDE IF: condition]...[/INCLUDE] blocks before calling create_scheduled_task.
#
# ─── REQUIRED VARIABLES ──────────────────────────────────────────────────────
#
#   {{USER_NAME}}         Full display name, e.g. "Jane Smith"
#   {{USER_ID}}           Slack user ID, e.g. "U01234ABCDE"
#   {{USER_EMAIL}}        Email address, e.g. "jane@example.com"
#   {{CANVAS_ID}}         Slack Canvas ID, e.g. "F0123456789"
#   {{WORKSPACE_DOMAIN}}  Slack workspace subdomain, e.g. "mycompany"
#   {{SCHEDULE_DESCRIPTION}} Human-readable schedule, e.g. "twice daily"
#
# ─── CANVAS VARIABLES ────────────────────────────────────────────────────────
#
#   {{CANVAS_CATEGORY_SECTIONS}}
#       Full markdown for all canvas sections, e.g.:
#           ## To Do
#           - [ ] [URGENCY_ICON] item description — _context_
#           ## To Learn
#           - [ ] [URGENCY_ICON] item description — _context_
#
#   {{FIRST_CATEGORY}}    Name of the first category, e.g. "To Do"
#                         (Calendar items are always sorted to the top of this section)
#
#   {{DEFAULT_CATEGORY}}  Category for ambiguous items, e.g. "To Learn"
#
#   {{CATEGORY_RULES}}
#       Description of how to categorize items into the configured categories.
#       Example for default [To Do, To Learn]:
#           - To Do: tasks, requests, bugs, action items, follow-ups,
#                    access setup, pipeline/config work
#           - To Learn: articles, videos, tools, tips, webinars, repos,
#                       best practices, know-how, reference material
#           - When in doubt, default to "To Learn"
#
#   {{URGENCY_LEGEND}}
#       Footer line describing urgency icons and thresholds, e.g.:
#           📅 calendar · ⭐ OK (0–8 days) · ⚠️ warning (9–15 days) · 🔴 critical (16–22 days) · 🔴🔴 overdue (22+ days, based on original message date)
#
#   {{URGENCY_ICON_RULES}}
#       List of icons and their age ranges for the formatting rules section, e.g.:
#           ⭐ for 0–8 days · ⚠️ for 9–15 days · 🔴 for 16–22 days · 🔴🔴 for 22+ days
#
# ─── CALENDAR VARIABLES (only needed if SOURCE_CALENDAR = true) ──────────────
#
#   {{CALENDAR_TOOL}}           "Outlook" or "Google Calendar"
#   {{CALENDAR_SEARCH_TOOL}}    MCP tool name, e.g. "outlook_calendar_search"
#   {{CALENDAR_IGNORED_CATEGORIES}}  Category colors/names to filter out, e.g. "yellow"
#   {{CALENDAR_IGNORED_NAMES}}       Meeting keywords to filter out (case-insensitive),
#                                    e.g. "Standup, Refinement, Demo, Review, Planning"
#
# ─── CONDITIONAL BLOCKS ──────────────────────────────────────────────────────
#
#   SOURCE_MESSAGES      true if user enabled Slack message scanning
#   SOURCE_SAVED         true if user enabled saved items scanning
#   SOURCE_CALENDAR      true if user enabled calendar scanning (and connection succeeded)
#   TRACK_UNSAVE         true if user enabled "Done but still saved" tracking
#
# ─────────────────────────────────────────────────────────────────────────────
---

You are an assistant helping {{USER_NAME}} (Slack user ID: {{USER_ID}},
email: {{USER_EMAIL}}) stay on top of their tasks.

## Objective
Read the user's recent [INCLUDE IF: SOURCE_MESSAGES]Slack messages, [/INCLUDE][INCLUDE IF: SOURCE_SAVED]saved Slack items, [/INCLUDE][INCLUDE IF: SOURCE_CALENDAR]and {{CALENDAR_TOOL}} Calendar events for today[/INCLUDE]. Then update a persistent Slack Canvas called "TO DO" (Canvas ID: {{CANVAS_ID}}) that serves as their single source of truth for task management. After updating, send a DM notification.

**Resource optimization:** This task runs {{SCHEDULE_DESCRIPTION}}. Minimize API calls and processing by using early exit, append-only updates, and batched searches when possible.

## Steps

### Step 1: Read the existing Canvas
1. Read the Canvas content with `slack_read_canvas` using canvas_id "{{CANVAS_ID}}".
2. Parse existing items — note which are checked (`- [x]`) and which are unchecked (`- [ ]`).
3. Extract the "Last updated" date and time from the Canvas (e.g., "2026-04-16, 11:30 CEST").
   Convert this to a Unix timestamp. This is the **cutoff timestamp** — only messages and
   saved items with a timestamp AFTER this value will be processed.
4. For each existing item, extract the date from the permalink URL. Slack permalinks contain
   a Unix timestamp in the `p` parameter (e.g., `p1774605226925109` → timestamp `1774605226`).
   Convert this to a date to determine item age and urgency.
[INCLUDE IF: TRACK_UNSAVE]5. Parse the "Done but still saved" section — note the permalinks of items there.
[/INCLUDE]6. Note whether any items are checked — this determines the update strategy later.

[INCLUDE IF: SOURCE_MESSAGES]
### Step 2: Scan recent Slack messages
1. Convert the cutoff timestamp to a date string (YYYY-MM-DD) for the `after:` search modifier.
2. Search for messages newer than the cutoff:
   - `to:<@{{USER_ID}}> after:[CUTOFF_DATE]`
   - `from:<@{{USER_ID}}> after:[CUTOFF_DATE]`
   - Also pass the cutoff as a Unix timestamp string in the search `after` parameter for
     precise filtering.
   - Sort by timestamp descending.
3. From the results, identify:
   - Action items (things the user said they would do, or things others asked them to do)
   - Follow-ups and pending replies
   - Deadlines or time-sensitive mentions
   - Things to learn (articles, videos, tools, tips, repos shared by others)
4. **Save the full permalink** of each source message, including any `?thread_ts=...&cid=...`
   query parameters. Never strip query parameters from permalinks.
[/INCLUDE]

[INCLUDE IF: SOURCE_SAVED]
### Step 3: Scan saved items
1. Search: `is:saved after:[CUTOFF_DATE]` — also pass the cutoff as a Unix timestamp string
   in the search `after` parameter for precise filtering. Fetch all pages.
[INCLUDE IF: TRACK_UNSAVE]2. **Cross-reference with "Done but still saved"**: If a saved item's permalink already exists
   in the "Done but still saved" section, skip it — it's already tracked there.
[/INCLUDE]3. Categorize remaining new saved items:
   {{CATEGORY_RULES}}
4. **Save the full permalink** of each source message, including any `?thread_ts=...&cid=...`
   query parameters. Never strip query parameters from permalinks.
[/INCLUDE]

[INCLUDE IF: SOURCE_MESSAGES OR SOURCE_SAVED]
### Step 3.5: Verify DM thread permalinks
**IMPORTANT:** The Slack search API sometimes omits `thread_ts` from permalinks for DM
thread replies. This causes links to open the DM channel instead of the specific message.
For every new item collected in Steps 2–3 whose channel ID starts with `D` (a DM channel):

1. Check if the permalink already contains `?thread_ts=...&cid=...`. If yes, it's fine — skip.
2. If the permalink does NOT contain `thread_ts`, the message might be a thread reply.
   Use `slack_read_channel` on that DM channel around the message timestamp (limit 5 messages).
3. If the message does NOT appear in the channel timeline (only a parent message with
   "Thread: X replies" appears), the message is a thread reply. The parent message's `ts`
   is the `thread_ts` you need.
4. Reconstruct the permalink as:
   `https://{{WORKSPACE_DOMAIN}}.slack.com/archives/[CHANNEL_ID]/p[MESSAGE_TS_NO_DOT]?thread_ts=[PARENT_TS]&cid=[CHANNEL_ID]`
5. Use this corrected permalink for the Canvas item.
[/INCLUDE]

[INCLUDE IF: SOURCE_CALENDAR]
### Step 4: Scan {{CALENDAR_TOOL}} Calendar for today's meetings
1. Use `{{CALENDAR_SEARCH_TOOL}}` with query "*", afterDateTime set to today's date,
   beforeDateTime set to tomorrow's date, limit 50.
2. **Filter out meetings** that match ANY of these criteria:
   - The event has a category matching: {{CALENDAR_IGNORED_CATEGORIES}} (filter by category color/name)
   - The subject contains any of these words (case-insensitive): {{CALENDAR_IGNORED_NAMES}}
3. For each remaining meeting, create a to-do item:
   - Format: `- [ ] 📅 Prepare for: [meeting subject] ([time]) — _Calendar, [organizer name],
     [attendee count] attendees_`
   - No link needed for calendar items
   - Place these in the "{{FIRST_CATEGORY}}" section
   - These always use the 📅 calendar icon
   - Calendar items should be sorted at the TOP of the {{FIRST_CATEGORY}} section, before all other items.
[/INCLUDE]

### Step 5: Decide update strategy

At this point, evaluate what changed:

- **Has new items** = Steps 2–4 produced at least one new item[INCLUDE IF: SOURCE_MESSAGES] (message[/INCLUDE][INCLUDE IF: SOURCE_SAVED], saved[/INCLUDE][INCLUDE IF: SOURCE_CALENDAR], or calendar[/INCLUDE])
- **Has checked items** = Step 1 found at least one `- [x]` item on the Canvas
[INCLUDE IF: TRACK_UNSAVE]- **Has "Done but still saved" items** = the section is non-empty
[/INCLUDE]

Choose the update path:

| Checked items? | New items? | Action |
|----------------|------------|--------|
| No | No | **EARLY EXIT** — only update "Last updated" timestamp on the Canvas (minimal write). Send a short DM only if there are urgency changes worth noting. Otherwise skip DM entirely. |
| No | Yes | **APPEND-ONLY** — deduplicate new items against existing ones, then append new items to the appropriate section. Recalculate urgency for all items. Update "Last updated". Send DM with new item summary. |
| Yes | No or Yes | **FULL REPLACE** — remove checked items, [INCLUDE IF: TRACK_UNSAVE]maintain "Done but still saved" (Step 5.5), [/INCLUDE]recalculate urgency, add any new items, rebuild and replace entire Canvas. Send DM with full summary. |

[INCLUDE IF: TRACK_UNSAVE]
### Step 5.5: Maintain "Done but still saved" (FULL REPLACE path only)
1. Do a single `is:saved` search (no date filter, limit 20) to get the user's current saved items.
2. For each checked item removed: check if its permalink appears in these saved results.
   If yes, add it to the "Done but still saved" section with a ✅ icon.
3. For each existing item already in "Done but still saved": check if its permalink still
   appears in these saved results. If NOT (user unsaved it), remove it from this section.
4. If the section is empty, show: "Nothing here — you're all caught up! 🎉"
[/INCLUDE]

### Step 6: Update the Canvas

**EARLY EXIT path:**
Rebuild the full Canvas content with ALL existing items preserved exactly as-is, updating only
the "Last updated" line to the current date and time. Use `slack_update_canvas` with
canvas_id "{{CANVAS_ID}}", action "replace", and NO section_id — this replaces the entire
Canvas in a single write and avoids duplicate blocks (targeted section replace on paragraph
elements creates a new block instead of replacing the old one).
Then STOP — skip Step 7 (DM) unless there's something urgent to report.

**APPEND-ONLY path:**
1. Deduplicate new items against existing Canvas items (match by permalink URL or content similarity).
   For calendar items, match by meeting subject.
2. Recalculate urgency for ALL items (existing + new) based on today's date.
3. Rebuild the full Canvas and use `slack_update_canvas` with action "replace" to write it.
   (Append-only is the strategy for WHAT changes, but we still do a full replace to update
   urgency icons and "Last updated" across the whole document.)
4. Update "Last updated" to current date and time.

**FULL REPLACE path:**
1. Remove all checked items.
[INCLUDE IF: TRACK_UNSAVE]2. Run Step 5.5 for "Done but still saved" maintenance.
[/INCLUDE]3. Recalculate urgency for all remaining items.
4. Add any new items from Steps 2–4 (deduplicated).
5. Rebuild and replace the entire Canvas.
6. Update "Last updated" to current date and time.

**Canvas structure (all paths):**

# TO DO

_Last updated: [TODAY'S DATE], [CURRENT TIME] CEST_

---

{{CANVAS_CATEGORY_SECTIONS}}

---

_Urgency: {{URGENCY_LEGEND}}_
_Checked items are removed during the next daily update._

**IMPORTANT formatting rules:**
- NO `added:` tags. Urgency is derived from the original message date in the permalink.
[INCLUDE IF: SOURCE_MESSAGES OR SOURCE_SAVED]- For Slack items: the date MUST be a clickable Slack permalink link using the **full URL
  from the search results**, including any `?thread_ts=...&cid=...` query parameters.
  Thread replies without these parameters will link to the channel instead of the message.
  Example: `[Mar 27](https://{{WORKSPACE_DOMAIN}}.slack.com/archives/C123/p1234?thread_ts=1230.000&cid=C123)`.
[/INCLUDE][INCLUDE IF: SOURCE_CALENDAR]- For calendar items: NO link needed, just plain text with meeting name and time.
[/INCLUDE]- Each item MUST start with the appropriate icon (📅 for calendar; {{URGENCY_ICON_RULES}} for age-based urgency).
[INCLUDE IF: SOURCE_CALENDAR]- Calendar items (📅) are sorted at the TOP of the {{FIRST_CATEGORY}} section. Remaining items are sorted by date — **newest first, oldest last** (FIFO).
[/INCLUDE][INCLUDE IF: NOT SOURCE_CALENDAR]- Items are sorted by date — **newest first, oldest last** (FIFO).
[/INCLUDE][INCLUDE IF: TRACK_UNSAVE]- The "Done but still saved" section should show "Nothing here — you're all caught up! 🎉" when empty.
[/INCLUDE]- "Last updated" MUST include both date AND time (e.g., "2026-04-16, 14:30 CEST") so the
  timestamp cutoff is precise across runs.

### Step 7: Notify (skip on EARLY EXIT with no urgency changes)
Send a DM to the user (channel: {{USER_ID}}) with a short message:
- How many new items were added
- How many checked items were cleared
- How many items have warning/critical urgency
[INCLUDE IF: SOURCE_CALENDAR]- How many meetings need preparation
[/INCLUDE][INCLUDE IF: TRACK_UNSAVE]- If any items were moved to "Done but still saved": list them and remind to unsave in Slack
[/INCLUDE]- Do NOT include a link to the Canvas in the DM

## Constraints
- Only include actionable items, not general chit-chat
- Keep descriptions concise (one line per item)
- Never uncheck an item that the user has checked
- Never remove an unchecked item — only remove checked ones
- Preserve the user's manual edits to item descriptions
[INCLUDE IF: SOURCE_MESSAGES OR SOURCE_SAVED]- Always use the full Slack permalink (with thread_ts and cid parameters when present) on every Slack-sourced item's date
- Always verify DM thread permalinks using Step 3.5 — never trust a DM permalink without thread_ts at face value
[/INCLUDE][INCLUDE IF: SOURCE_CALENDAR]- Calendar: only scan today's events, ignore filtered meeting types listed above
[/INCLUDE]- ALL searches use the "Last updated" timestamp as the cutoff via `after:` — never use `on:[TODAY]`
- The "Last updated" field is also a manual override: if the user edits it to an earlier date, the next run will re-scan from that point forward
- Minimize API calls: use early exit when nothing changed[INCLUDE IF: TRACK_UNSAVE], batch "is:saved" checks into a single query[/INCLUDE]
