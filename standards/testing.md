# Testing Requirements

## Framework
All backend tests are written and run with **pytest** (pytest-django for Django integration). Do not use Django's built-in `unittest`-style `TestCase` runner as the primary test framework.

## Coverage
- Target as close to 100% test coverage as practically possible, on both backend and frontend.
- New code should not lower overall coverage. If a line/branch genuinely can't be tested (e.g. defensive code that can't be triggered), it's fine to exclude it explicitly (`# pragma: no cover`) with a one-line reason — don't leave gaps silently.
- Run coverage reporting (`pytest --cov`) as part of validating each phase, not just once at the end of the project.

## Phase gating
Every implementation phase must have its full pytest suite passing before moving on to the next phase. Never proceed to the next phase with failing or skipped tests. "Full suite" includes both the implementer's own tests and the independently spec-authored tests (see workflow.md's "Independent test authoring") — a phase isn't done until both pass.

If tests fail after implementing a phase:
1. Diagnose and attempt to fix the failure.
2. Retry up to **3 attempts total** (the initial run plus 2 fix attempts).
3. If tests are still failing after the 3rd attempt, stop. Summarize what's failing, what you already tried, and ask the user for guidance before continuing. Do not keep retrying indefinitely, and do not move on to the next phase with known-failing tests.

Once a phase's suite passes, its pull request (see workflow.md) must describe what was tested and how — this is what the independent reviewer checks the code against before merge.

A green suite is not the same as a working feature: automated tests exercise application code directly and don't catch session-state or environment-specific bugs (auth/token expiry being the canonical example) the way a real browser session does. For phases that ship or change user-facing behavior, the suite passing is necessary but not sufficient — see workflow.md's "Independent feature verification (browser click-through)" for the additional check required before merge.

(The retry limit is a default — a specific project can override it in its own CLAUDE.md if the user wants a different number.)

## Frontend
Frontend changes need corresponding tests (React Testing Library or equivalent) held to the same "must pass before moving on" bar, even though the retry protocol above is described in pytest terms.  Playwright should be used to confirm frontend functionality.
