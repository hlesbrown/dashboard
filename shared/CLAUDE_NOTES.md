# Claude Handbook — Permanent Notes

This is the shared reference document for all Claudes working on Les Brown's projects. Fetch this at the start of every session.

```
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

Check the notice board for announcements when new files are added or existing ones are updated.

---

## 3. Who's Who

| Claude | Responsibility | Project Group(s) |
|--------|---------------|-------------------|
| **Claude-Dashboard** | Dashboard maintenance, daily planning, Due Today list, cross-project coordination | `today`, all groups |
| **Claude-NovelBank** | NovelBank app development (Flask/PostgreSQL on Railway) | `novelbase` |
| **Claude-Podcast** | Podcast production, social media, content distribution | `podcast` |
| **Claude-Sycamore** | Sycamore Boy publication pipeline | `sycamore` |
| **Claude-Website** | hlesbrown.com, author marketing | `author` |
| **Claude-Liturgy** | GB Chapel liturgy, homilies, Lenten series | `gbchapel` |
| **Claude-JC** | Jonathan's Circle, CIRCLING magazine | `circle` |

Not all Claudes may be active at any given time. New Claudes may be added.

---

## 4. Session Protocol

### Starting a session:
1. Fetch this handbook (if not already in your project knowledge)
2. Fetch the dashboard: `GET /dashboard` — read the notice board and check your tasks
3. If relevant, fetch `DASHBOARD_API_INSTRUCTIONS.md` for full API details
4. Do your work

### Ending a session:
1. Update your tasks: `PUT /tasks/<id>` with new status, next_actions, last_touched
2. Post a progress log: `POST /logs` with `group_id`, `source` (your name), `summary`, `details`
3. If you have updates for other Claudes, post a notice: `POST /notices`

---

## 5. Due Today List

Claude-Dashboard posts a daily prioritized task list in the `today` group each morning.

**Rules for other Claudes:**
- You may add items using bracket notation: `"[YourProject] Description"`
- Always `GET /tasks?group=today` first so you don't overwrite others' additions
- To mark done, prepend ✅ to the action text — don't remove items
- Do NOT create new tasks in the `today` group — only Claude-Dashboard does that
- The task ID changes daily — always fetch fresh

---

## 6. Notice Board Etiquette

- Notices are for **transient announcements** — new files, API changes, task reassignments, session handoffs
- Permanent info belongs in this handbook or in `DASHBOARD_API_INSTRUCTIONS.md`
- When posting: `POST /notices` with `message`, `priority` (fyi, normal, important), `author` (your Claude name)
- Old notices will be deactivated periodically by Claude-Dashboard

---

## 7. Important Conventions

- **Always GET before PUT** — don't overwrite another Claude's recent changes
- **Set `last_touched`** to today's date on every task update
- **Send full arrays** — when updating `next_actions` or `links`, send the complete array, not just additions
- **Les has final say** — do not change task priorities or remove items without checking with Les first
- **The dashboard is live** — Les sees your updates in real time at https://hlesbrown.github.io/dashboard/

---

## 8. Les's Schedule & Preferences

- **Daily hours:** 4:30 AM – 10:00 PM (Sundays: 4:00 AM – 10:00 PM)
- **Timezone:** Pacific (America/Los_Angeles). The dashboard has a live clock showing Les's local time.
- **Knee trouble** — daily walk is on the calendar but not happening right now
- **Calendar:** Now syncs automatically from iCloud (see Section 10). Blue = Les, orange = Craig, green = shared with Dennis.
- **Monday noon AA meeting** is generally shared with Craig
- **Sunday:** Liturgy prep 4:00 AM, Liturgy on Zoom 6:00 AM, then post-liturgy work block
- **Saturday recurring:** "Print Celebrant Booklet" MUST appear on every Saturday Due Today list — Les forgets this often
- **Sunday recurring:** Post homily to Medium & process Zoom videos; Update Liturgy booklets. These go on every Sunday Due Today list.

### Due Today Scheduling Protocol
- **Check the clock.** Compare current Pacific time against today's calendar events. Do NOT put events on the Due Today list that have already passed — Les doesn't need to be told to do something he's already done.
- **Calendar events are NOT tasks.** The calendar shows scheduled events (liturgy, walk, meetings). Due Today shows *work items* — things Les needs to remember to do. Don't duplicate calendar items into Due Today.
- Carry-forward items from yesterday only if they weren't completed.
- Recurring items (Saturday booklet, Sunday liturgy processing) go on automatically regardless.

---

## 9. Project URLs

| Resource | URL |
|----------|-----|
| Dashboard PWA | https://hlesbrown.github.io/dashboard/ |
| Dashboard API | https://dashboardapi-production.up.railway.app |
| NovelBank (production) | https://novelbank.net |
| NovelBank (test) | https://test.novelbank.net |
| Author website | https://hlesbrown.com |
| GB Chapel | https://gbchapel.com |
| Shared files | https://raw.githubusercontent.com/hlesbrown/dashboard/main/shared/ |

---

## 10. Calendar — iCloud Live Sync

The dashboard calendar now syncs directly from Les's iCloud calendars via CalDAV. No more manual screenshot loading.

### How it works
- `POST /calendar/sync?days=21` — pulls events from iCloud and loads them into the dashboard
- `GET /calendar/list` — shows all iCloud calendars and which ones are being synced
- The dashboard webapp fetches calendar data from the API on every load

### Calendars synced

| iCloud Calendar | Dashboard Source | Color | Contents |
|----------------|-----------------|-------|----------|
| Les Shared Calendar | `les` | Blue (#3B82F6) | Les's main calendar — liturgy, walk, meetings, appointments |
| Craig's Calendar | `craig` | Orange (#F97316) | Craig's items — tennis, gardening, birthdays, watering |
| Les' Workout Calendar | `shared` | Green (#22C55E) | Shared with Dennis — sponsee meetings, lunch |

### Sync workflow
Any Claude can trigger a sync at any time:
```bash
curl -s -X POST -H "X-API-Key: hlbdashboard" "https://dashboardapi-production.up.railway.app/calendar/sync?days=21"
```
This clears and reloads all events for the date range. Safe to run repeatedly.

### Manual event management
Events can still be added/edited/deleted manually via the calendar API endpoints (`POST /calendar`, `PUT /calendar/<id>`, `DELETE /calendar/<id>`). Manual events will be overwritten on next sync for dates in the sync range.

### Dashboard features
- **Week navigation:** ◀ ▶ arrows to move between weeks, "Today" button to return
- **Time sorting:** Events sorted by time, all-day events at top
- **Past event hiding:** On today's view, events whose time has passed auto-hide based on Les's local Pacific time
- **Live clock:** Dashboard header shows Les's current Pacific time (ticks every second), visible to all Claudes checking the dashboard

### If calendars stop syncing
Run `GET /calendar/list` to check if calendar names have changed. The sync matches by name, so renamed calendars will be silently skipped. Update the CALENDAR_MAP in `app.py` if names change.

### Apple credentials
iCloud access uses an app-specific password stored as Railway environment variables (`APPLE_ID`, `APPLE_APP_PASSWORD`). If authentication fails, Les may need to generate a new app-specific password at https://appleid.apple.com.

---

*Last updated: February 14, 2026 by Claude-Dashboard*
