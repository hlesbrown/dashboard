# PROTOCOL-INDEX.md — Where every project's protocols live

*Master index for all Claudes working on Les Brown's projects. Built October 14, 2026.*

`CLAUDE_NOTES.md` section 2 has long promised this file. It did not exist — instead each
project quietly built its own local anchor. This file is the index those references meant.

**What these files are:** each project's `PROTOCOL-INDEX.md` is that Claude's *session anchor* —
who it is, its dashboard group, its start sequence, and where its working protocols live.
They are orientation files, not protocol documents. Only Claude-Liturgy keeps a genuine
separate protocol set (seven files in its own `protocols/` folder).

---

## The two files that apply to everyone

| File | Location |
|---|---|
| `CLAUDE_NOTES.md` | `https://raw.githubusercontent.com/hlesbrown/dashboard/main/shared/CLAUDE_NOTES.md` |
| `DASHBOARD_API_INSTRUCTIONS.md` | `https://raw.githubusercontent.com/hlesbrown/dashboard/main/shared/DASHBOARD_API_INSTRUCTIONS.md` |

Both are public, need no credentials, and are fetched at session start.
**CLAUDE_NOTES.md is authoritative.** Where a project file disagrees with it, the handbook wins.

---

## Per-project anchors

All paths are Dropbox unless stated. Reach Dropbox through the Dropbox connector (your Dropbox tools); there is no shared Dropbox token any more (retired Nov 23, 2026).

| Claude | Group | Anchor file | Covers | Last updated |
|---|---|---|---|---|
| Claude-Liturgy | `gbchapel` | `/Claude Shared/Claude-Liturgy/protocols/PROTOCOL-INDEX.md` | Index to 7 real protocol files: Sunday post-liturgy, weekly Mailchimp newsletter, general intercessions, calendar events, Liturgy Videos pages, homily posts, Mailchimp gotchas | Sep 27 2026 |
| Claude-CSP | `csp` | `/Claude Shared/Claude-CSP/PROTOCOL-INDEX.md` | Identity, session start, CSP website and infrastructure | Sep 18 2026 |
| Claude-JC | `circle` | `/Claude Shared/Claude-JC/PROTOCOL-INDEX.md` | CIRCLING build. **Carries an absolute rule: contributor content is NEVER edited for content** | Aug 9 2026 |
| Claude-Docent | `docent` | `/Claude Shared/Claude-Docent/PROTOCOL-INDEX.md` | Docent training app, session start sequence | May 19 2026 |
| Claude-Bondings | `booksix` | `/Claude Shared/Claude-Bondings/PROTOCOL-INDEX.md` | Book Six world-building. **Stale: still self-identifies as "Claude-Six"** | Apr 20 2026 |
| Claude-Marketing | `author` | Book Marketing & Promotion project docs → `PROTOCOL-INDEX.md` | Marketing strategy, brand assets, Sycamore Boy status, Higgsfield element registry | Oct 14 2026 |

**No anchor file exists for:** Claude-Dashboard, Claude-WB, Claude-Podcast, Claude-Editor,
Claude-Cowork, Claude-Interface. Not necessarily a problem — only worth creating one where a
project has real procedure that would be expensive to rediscover.

---

## Logs are not state

`POST /logs` appends narrative. **Nothing is built from it.** The Due Today list and every
priority call are built from TASKS.

If a finding changes the state of a task — something verified, a question answered, a blocker
cleared — you must `PUT /tasks/<id>` as well as posting the log. This is already mandatory in
CLAUDE_NOTES.md section 4, STEP 1. It is the step most often skipped.

Worked example, October 2026: the IngramSpark wholesale discount was verified with Les on Oct 2
and logged in full. The task was never updated. On Oct 6 a scheduled run read the task, found no
verification, and wrote "still unverified". By Oct 14 that had escalated through a notice to the
top of Les's priority list — twelve days of phantom urgency pointing at a decision that had
already been made correctly and deliberately.

---

## Keeping this file honest

Add a row when a project gains an anchor file. Correct the date when one is revised.
A stale index is worse than none, because it is believed.
