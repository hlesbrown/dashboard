# Dashboard API — Claude Project Instructions

You have access to Les Brown's project dashboard via a REST API hosted on Railway. Use this to read current project status and update tasks when work is completed or status changes.

## Connection Details

- **Base URL:** `https://dashboardapi-production.up.railway.app`
- **Auth Header:** `X-API-Key: hlbdashboard`
- **Format:** JSON

## Shared Files

Shared documents (like this one) are stored on GitHub and can be fetched by any Claude:

```
https://raw.githubusercontent.com/hlesbrown/dashboard/main/shared/FILENAME
```

Check the notice board for announcements when shared files are updated.

## Endpoints

### Read Data

**GET /dashboard** — Full dashboard (groups + tasks + notices). Use this first to see everything.
```
curl -H "X-API-Key: hlbdashboard" https://dashboardapi-production.up.railway.app/dashboard
```
Returns: `{ "groups": [...], "notices": [...] }`

**GET /groups** — List all project groups (without tasks).

**GET /tasks** — List all tasks. Optional filter: `?group=<group_id>`

**GET /tasks/<id>** — Get one task by ID.

**GET /logs** — Progress logs. Optional: `?group=<group_id>&limit=20`

### Write Data

**PUT /tasks/<id>** — Update a task. Send only the fields you want to change.
```
curl -X PUT -H "X-API-Key: hlbdashboard" -H "Content-Type: application/json" \
  -d '{"status": "New status text", "last_touched": "2026-02-13"}' \
  https://dashboardapi-production.up.railway.app/tasks/<id>
```

**POST /tasks** — Create a new task. Required fields: `group_id`, `name`.

**POST /logs** — Add a progress log entry. Required fields: `group_id`, `source`, `summary`. Optional: `details` (array of strings).
```
curl -X POST -H "X-API-Key: hlbdashboard" -H "Content-Type: application/json" \
  -d '{"group_id": "your_group", "source": "Claude-YourName", "summary": "What you did", "details": ["Detail 1", "Detail 2"]}' \
  https://dashboardapi-production.up.railway.app/logs
```

## Updatable Task Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Task name |
| `status` | string | Current status summary |
| `urgency` | string | One of: `urgent`, `active`, `recurring`, `low` |
| `last_touched` | string | Date string, e.g. `"2026-02-13"` |
| `deadline` | string or null | Fixed deadline, e.g. `"2026-02-25"` |
| `recurring_deadline_day` | integer or null | Day of week (0=Sunday, 6=Saturday) |
| `next_actions` | array of strings | Action items list |
| `notes` | string | Notes and reminders |
| `links` | array of objects | Each: `{"label": "...", "url": "..."}` |
| `group_id` | string | Move task to different group |

## Project Groups

| group_id | Name | sort_order |
|----------|------|------------|
| `today` | Due Today | -1 |
| `novelbase` | NovelBank | 0 |
| `podcast` | Podcast & Social Media | 1 |
| `sycamore` | Sycamore Boy | 2 |
| `author` | Book Marketing & Promotion | 3 |
| `gbchapel` | GB Chapel | 4 |
| `circle` | Jonathan's Circle | 5 |
| `misc` | Misc. | 6 |

## Due Today List

Claude-Dashboard posts a daily prioritized task list in the `today` group each morning. Other Claudes may interact with it:

**To add an item:**
1. `GET /tasks?group=today` to find the current day's task
2. Append your item to `next_actions` — use bracket notation: `"[YourProject] Description"`
3. `PUT /tasks/<id>` with the updated `next_actions` array

**To mark something done:**
1. Prepend ✅ to the action text (do not remove items — Les checks them off in the browser)
2. Example: `"✅ [NovelBank] Fix AI Studio and test"`

**Rules:**
- Always GET first so you don't overwrite someone else's additions
- Keep items specific and short
- The task ID changes daily — always fetch fresh
- Do not create new tasks in the `today` group — only Claude-Dashboard does that

## Progress Logs

Post a log entry at the end of each session to record what was accomplished:
```
curl -X POST -H "X-API-Key: hlbdashboard" -H "Content-Type: application/json" \
  -d '{"group_id": "your_group", "source": "Claude-YourName", "summary": "Brief summary", "details": ["Specific item 1", "Specific item 2"]}' \
  https://dashboardapi-production.up.railway.app/logs
```

## Current Task IDs (for reference)

| ID | Task | Group |
|----|------|-------|
| 1 | NovelBank Development | novelbase |
| 15 | NovelBank Outreach & Marketing | novelbase |
| 2 | Podcast — So You Want to Write a Novel? | podcast |
| 8 | Content Distribution & Social Media | podcast |
| 13 | Dev Edit Revision | sycamore |
| 16 | Line Edit & Polish | sycamore |
| 17 | Cover Design | sycamore |
| 18 | Publication Prep | sycamore |
| 3 | Author Website — hlesbrown.com | author |
| 14 | Marketing | author |
| 4 | Lenten Series | gbchapel |
| 6 | Weekly Liturgy & Homilies | gbchapel |
| 7 | CIRCLING Magazine | circle |
| 10 | Review All Subscriptions | misc |
| 11 | Repair Front Yard Lighting | misc |
| 19+ | Due Today (daily, ID changes) | today |

## When to Update the Dashboard

- **When you complete work** — update `status`, `last_touched`, and remove completed items from `next_actions`
- **When priorities change** — update `urgency` or `deadline`
- **When new action items are identified** — add to `next_actions`
- **When adding useful links** — add to `links` array
- **At end of session** — POST a log entry to `/logs`

## Important

- Always **GET the current task first** before updating, so you don't overwrite recent changes.
- Set `last_touched` to today's date whenever you update a task.
- When updating `next_actions`, send the full updated array (not just the ones to add/remove).
- The dashboard PWA at https://hlesbrown.github.io/dashboard/ shows this data live — Les can see your updates there.
- Check the **notice board** (in `/dashboard` response) for announcements from Claude-Dashboard.
- Check **shared files** on GitHub for updated documents: `https://raw.githubusercontent.com/hlesbrown/dashboard/main/shared/`
