# Infrastructure

## Containers
Every project is fully docker-ized. At minimum, docker-compose should define:
- `web` (or `backend`) — Django/DRF app
- `frontend` — React/TypeScript app (dev server or build step, depending on stage)
- `db` — PostgreSQL
- `redis` — Redis
- `celery-worker` — Celery worker process
- `celery-beat` — Celery beat scheduler (only if the project actually has scheduled tasks)

## Conventions
- Every service should be runnable with a single `docker compose up`.
- Environment variables belong in `.env` (gitignored) with a checked-in `.env.example` documenting every required variable.
- Database migrations run via `docker compose exec web python manage.py migrate` — never against a local, non-containerized Postgres.
- Never commit secrets, API keys, or `.env` files.

## When scaffolding a new project
- Start from a docker-compose.yml covering the services above, sized to what the project actually needs (e.g. skip `celery-beat` if there's nothing to schedule).
- Confirm the compose stack boots (`docker compose up --build`) before considering infrastructure "done" for a phase.
