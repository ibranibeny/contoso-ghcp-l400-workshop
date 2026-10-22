# Demo script: two kinds of skills, one full-stack app, running on Docker

**Workshop:** GitHub Copilot L400 for Contoso · 13:00–17:00 WIB (coffee break 15:00–15:20)
**Presenter:** Microsoft Indonesia Solution Engineering
**Tools:** VS Code + GitHub Copilot (agent mode, custom agents) + the **superpowers** agent plugin + Docker Engine in Ubuntu 24.04 on WSL 2 on the presenter's Windows laptop
**Running time:** Act 1 = 10 min (14:15–14:25, end of Module 4) · Acts 2–6 = 25 min (15:45–16:10, start of Module 6, after Lab 2)
**Assets:** demo repository `jagajalur-demo` (one git tag per act) · replay `simulation.html` on the workshop site

> Status: DRAFT v0.6. All data in this demo is synthetic. Coordinates are random points in a fictional service area and are **not** real pipeline routes.

---

## 1. The story in one minute (say this)

> "Every team has a golden path: the stack, the Docker setup, the security rules, the way we check that an app actually runs. We'll capture it once as a GitHub Copilot **agent skill**. Then we add a second kind of skill: **superpowers**, an open-source library that teaches the agent how good engineers work: design before code, a written plan, tests first, root cause before fixes, evidence before 'done'. With both, Copilot builds **JagaJalur**, a pipeline patrol and work-order hub for Contoso, runs it on Docker on this laptop, changes it when HSSE adds a rule, and gets it reviewed. A human approves every step."

## 2. What we build

### The skills

| Skill | Kind | Role in the demo |
| --- | --- | --- |
| `contoso-fullstack-docker` | Workspace skill in `.github/skills/`, written in Act 1 | **What** we build with and **how we run it**: templates, conventions, Docker rules, smoke test |
| `brainstorming` | superpowers | Design before code. Architectural path for a new app (Act 2), bounded path for a change (Acts 1 and 5) |
| `writing-plans` | superpowers | Turns the approved spec into small, test-first tasks (Act 2) |
| `executing-plans` | superpowers | Works through the plan task by task (Act 3) |
| `test-driven-development` | superpowers | Red, then green, for every change (Acts 3 and 5) |
| `systematic-debugging` | superpowers | Root cause before any fix (Act 4) |
| `verification-before-completion` | superpowers | Fresh evidence before saying it works (Acts 4 and 5) |
| `finishing-a-development-branch` | superpowers | Verify tests, then merge, open a PR, keep or discard (Act 6) |
| `hsse-code-reviewer` | Custom agent from Lab 2 | Read-only review: file, line, risk, impact, fix (Act 6) |

### The app: JagaJalur, a Pipeline Patrol and Work Order Hub

| | |
| --- | --- |
| **Users** | Patrol officer (field), HSSE supervisor, maintenance planner (role switcher in the header, no sign-in) |
| **Flow** | Log a finding on a pipeline segment → triage → work order with an SLA → done closes the finding |
| **Stack** | React + TypeScript (Vite) on Nginx · FastAPI · PostgreSQL 17 · Docker Compose · UI in Bahasa Indonesia |
| **Rules** | critical → P1 (4 h), high → P2 (24 h), medium → P3 (72 h), low → P4 (168 h); an audit entry for every change |
| **Act 5 rule** | Critical `gas_odor` or `exposed_pipe` opens a P1 work order at once; the dashboard shows overdue work orders |
| **Quality** | 54 pytest tests (44 before Act 5), a five-URL smoke test, `application/problem+json` errors |

### The repository and its tags

| Tag | State | If an act goes wrong |
| --- | --- | --- |
| `act-0-start` | Lab 2 agents, repository instructions, superpowers setup | Start of the demo |
| `act-1-skill` | + `contoso-fullstack-docker` skill | `git checkout act-1-skill` |
| `act-2-spec` | + spec and plan in `docs/superpowers/` | `git checkout act-2-spec` |
| `act-3-app` | + the full stack, **with one realistic defect** for Act 4 | `git checkout act-3-app` |
| `act-4-deploy` | + Nginx fix, two more smoke checks | `git checkout act-4-deploy` |
| `act-5-sla` | + HSSE rule, overdue tile, 10 new tests | `git checkout act-5-sla` |

