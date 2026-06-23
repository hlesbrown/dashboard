# Claude Handbook — Permanent Notes

> ## ⚠️ TIMEZONE — READ THIS FIRST
> **Claude's servers run UTC. Les is in Palm Springs, California — Pacific Time (America/Los_Angeles).**
> Pacific is UTC−7 (PDT summer) or UTC−8 (PST winter).
> **Always run `TZ=America/Los_Angeles date` at session start. State the correct local day, date, and time before doing anything else.**
> Getting the date wrong disrupts scheduling. A UTC "Tuesday morning" may be "Monday evening" in Palm Springs. Never assume — always verify.

This is the shared reference document for all Claudes working on Les Brown's projects. Fetch this at the start of every session.

```bash
curl -s https://raw.githubusercontent.com/hlesbrown/dashboard/main/shared/CLAUDE_NOTES.md
```

---

## 1. Dashboard API — Quick Start

- **Base URL:** `https://dashboardapi-production.up.railway.app`
- **Auth:** `X-API-Key: hlbdashboard`
- **Full API docs:** Fetch `DASHBOARD_API_INSTRUCTIONS.md` from the shared folder (same GitHub location as this file)

**First thing every session:**
```bash
curl -s -H "X-API-Key: hlbdashboard" https://dashboardapi-production.up.railway.app/dashboard
```
This gives you all groups, tasks, notices, and calendar events.

---

## 2. Shared Files on GitHub

All Claudes can read shared project files via:

```
https://raw.githubusercontent.com/hlesbrown/dashboard/main/shared/FILENAME
```

**Currently available:**
- `DASHBOARD_API_INSTRUCTIONS.md` — Full API reference, endpoints, field definitions, Due Today protocol
- `CLAUDE_NOTES.md` — This file (permanent handbook)
- `PROTOCOL-INDEX.md` — Index of all protocol documents across projects

Check the notice board for announcements when new files are added or existing ones are updated.

---

## 3. Who's Who

| Claude | Responsibility | Project Group(s) |
|--------|---------------|-------------------|
| **Claude-Dashboard** | Dashboard maintenance, daily planning, Due Today list, cross-project coordination | `today`, all groups |
| **Claude-WB** | WorksBench app development (Flask/PostgreSQL on Railway) | `novelbase` |
| **Claude-Podcast** | Podcast production, social media, content distribution | `podcast` |
| **Claude-Editor** | Sycamore Boy dev editing and publication pipeline | `sycamore` |
| **Claude-BookMarketing** | Book marketing, hlesbrown.com, author platform | `author` |
| **Claude-Liturgy** | GB Chapel liturgy, homilies, newsletter | `gbchapel` |
| **Claude-JC** | Jonathan's Circle, CIRCLING magazine | `circle` |
| **Claude-Bondings** | Bondings novel development | `booksix` |
| **Claude-Docent** | Palm Springs Art Museum docent training app | `docent` |
| **Claude-Cowork** | Cross-project coordination, file management, operations | `cowork` |
| **Claude-Interface** | Avatar/interface experiments | `interface` |

Not all Claudes may be active at any given time. New Claudes may be added. When in doubt, check the notice board.

---

## 4. Session Protocol — MANDATORY

### Starting a session (required sequence):

**STEP 0 — Verify the local date and time FIRST:**
```bash
TZ=America/Los_Angeles date
```
State the result (e.g. "Tuesday, June 23, 2026 — 9:30 AM PDT") before doing anything else. Do not proceed until you have confirmed Pacific time. UTC is not acceptable — it will produce wrong dates for Les's scheduling.

1. **Fetch this handbook** (if not already in your project knowledge).
2. **Sync the calendar:** `POST /calendar/sync?days=21`
3. **Fetch the dashboard:** `GET /dashboard` — read ALL groups, tasks, and notices, not just your own.
4. **Scan your own tasks for stale content.** Before doing anything else, check every `next_actions` and `status` field under your group. If items are completed, clear them. If status is outdated, update it. Do this BEFORE reporting to Les.
5. **Check the notice board** for messages from other Claudes.
6. Do your work.

### Ending a session — NON-NEGOTIABLE

**A session that ends without these steps is an incomplete session.** Every session MUST finish with all of the following:

---

**STEP 1 — Update every task you touched:**

```bash
curl -X PUT -H "X-API-Key: hlbdashboard" -H "Content-Type: application/json" \
  -d '{
    "status": "Current accurate description of state",
    "next_actions": ["Only items not yet done"],
    "last_touched": "YYYY-MM-DD"
  }' \
  https://dashboardapi-production.up.railway.app/tasks/<id>
```

Rules:
- Set `last_touched` to today's **Pacific** date — every time, no exceptions
- `status` must reflect the **current** state, not last session's state
- `next_actions` must contain **only items not yet done** — remove completed items
- If a task is fully complete: set `urgency` to `low`, clear `next_actions`, update `status` to `COMPLETE`
- Never leave ✅-prefixed items sitting in `next_actions` — they've been done; remove them

---

**STEP 2 — Post an EOD log:**

```bash
curl -X POST -H "X-API-Key: hlbdashboard" -H "Content-Type: application/json" \
  -d '{
    "group_id": "your_group_id",
    "source": "Claude-YourName",
    "summary": "EOD YYYY-MM-DD — one-line description of session",
    "details": [
      "STATE: current version or project state",
      "COMPLETED: what was finished this session",
      "IN PROGRESS: what is underway but not done",
      "NEXT SESSION: recommended starting point next time",
      "BLOCKERS: anything blocking progress, or None"
    ]
  }' \
  https://dashboardapi-production.up.railway.app/logs
```

