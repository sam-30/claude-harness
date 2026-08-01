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

## Applying this
- Assume this stack for any new project scaffolding, dependency choices, or architecture suggestions.
- If a requested feature doesn't fit cleanly into this stack (e.g. needs a different DB, a queue other than Celery/Redis, a non-React frontend), flag the mismatch and ask before proceeding rather than silently substituting.