`main` = `act-5-sla` plus `DEMO.md` (presenter notes). To run a stage: `git switch -c run-1 <tag>`, then `docker compose up -d --build`.

---

## 3. Before the day (presenter checklist)

- [ ] **WSL 2 + Docker Engine:** Ubuntu 24.04 on WSL 2 with Docker Engine, set up with steps 1a to 1e of the Docker guide on the workshop site. In Ubuntu, `docker compose version` shows v2 and `docker run --rm hello-world` works without `sudo`. No Docker Desktop.
- [ ] VS Code (latest) on Windows with the WSL extension and GitHub Copilot signed in; the window is connected to WSL (status bar: *WSL: Ubuntu-24.04*); agent mode and the agent picker are visible. Pick a strong model, or Auto.
- [ ] **superpowers installed:** setting `chat.plugins.enabled` on, then in Copilot Chat (in the WSL window) `/plugin marketplace add obra/superpowers-marketplace` and `/plugin install superpowers@superpowers-marketplace`. If plugins are blocked by policy, or the skills don't show up in WSL, run `scripts/install-superpowers.sh` in Ubuntu: it copies the skills into the repository's `.agents/skills/`, which is git-ignored. Check **Chat: Configure Skills**.
- [ ] Pull the base images: `python:3.12-slim`, `node:22-alpine`, `nginx:stable-alpine`, `postgres:17-alpine`.
- [ ] Build every tag once (`git checkout <tag> && docker compose build`) so rebuilds at the venue use the cache.
- [ ] For the red/green loop in Act 3, before the containers exist: `sudo apt install python3-venv`, then in `api/`: `python3 -m venv .venv && . .venv/bin/activate && pip install -r requirements.txt -r requirements-dev.txt`.
- [ ] Ports 8080 and 8000 are free (`docker compose ls` shows no other stacks).
- [ ] Demo repository cloned into `~/src` inside Ubuntu (not under `/mnt/c`) and reset: `git switch -c live act-0-start`; open it with `code .`.
- [ ] `simulation.html` opened in a browser tab, offline, as the fallback for any act.
- [ ] Full rehearsal on the presentation laptop and the venue Wi-Fi, timed: Act 1 in 10 minutes, Acts 2 to 6 in 25.
- [ ] On the day, by 12:30: laptop on the projector, Ubuntu and VS Code open on `act-0-start`, no stack running (`docker compose ls`), `simulation.html` open in a browser tab.
- [ ] Presentation setup: editor font 18+, terminal font 16+, notifications off, this script on a second screen.
- [ ] Approve terminal commands one by one during the demo. Never pre-approve shell in a skill's `allowed-tools`.

---

## 4. Run of show

### Act 1 — Capture the golden path as a skill (10 min, 14:15–14:25, end of Module 4)

**Say:** "Skills are folders of instructions, scripts and resources that Copilot loads only when they are relevant. Let's write our golden path down once."

**Do:** Copilot Chat → agent mode → paste the prompt.

```text
Create an agent skill in .github/skills/contoso-fullstack-docker that captures
our golden path for full-stack apps on Docker: SKILL.md (name, a precise
description of when to use it, a step-by-step workflow and rules),
templates/ (compose.yaml with db, api, web and a smoke service under the
"test" profile, api.Dockerfile, web.Dockerfile, nginx.conf, .env.example,
.gitattributes), references/conventions.md and scripts/smoke_test.py
(standard library only). Pin image tags, healthchecks on db and api, no
host port for the database, non-root API, secrets only in .env, ask
before deleting volumes. On Windows everything runs in Ubuntu on WSL 2
with Docker Engine. Do not pre-approve shell in allowed-tools.
```

