# Tech Stack

Every project uses this stack unless the user explicitly says otherwise for that specific project. Do not introduce alternative frameworks, ORMs, state managers, databases, or infra without flagging the mismatch and asking first.

## Backend
- Python, Django + Django REST Framework (DRF) for all APIs
- PostgreSQL as the primary database
- Redis as the broker/result backend for Celery
- Celery for background/async task processing (worker + beat, as needed)

## Frontend
- React + TypeScript
- No untyped JS files in new code

## Infrastructure
- All services run in Docker containers via docker-compose (see infrastructure.md)
- No local, non-containerized dev environments — commands should assume `docker compose exec` / `docker compose run`

## Version control
- All project repos are hosted on [`sdlc-tracker`](https://github.com/sam-30/sdlc-tracker), a self-hosted git forge, not directly on github.com. It runs locally via Docker as its own standalone stack (see infrastructure.md).
- Day-to-day repo/PR operations go through its `forge` CLI rather than raw `git push`/manual PR creation on a hosted provider — see workflow.md for the commit → PR → review cadence expected within each phase.

## Local LLM agents
- Code generation, testing, review, and style-checking are performed by role-specific agents backed by local models served through Ollama, orchestrated per phase by a Project Manager role — see workflow.md for the roles and cadence.
- Default model for every agent role: `qwen2.5-coder:32b`. Other locally-downloaded models can be substituted per role/project if noted in the project plan.

## Applying this
- Assume this stack for any new project scaffolding, dependency choices, or architecture suggestions.
- If a requested feature doesn't fit cleanly into this stack (e.g. needs a different DB, a queue other than Celery/Redis, a non-React frontend), flag the mismatch and ask before proceeding rather than silently substituting.