The `details` array is required and must follow this structure. A log with no details is not useful to Les or other Claudes.

---

**STEP 3 — Post a notice if other Claudes need to know:**

Ask yourself before closing:
- Did my work affect shared infrastructure (Railway, Dropbox, API keys)?
- Did I complete something that unblocks another Claude?
- Did I discover something relevant to another project?
- Did I create or modify a shared file in Dropbox?

If yes to any of these, post a notice:

```bash
curl -X POST -H "X-API-Key: hlbdashboard" -H "Content-Type: application/json" \
  -d '{
    "message": "Clear description of what other Claudes need to know",
    "priority": "fyi|normal|important",
    "author": "Claude-YourName"
  }' \
  https://dashboardapi-production.up.railway.app/notices
```

---

## 5. Dashboard Accountability — Staleness Rules

Claude-Dashboard reviews all task `last_touched` dates during every morning refresh. The following rules apply:

- **7 days without update:** Claude-Dashboard will flag the task in the morning report to Les.
- **14 days without update:** Claude-Dashboard will flag it prominently and post a notice directed at the owning Claude.
- **Recurring tasks** (podcast, liturgy): must show a `last_touched` within the current cycle. A podcast task last touched a month ago is stale even if it's recurring.

**What counts as an update:** Any session where you do substantive work must result in a `PUT /tasks/<id>` with today's `last_touched`. Reading the dashboard and doing nothing does not count.

**The right attitude:** Keeping your tasks current takes 5 minutes at session end. Les sees this dashboard live. Stale tasks create confusion and erode trust in the system. It is part of your job.

---

## 6. Due Today List

Claude-Dashboard posts a daily prioritized task list in the `today` group each morning.

**Rules for other Claudes:**
- You may add items using bracket notation: `"[YourProject] Description"`
- Always `GET /tasks?group=today` first so you don't overwrite others' additions
- To mark done, prepend ✅ to the action text — don't remove items (Les checks them off in the browser)
- Do NOT create new tasks in the `today` group — only Claude-Dashboard does that
- The task ID changes daily — always fetch fresh

---

## 7. Notice Board Etiquette

- Notices are for **transient announcements** — new files, API changes, task reassignments, session handoffs, cross-Claude alerts
- Permanent info belongs in this handbook or in `DASHBOARD_API_INSTRUCTIONS.md`
- When posting: `POST /notices` with `message`, `priority` (`fyi`, `normal`, `important`), `author` (your Claude name)
- Old notices (>30 days) will be deactivated by Claude-Dashboard during morning refresh

---

## 8. Important Conventions

- **Always GET before PUT** — never overwrite another Claude's recent changes
- **Set `last_touched`** to today's **Pacific** date on every task update — no exceptions
- **Send full arrays** — when updating `next_actions` or `links`, send the complete array
- **Les has final say** — do not change task priorities or remove items without checking with Les first
- **The dashboard is live** — Les sees your updates in real time at https://hlesbrown.github.io/dashboard/
- **Always bump the API version** — every deploy must increment the version number in `app.py`
- **Recurring workflow steps belong in `notes`, not `next_actions`** — `next_actions` is for specific current to-dos, not standing procedures

---

## 9. Les's Schedule & Preferences

- **Working hours:** Roughly 8:00 AM – 5:30 PM Pacific
- **Sunday:** Liturgy prep 4:00 AM, Liturgy on Zoom 6:00 AM, then post-liturgy work block
- **Homily prep:** Never starts before Friday afternoon — do not add to Due Today earlier than that

### Calendar Colors
- 🔵 **Blue = Les** — his events. Act on these when planning the day.
- 🟠 **Orange = Craig** (Les's husband) — informational only
- 🟢 **Green = Dennis** (shared calendar) — informational

### Recurring items (Due Today)
- **Saturday:** "Print Celebrant Booklet" — always on Saturday's list

### Sunday post-liturgy workflow (in this order):
  1. Export Videos
  2. Create Mailchimp Email
  3. Post Homily to Website
  4. Post Homily to Medium.com
  5. ~~Upload Videos to YouTube~~ — **Videos go to Vimeo, not YouTube. Vimeo upload happens automatically — do NOT add this to the Due Today list.**
  6. Add Video links to Calendar
  7. Add Video links to Mailchimp
  8. Update Liturgy booklets

---

## 10. Project URLs

| Resource | URL |
|----------|-----|
| Dashboard PWA | https://hlesbrown.github.io/dashboard/ |
| Dashboard API | https://dashboardapi-production.up.railway.app |
| WorksBench (production) | https://worksbench.app |
| WorksBench (test) | https://test.worksbench.app |
| Author website | https://hlesbrown.com |
| GB Chapel | https://gbchapel.com |
| Shared files | https://raw.githubusercontent.com/hlesbrown/dashboard/main/shared/ |

---

## 11. Calendar — iCloud Live Sync

The dashboard calendar syncs directly from Les's iCloud calendars via CalDAV.

```bash
# Sync calendar (run at session start)
curl -s -X POST -H "X-API-Key: hlbdashboard" \
  "https://dashboardapi-production.up.railway.app/calendar/sync?days=21"
```

| iCloud Calendar | Source | Color | Contents |
|----------------|--------|-------|----------|
| Les Shared Calendar | `les` | Blue | Les's main calendar |
| Craig's Calendar | `craig` | Orange | Craig's items |
| Les' Workout Calendar | `shared` | Green | Shared with Dennis |

Safe to run repeatedly. Clears and reloads the date range each time.

---

*Last updated: June 23, 2026 by Claude-Dashboard*