**Expect:** superpowers `brainstorming` classifies this as **bounded** (the prompt already is the design), shows a short design and waits. Answer: *"Yes, create it."* It may also use `writing-skills` for the structure. Then the folder tree in section 5.

**Point out:**
1. Even a small job gets a design and an explicit yes: that is the superpowers hard gate.
2. The `description` is the trigger: Copilot reads it to decide when to load the skill.
3. Templates and scripts load only when needed, so ordinary chats stay lean.
4. A skill is code: review it, then commit `feat: add contoso-fullstack-docker skill`.

**If it goes wrong, or Copilot is still writing at minute 8:** `git checkout act-1-skill` and walk through the files instead. Lab 1 starts at 14:25 sharp.

### Act 2 — Plan with superpowers (4 min, from 15:45, start of Module 6)

**Say:** "Plan before code. superpowers will not let the agent write code before we approve a design."

**Do:** New chat → agent mode → paste the prompt.

```text
Use superpowers to design JagaJalur, a Pipeline Patrol and Work Order Hub
MVP for Contoso, built with the contoso-fullstack-docker skill. Roles: patrol
officer, HSSE supervisor, maintenance planner. Flow: log a finding on a
pipeline segment, triage it, turn it into a work order with an SLA, track
it to done. Synthetic seed data only: 12 segments, 40 findings, 10 work
orders in a fictional service area, not real pipeline routes.
```

**Expect and answer:**

| Copilot (brainstorming) | You |
| --- | --- |
| Classifies the request as **architectural** and explores the repository | — |
| Q1: sign-in? | "No sign-in; a role switcher in the header; record the role in the audit log." |
| Q2: language? | "UI in Bahasa Indonesia; code, API and database in English." |
| Q3: map? | "No map in the MVP; show coordinates." |
| Two or three approaches, recommends the golden path | "Option 1." |
| Design in sections (architecture, data and rules, API and errors, testing) | "Yes" after each |
| Writes `docs/superpowers/specs/<date>-jagajalur-design.md`, commits, asks you to review | Skim it on screen: "Approved. Write the plan." |
| `writing-plans` writes `docs/superpowers/plans/<date>-jagajalur-mvp.md` and asks Subagent-driven or Native | "Native. Work on a new branch in this folder, no worktree." |

**Point out:** one question at a time; approaches before a design; a written spec with a self-review; the plan has exact files, interfaces, failing tests and commands. Copilot can't skip the gate.

**If it goes wrong or runs long:** `git checkout act-2-spec` and open the two files in `docs/superpowers/`.

### Act 3 — Build from the plan, test first (8 min)

**Say:** "Now the agent executes the plan, test first, with the skill's templates, not its imagination."

**Do:** Let it run; approve each terminal command. Narrate Task 2 in full, fast-forward the rest.

**Expect:**
- `git switch -c feat/jagajalur-mvp`.
- Task 1: `db/init/01-schema.sql`, `scripts/generate_seed.py`, then `python3 scripts/generate_seed.py > db/init/02-seed.sql`.
- Task 2 (show it): `api/tests/test_rules.py` first → `python3 -m pytest tests/test_rules.py` fails (`No module named 'app'`) → `api/app/rules.py` → 25 passed.
- Tasks 3–8: API, container, web, compose; 44 tests pass. One commit per task.

**Point out:** the red run before the code; the chat names the skill it is using; Dockerfiles with pinned tags and a non-root user; services call each other by name.

**If it isn't done by minute 8:** stop it, `git checkout act-3-app`, and say "this is what the same plan produced in rehearsal".

### Act 4 — Deploy on Docker and verify (5 min)

**Say:** "Deploying here means running it on this laptop with Docker. Watch the loop: build, run, verify, fix."

**Do:** paste the prompt; approve each command.

```text
Deploy JagaJalur on Docker: run docker compose up -d --build, then the
smoke test. Add http://web/api/findings?size=1 and http://web/api/stats
to SMOKE_CHECKS. If anything fails, read the logs, find the root cause,
fix it and repeat until every check passes. Then give me the URLs.
```

