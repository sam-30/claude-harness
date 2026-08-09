# Infrastructure

## Containers
Every project is fully docker-ized. At minimum, docker-compose should define:
- `web` (or `backend`) — Django/DRF app
- `frontend` — React/TypeScript app (dev server or build step, depending on stage)
- `db` — PostgreSQL
- `redis` — Redis, used by Celery
- `celery-worker` — Celery worker process
- `celery-beat` — Celery beat scheduler (only if the project actually has scheduled tasks)

## Conventions
- Every service should be runnable with a single `docker compose up`.
- Environment variables belong in `.env` (gitignored) with a checked-in `.env.example` documenting every required variable.
- Database migrations run via `docker compose exec web python manage.py migrate` — never against a local, non-containerized Postgres.
- `.env` files can have tokens and secrets that link to our standard netlify, railway, twilio, github, aws, or other 3rd party auth information 

## When scaffolding a new project
- Start from a docker-compose.yml covering the services above, sized to what the project actually needs (e.g. skip `celery-beat` if there's nothing to schedule).
- Confirm the compose stack boots (`docker compose up --build`) before considering infrastructure "done" for a phase.

## Version control platform (sdlc-tracker)
- All projects are hosted on [`sdlc-tracker`](https://github.com/sam-30/sdlc-tracker), a self-hosted git forge (repos, PRs, merges — its own thing, not github.com). It runs as its own standalone Docker stack, started once and shared across projects — do **not** declare it as a service inside each individual project's docker-compose.yml.
- Start the forge itself from its own repo: `cd sdlc-tracker/infra && docker compose up -d`. That project is still under active development (its own ARCHITECTURE.md/plan tracks current phase); confirm what's actually running there before assuming the full API is up — the infra stack may only cover Postgres/Redis/MinIO until its backend service lands.
- One-time per machine: `forge setup` — registers a git credential helper so plain `git push`/`git pull`/`git clone` against the forge work with no extra flags.
- New project repo: `forge repo init <org>/<repo>` creates the bare repo on the forge; then in the project, `git remote add origin http://127.0.0.1:8000/<org>/<repo>.git` (adjust host/port if the forge isn't running on its default).
- The `forge` CLI (`scripts/forge.py` in sdlc-tracker) is also what phases use to open and merge pull requests — see workflow.md for when/how each phase invokes it.

## Local LLM (Ollama)
- Ollama runs locally on the dev machine, shared across projects the same way `sdlc-tracker` is — not declared as a per-project docker-compose service.
- Multiple models are already downloaded; default to `qwen2.5-coder:32b` for the agent roles described in workflow.md unless a project/role specifies a different model.

## Cloud based hosting
- Netlify will host the frontend for each project.  Netlify CLI is authorized on the local development macbook where new projects will be built
- Railway will host the backend, db, worker, redis, celery code. The Railway CLI is also on the local development macbook.  If either railway or netlify needs authentication, ask the user.
- Twilio for SMS management. Credentials (`TWILIO_ACCOUNT_SID_TEST`, `TWILIO_ACCOUNT_SID_PROD`, `TWILIO_AUTH_TOKEN_TEST`, `TWILIO_AUTH_TOKEN_PROD`) live in `.env`, never in this file — see `.env.example` in the repo root for the variable names and the "Conventions" section above.
