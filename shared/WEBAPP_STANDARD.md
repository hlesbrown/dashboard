# Lesland Webapp Standard

**In force from November 23, 2026.** Approved by Les. Kept by Claude-Dashboard; only Les changes it.
Applies to every Claude that builds, changes, deploys, or operates a webapp for Les.

Read this before touching any webapp. Each app's own `CLAUDE.md` adds only what is specific to that app; where they conflict, ask Les.

---

## Why this exists

In November 2026 four apps had been built four different ways. In two days that produced:
- a service Railway could not rebuild, because its code lived only on a laptop;
- a redeploy that silently rolled the dashboard back nine months, because the real code had never been committed;
- an API key leaked through a public file;
- an MCP server anyone on the internet could use.

Each of those came from a Claude improvising its own workflow. This standard is the one workflow. All four apps were brought up to it between Nov 21 and Nov 23, 2026.

---

## 1. Every app has one home

- **Code:** one private GitHub repo under `hlesbrown` (or the app's own organization, e.g. CSP). The repo is the only copy that matters. If it isn't committed, it doesn't exist.
- **Hosting:** its own Railway project, named for the app. Never put two apps in one Railway project.
- **Environments:** `staging` and `production`, each deploying from its own branch (`staging`, `main`).
- **Optional beta tier** (see §2a): a second service inside the production environment, deploying from branch `beta`.
- **Addresses:** anything people or other systems depend on gets a domain Les owns (e.g. `docenttraining.org`, `api.hlesbrown.com`), not a `*.up.railway.app` address. Railway addresses are fine for staging.
- **Specs, notes, screenshots** can live in Dropbox. **Deployable code never does.**

## 2. One way to ship

1. Work on the `staging` branch. Bump the app's version number.
2. Push `staging`. Railway deploys staging automatically.
3. Tell Les what changed and what to check on the staging site. Wait for his OK.
4. Merge `staging` into `main` and push. Railway deploys production automatically.
5. Confirm production is serving the new version.

**Never:**
- produce zip files;
- ask Les to download, upload, unzip, or "Upload Folder";
- use `railway up`;
- push untested work, or anything without Les's OK, to `main` (or `beta`);
- change Railway settings, variables, or domains without asking Les.

Railway's "Deploy" button and variable changes rebuild from GitHub. That is exactly why the code must be there.

### 2a. Beta tier (optional)

Use it when real users try a release before everyone gets it (WorksBench does).

- Beta is a service **inside the production environment** (e.g. `worksbench-beta`, branch `beta`). It shares production's database, Redis and worker over Railway's private network.
- Flow: `staging` → `beta` → `main`, with Les's OK at each step.
- **Beta is production with newer code:**
  - It works on real data.
  - A schema change shipped to beta is a production schema change: it needs Les's OK, and it must stay backward-compatible with `main`.
  - Background jobs started from beta run on the production worker.
- If beta should instead be a sandbox, it needs its own environment and database. Decide that with Les before building it.

## 3. Secrets

- **App secrets** (database URLs, API keys, OAuth secrets) live only in **Railway variables**, per environment.
- **Claude credentials** live only in the **project instructions of the Claude that uses them**.
- **Prefer connectors to credentials.** Dropbox is reached only through the Dropbox connector; the shared Dropbox token was retired Nov 23, 2026.
- **Never** put a secret in:
  - a repo (public or private);
  - the dashboard (tasks, notices, logs);
  - Dropbox;
  - a handbook;
  - a chat reply;
  - a commit message.

  The dashboard is a shared board, not a vault.
- **Never paste a full database URL into chat.** It contains the password. Host and port are enough for troubleshooting.
- **Copy variables between services with Railway's Raw Editor** (copy all from the old service, paste into the new one). Nothing passes through chat.
- **Code must not carry a working fallback secret.** `os.environ.get("KEY", "real-value")` is forbidden; use an empty or obviously fake default.
- **Read the variable name the code actually uses before setting one.** The dashboard read `API_KEY` while Railway held `DASHBOARD_API_KEY`, so the rotation silently didn't happen.
- **A service exposed to the internet must check its credential.** If a variable like `ACCESS_KEY` is read, it must be enforced, and tested from outside.

## 4. Builds that don't drift

- **Pin every dependency to an exact version** (`flask==3.1.0`, not `flask>=3`). An unpinned `mcp` jumped to 2.x on rebuild and broke the dashboard MCP.
- **Start command in a `Procfile`.** Don't use `railway.json` / `railway.toml`; Railway stopped honoring them Dec 1, 2026. Worker services set their start command in Railway Settings (e.g. `python worker.py`).
- **Python version in `runtime.txt`, and keep it a recent patch release.**
  - Railway's builder refuses Python builds that predate GitHub artifact attestations. `3.11.9` failed; `3.11.14` worked.
  - Fix it by bumping the patch version. **Never** set `MISE_PYTHON_GITHUB_ATTESTATIONS=false`.
- **`.gitignore`** keeps out `.env`, caches, and zip files.
- **README** in every repo: what the app is, who uses it, how it deploys, and a table of every Railway variable it needs (names and purpose, never values).
- **`CLAUDE.md`** in every repo: points to this standard, then gives the app-specific rules.

## 5. Staging must be safe

- Staging has **its own database**, a copy of production made inside Railway (see §6).
- **Outside connections that act as a person or move money are disabled on staging:**
  - no Gmail/Outlook secrets, and stored OAuth tokens cleared in the staging copy;
  - **no live payment keys** (Stripe).

  A staging app must never charge a card, send email as Les, or touch a real calendar or inbox.
- **Shared storage is shared:** if staging and production use the same Cloudinary, Dropbox or similar, staging never deletes and avoids test uploads.
- Staging and production use **different access keys** where the app has its own (e.g. the MCP).

## 6. Data and databases

- **Schema changes:** staging first, verify, then production after the code reaches `main`. Migrations are named, idempotent, append-only.
- **No destructive SQL on production** (DROP, DELETE, TRUNCATE, UPDATE without WHERE) unless Les approves the exact statement.
- **Copying data** between databases:
  - Run it in the **target** database's Railway Console with `pg_dump "<source public URL>" --no-owner --no-acl | psql "$DATABASE_URL"`.
  - Check the top bar shows the right project and environment first.
  - Copy **before** the app is given its `DATABASE_URL`, so the app can't create empty tables that collide with the copy.
  - Verify with `\dn` and a table count per schema.
- **Turn off public networking** on a database when nothing outside its project needs it. When migrating, copy from the *old* database's public URL, so the new production database never needs one.

## 7. Moving or replacing a live service

Build the new home beside the old one; never modify the live service in place.

1. Les creates the empty private repo. Commit the running code unchanged, then fixes as separate commits.
2. Les renames the old Railway project to **Legacy** and creates the new one with the app's name.
   - Connecting the repo makes Railway build `main` immediately. A failed first build there is harmless.
3. Build production in the new project:
   - Add the database and Redis.
   - Copy the data.
   - Add the services and paste their variables, pointing at the new services with references (`${{service-name.DATABASE_URL}}`).
   - Deploy, then test on the Railway address: login, data, version, a background job.
4. Move each domain, one at a time:
   - remove it from the old service;
   - add it to the new one (port shown on the old service, usually 8080);
   - update the CNAME and `_railway-verify` TXT at the DNS host;
   - test.

   Keep each record's existing Cloudflare proxy setting. Add DNS records by hand; don't use Railway's one-click DNS connect, which gives Railway standing access to the DNS.
5. Build staging the same way, with no live payment or email secrets.
6. **Stop** the old services, but keep them and their databases for about two weeks as a rollback. Then delete them.
7. Update every client, scheduled task, connector and Claude that references an old address. Leave a READ-FIRST handoff note in the app Claude's Dropbox folder.

**Railway names:** service names must be unique across a whole project. Production gets the plain name (`worksbench-app`, `worksbench-db`); staging adds `-staging` (`worksbench-app-staging`). Variable references must use the exact names.

## 8. Housekeeping

See handbook §5 *Housekeeping*:
- Every Claude tidies its own dashboard entries on every visit.
- Every Claude reviews its own Dropbox folder periodically for obsolete files and secrets, and deletes only with Les's OK.

## 9. Checklists

**Starting a new app**
- [ ] Les has created the empty private repo; the Claude GitHub app can see it
- [ ] Railway project created with `staging` and `production`, each wired to its branch (plus `beta` service if needed)
- [ ] Procfile, recent-patch runtime.txt, pinned requirements, .gitignore, README, CLAUDE.md
- [ ] Secrets in Railway variables only; no fallback secrets in code
- [ ] Staging safeguards (§5) in place
- [ ] Domain planned for anything people will rely on
- [ ] Added to the dashboard as a group/task, and to `WEBAPP_INVENTORY.md`

**Bringing an existing app up to standard**
- [ ] Find the code that is **actually running** (Railway console, deploy folder, version shown on the live site), not the newest file you can find. Verify byte-for-byte before committing.
- [ ] Commit it unchanged first; make fixes as separate commits
- [ ] Audit: config files, unpinned deps, Python patch version, hard-coded secrets, variable-name mismatches, unenforced access checks, live payment keys on non-production tiers
- [ ] Build per §7, test, switch over, keep Legacy two weeks

**Every deploy**
- [ ] Version bumped
- [ ] Tested on staging
- [ ] Les's OK
- [ ] Merged to main (via beta if the app has one)
- [ ] Production verified

---

## Current apps

| App | Repo | Railway project | Production address | Status |
|---|---|---|---|---|
| Docent Training | `hlesbrown/docent-training` | Docent Training | docenttraining.org | Meets standard (Nov 21) |
| Dashboard API + MCP | `hlesbrown/dashboard_api`, `hlesbrown/dashboard-mcp` | Dashboard | api.hlesbrown.com, mcp.hlesbrown.com | Meets standard (Nov 22) |
| WorksBench | `hlesbrown/worksbench` | WorksBench (staging; production + beta) | worksbench.app, beta.worksbench.app | Meets standard (Nov 23) |
| College Support Program | `CollegesupportProgramCV/cspcv-website` | CSP's own account | — | Close; retire the boot-time code download workaround |

Retiring: the **Legacy** Railway project (old WorksBench, old dashboard API, old Docent database) is to be deleted about two weeks after Nov 23, 2026, once nothing uses it.