**Expect (from `act-3-app`):** `db` and `api` healthy; the smoke test shows `PASS` for `api:8000/health` and `web/`, `FAIL` (404) for the three `web/api/...` URLs. `systematic-debugging`: compare direct and proxied calls → `docker compose logs api` shows `GET /api/health 404` → root cause: `proxy_pass http://api:8000;` keeps the `/api` prefix → one-line fix to `http://api:8000/;` → `docker compose up -d --build web` → five `PASS`. Then `verification-before-completion`: `docker compose ps`, `docker compose run --rm api pytest`.

> The defect is deliberate in `act-3-app` so the fix loop always happens. If your live Act 3 produced a working proxy, say so and show this loop in the replay.

**Tour (2 min):** `http://localhost:8080` in the Windows browser (WSL forwards `localhost`) → dashboard → **Temuan** → a triaged finding as **Supervisor HSSE** → "Buat work order" (P2 / 24 h preview) → **Work order** board as **Perencana pemeliharaan** → `http://localhost:8000/docs`.

**If it goes wrong:** `git checkout act-4-deploy` and `docker compose up -d --build`, or play Act 4 in the replay.

### Act 5 — A new HSSE rule, test first (6 min)

**Say:** "HSSE has a new rule. In real life that's a ticket; here it's a prompt."

```text
New HSSE rule: when a finding is gas_odor or exposed_pipe with severity
critical, create a P1 work order with a 4-hour SLA, write an audit entry
and show overdue work orders on the dashboard. Add API tests. Redeploy
only the changed services and rerun the smoke test.
```

**Expect:** `brainstorming` takes the **bounded** path: a short design in chat (rule function, reuse `open_work_order`, actor `system:hsse-rule`, overdue condition, dashboard tile), no spec file. Answer *"Yes."* Then `test-driven-development`: `api/tests/test_hsse_rule.py` first → `docker compose run --rm --build api pytest tests/test_hsse_rule.py` → 9 failed, 1 passed (the low-severity guard test passes on purpose) → code → 54 passed → web changes → `docker compose up -d --build api web` → smoke test green.

**Show:** log a critical gas odor on `SEG-CTS-07` as **Petugas patroli** → the detail shows a P1 work order due in 4 hours and the audit entry "WO P1 otomatis (aturan HSSE)" → the dashboard's **WO terlambat** tile.

**Point out:** no schema change, so no database reset; only `api` and `web` are rebuilt.

**If it goes wrong:** `git checkout act-5-sla`, then `docker compose up -d --build`.

### Act 6 — Review and finish (2 min)

**Do:**
1. Agent picker → `hsse-code-reviewer` → "Review the changes on this branch against main." Read-only; a table of file, line, risk, impact, fix; "Merge: yes" or "after fixes".
2. Agent mode → "Finish the branch." `finishing-a-development-branch` runs the tests and offers four options; choose **Push and create a pull request**. Request **Copilot code review** on the pull request.

**Close (say):** "The same skills live in the repository and in the plugin, so Copilot CLI and the cloud agent use them too. Assign an issue like 'Add a CSV export of work orders' to Copilot, then pull and redeploy here with one command."

---

## 5. Reference: the skill from the repository

### `SKILL.md`

