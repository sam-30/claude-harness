# claude-harness

Personal template for standard project conventions: Django + DRF backend, React + TypeScript frontend, Docker, Postgres, Redis/Celery, plus the phased-delivery and testing workflow described in `standards/workflow.md` and `standards/testing.md`.

## Usage

Copy `CLAUDE.md` and `standards/` into the root of a new project:

```bash
cp CLAUDE.md /path/to/new-project/
cp -r standards /path/to/new-project/
```

Claude Code auto-loads `CLAUDE.md` at the project root, which imports each file under `standards/` via `@standards/...` syntax.

If a project needs to deviate from a default (different retry limit, extra service, different frontend testing tool), add an override directly in that project's own `CLAUDE.md` rather than editing these shared files — keep this repo as the canonical, unmodified baseline.

## Updating the baseline

Edit files here, commit, and re-copy into projects as needed. This repo isn't symlinked into projects, so updates don't propagate automatically — that's intentional, since an in-flight project shouldn't have its ground rules change out from under it.