```markdown
---
name: contoso-fullstack-docker
description: >-
  Use when asked to scaffold, build, containerize, run, deploy or extend a
  full-stack web app or one of its services. Our golden path on a laptop:
  React + TypeScript (Vite) served by Nginx, a FastAPI API and PostgreSQL,
  wired with Docker Compose and verified by a smoke test.
---

# Full-stack on Docker: our golden path

This skill says **what** we build with and **how we run it**. When the
superpowers skills are installed, they say **how we work**: `brainstorming`
for the design, `writing-plans` for the plan, `test-driven-development` while
coding, `systematic-debugging` when something fails and
`verification-before-completion` before saying it works. Follow both.

## Workflow

1. Find the approved design: `docs/superpowers/specs/*-design.md` (superpowers)
   or `docs/spec.md`. If there is none, stop and get one approved first; with
   superpowers installed, that means the brainstorming skill.
2. Create this layout, starting from the files in `templates/`:
   - `web/`: React + TypeScript (Vite). Nginx serves the build and proxies
     `/api/` to `api:8000/`. Keep the trailing slash in `proxy_pass` so the
     `/api` prefix is stripped.
   - `api/`: FastAPI with `/health` that also runs `SELECT 1`, plus
     `app/healthcheck.py` for the container healthcheck; Pydantic validation;
     OpenAPI docs at `/docs`.
   - `db/init/`: `01-schema.sql` and `02-seed.sql`. Synthetic data only;
     generate the seed with a script that uses a fixed random seed.
   - `tests/smoke/smoke_test.py`, copied from `scripts/`.
   - `compose.yaml`, `.env.example`, `.gitattributes`, one `.dockerignore`
     per service.
3. Apply [the conventions](references/conventions.md) to every endpoint and screen.
4. Run `docker compose up -d --build`, then `docker compose run --rm smoke`.
5. If anything fails, read `docker compose logs --tail 100 <service>`, find the
   root cause, fix it and repeat step 4 until every smoke check passes.
6. Run the API tests in the container: `docker compose run --rm api pytest`.
7. Finish with a summary: what was built, the URLs (http://localhost:8080 and
   http://localhost:8000/docs) and how to stop the stack (`docker compose down`).

## Rules

- Pin image tags: `python:3.12-slim`, `node:22-alpine`, `nginx:stable-alpine`,
  `postgres:17-alpine`. Never `latest`.
- Healthchecks on `db` and `api`; `depends_on` with `condition: service_healthy`.
- Services call each other by service name (`db`, `api`), never `localhost`.
- The database gets no host port. The API runs as a non-root user.
- Secrets only in `.env` (git-ignored); commit `.env.example`.
- Synthetic data only: no real customer, asset or location data.
- Windows laptops run everything in Ubuntu on WSL 2 with Docker Engine; keep the
  repository in the Linux file system (`~/src`), not under `/mnt/c`.
- `db/init` scripts run only when the database volume is empty. A schema change
  needs a migration, or in development a fresh volume.
- Ask before `docker compose down -v`, `docker system prune` or deleting volumes.

## Files in this skill

- [templates/compose.yaml](templates/compose.yaml): `db`, `api`, `web`, plus `smoke` under the `test` profile
- [templates/api.Dockerfile](templates/api.Dockerfile): `python:3.12-slim`, non-root, optional dev dependencies
- [templates/web.Dockerfile](templates/web.Dockerfile): build on `node:22-alpine`, serve on `nginx:stable-alpine`
- [templates/nginx.conf](templates/nginx.conf): serve the SPA and proxy `/api/`
- [templates/.env.example](templates/.env.example) and [templates/.gitattributes](templates/.gitattributes)
- [references/conventions.md](references/conventions.md): API style, errors, audit log, data rules
- [scripts/smoke_test.py](scripts/smoke_test.py): standard library only; runs inside the Compose network
```

### `templates/compose.yaml`

```yaml
# Golden-path Compose file. Copy it to the repository root as compose.yaml.
services:
  db:
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-app}
      POSTGRES_DB: ${POSTGRES_DB:-app}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?Set POSTGRES_PASSWORD in .env}
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./db/init:/docker-entrypoint-initdb.d:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 5s
      timeout: 3s
      retries: 12
    restart: unless-stopped

  api:
    build:
      context: ./api
      args:
        INSTALL_DEV: "true" # adds pytest, so `docker compose run --rm api pytest` works
    environment:
      DB_HOST: db
      DB_NAME: ${POSTGRES_DB:-app}
      DB_USER: ${POSTGRES_USER:-app}
      DB_PASSWORD: ${POSTGRES_PASSWORD:?Set POSTGRES_PASSWORD in .env}
    ports:
      - "${API_PORT:-8000}:8000"
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "python", "-m", "app.healthcheck"]
      interval: 10s
      timeout: 5s
      retries: 6
      start_period: 10s
    restart: unless-stopped

  web:
    build: ./web
    ports:
      - "${WEB_PORT:-8080}:80"
    depends_on:
      api:
        condition: service_healthy
    restart: unless-stopped

  smoke:
    image: python:3.12-slim
    profiles: ["test"]
    environment:
      SMOKE_CHECKS: http://api:8000/health,http://web/,http://web/api/health
    volumes:
      - ./tests/smoke:/smoke:ro
    command: ["python", "/smoke/smoke_test.py"]
    depends_on:
      web:
        condition: service_started

volumes:
  pgdata:
```

### `templates/nginx.conf`

```nginx
server {
  listen 80;
  server_name _;
  root /usr/share/nginx/html;
  index index.html;

  # API: the trailing slash in proxy_pass strips the /api prefix,
  # so /api/findings reaches the API as /findings.
  location /api/ {
    proxy_pass http://api:8000/;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  # Fingerprinted build assets can be cached for a long time.
  location /assets/ {
    try_files $uri =404;
    add_header Cache-Control "public, max-age=604800, immutable";
  }

  # Single-page app: unknown paths fall back to index.html.
  location / {
    try_files $uri $uri/ /index.html;
  }
}
```

The full skill, including `references/conventions.md` and `scripts/smoke_test.py`, is in `.github/skills/contoso-fullstack-docker/` at tag `act-1-skill`.

## 6. Reference: spec and plan

The approved design is `docs/superpowers/specs/2026-10-22-jagajalur-design.md` and the plan is `docs/superpowers/plans/2026-10-22-jagajalur-mvp.md` (tag `act-2-spec`). In short:

```text
segments     (id, code, name, area, network_type, length_km)
findings     (id, segment_id, category, severity, status, description, latitude, longitude,
              reported_by, reported_at, updated_at)
work_orders  (id, finding_id, priority P1–P4, team, status open | in_progress | done,
              due_at, created_at, closed_at)
audit_log    (id, entity, entity_id, action, actor, at, details)

GET /health · GET /segments · GET|POST /findings · GET|PATCH /findings/{id}
POST /findings/{id}/work-orders · GET /work-orders · PATCH /work-orders/{id} · GET /stats
```

## 7. Recovery cheat sheet

| Problem | Do this |
| --- | --- |
| superpowers skills don't show up | Check `chat.plugins.enabled` and **Chat: Configure Skills** in the WSL window; or run `scripts/install-superpowers.sh` in Ubuntu and reload VS Code |
| `Cannot connect to the Docker daemon` | In Ubuntu: `ps -p 1 -o comm=` must print `systemd`; then `sudo systemctl start docker` |
| The app is gone after closing Ubuntu and VS Code | WSL stopped the idle distribution; open Ubuntu again and the containers restart (`restart: unless-stopped`) |
| Copilot skips the design and starts coding | Stop it: "Use the superpowers brainstorming skill first." |
| Port 8080 or 8000 busy | Change `WEB_PORT` or `API_PORT` in `.env`, then `docker compose up -d` |
| Docker Hub pull limit | `docker login`; the base images should already be cached |
| npm or pip is slow on venue Wi-Fi | Jump to the next tag; rebuilds use the rehearsal cache |
| Agent drifts or loops | Stop it, new chat: "Continue the plan in docs/superpowers/plans/… from Task N", or jump to the tag |
| Stack in a bad state | Ask the room first, then `docker compose down -v` and `git checkout <tag>` |
| Anything else | Play the act in `simulation.html` |

## 8. Key messages to land

1. A skill turns team know-how into a reusable, reviewable asset.
2. Process skills (superpowers) make the agent design, test and verify before it claims "done".
3. Agent mode closes the loop with Docker: build, run, verify, fix.
4. Humans stay in charge: they approve the design, the plan, every terminal command and the merge.
5. Everything ran on a laptop, with synthetic data.
