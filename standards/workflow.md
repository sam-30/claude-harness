# Development Workflow

## Phased delivery
- Break every non-trivial project into explicit phases (e.g. "Phase 1: models + migrations", "Phase 2: DRF serializers/viewsets", "Phase 3: frontend scaffold + auth", ...). Phases should be small enough that each is independently testable.
- Before writing any code, produce a written project plan listing the phases, what each phase delivers, and what "done" looks like for that phase (including what the tests will cover).
- **Do not start writing code until the user has explicitly confirmed the plan.** Use plan mode (or an equivalent explicit check-in) rather than assuming approval from silence or a vague go-ahead.

## Local LLM agent roles
- Code generation, testing, review, and style-checking for a phase are carried out by role-specific agents backed by local models served through Ollama, rather than by whatever is driving the overall session.
- A **Project Manager** role owns the confirmed phase plan and invokes each specialized agent below for a phase. The Project Manager can be Claude Code, another AI coding harness, or the user driving the invocations by hand — this standard doesn't assume a specific driver, only that something is responsible for sequencing the roles and gating a phase's completion on their output.
- Roles the Project Manager invokes:
  - **Engineer** — implements the phase (see "Executing a phase" below).
  - **Tester** — writes the phase's tests, including the independently spec-authored tests (see "Independent test authoring").
  - **PR Reviewer** — performs the independent review before merge (see "Independent review").
  - **Style checker** — performs the styling pass (see "Styling").
- Ollama has multiple models downloaded locally. Default every role to `qwen2.5-coder:32b` unless a project or role calls for a different model (e.g. a smaller/faster model for style-checking) — note any override in the project plan.
- This section only establishes who does the invoking and what model backs each call; the responsibilities and pass/fail gating for each role stay defined in the sections referenced above.

## Executing a phase
1. Implement the phase. In parallel, kick off independent test authoring (see below) so it isn't gated on implementation finishing first.
2. Write/update tests for the phase (see testing.md for the coverage bar).
3. Run the full pytest suite, including the independently-authored tests once they land. All tests must pass before the phase is considered complete.
4. On failure, follow the retry protocol in testing.md (up to 3 attempts, then ask the user for help).
5. Once a phase's tests pass, briefly summarize what was completed and move to the next phase — don't silently re-plan or expand scope mid-phase.

## Independent test authoring
- For each phase, alongside the implementer's own tests (step 2 above), a separate model or agent — given only the phase's entry from the confirmed project plan (its scope and "done"/test criteria), never the implementation code or diff — writes its own tests for the phase from the spec alone. This supplements the implementer's tests; it doesn't replace them.
- Keep the two sets distinguishable (e.g. a separate `test_phase_N_spec.py` file or describe block) so the PR makes clear which tests came from the spec-only agent versus the implementer.
- Both suites must pass against the final implementation before the phase is done (see "Executing a phase"). A failure in the spec-authored tests is a real signal, not noise — it means the implementation doesn't match the plan, the spec-writing agent misread the plan, or the plan itself was ambiguous. Diagnose which before touching the test: don't quietly edit a spec-written test to match a possibly-wrong implementation.
- If the mismatch traces back to an ambiguous or underspecified phase plan, surface it and get it clarified (see "Mid-project changes") rather than resolving the ambiguity unilaterally.
- Note in the PR body (see "Committing & pull requests") that the independent spec-based tests passed, alongside the implementer's own test description.

## Committing & pull requests
- Once a phase's tests pass, commit its changes with plain `git` (`git add` / `git commit`), then push and open a pull request through the `forge` CLI against [`sdlc-tracker`](https://github.com/sam-30/sdlc-tracker) (see infrastructure.md for how the forge is run/reached): `forge pr create <org>/<repo> --source <branch> --target main --title "..." --body "..."`.
- The PR body must describe what was tested and how — which tests were added/changed and what behavior they verify against the phase's "done" criteria (see testing.md) — not just "added tests for phase N." A reviewer who never saw the phase plan should be able to tell from the PR body alone what's being claimed and how to check it.
- Do not merge your own PR (`forge pr merge`) immediately after opening it — see Independent review below.

## Independent review
- Before a phase's PR is merged, an independent model or agent — one that did not write the code under review — must read the diff and the PR's test description, and confirm the code actually does what the PR claims and that the described tests genuinely cover it.
- In practice: run a separate review pass (e.g. `/code-review`, or a fresh subagent with no prior context on the change) against the PR before calling `forge pr merge`. Once sdlc-tracker's own automated PR-review bot is available, PRs can route through it instead.
- If the review finds a mismatch between the PR's description and the actual code or tests, fix it and re-review before merging. Never merge on a failed, skipped, or self-performed review.

## Mid-project changes
- If new information changes the phase plan (a new requirement, a discovered blocker), surface the change and get confirmation before altering the plan, rather than quietly re-scoping.
- If a new phase is added, or an existing phase has to change, ensure the project plan, markdown files, and any execution plan is updated first before code is written that changes the phase

## Styling
- Have a pass at the end to ensure the styling of the application matches the overall plan.  
- For example, a dog grooming website should have a style that is fun and colorful.  A corporate application that tracks cyber security should be more professional and clean.  

## Example phases
Use the following as an example of how phases can be created to break up building features and separating the project build into understandable chunks.  If the example app is a to-do list application, the project phases can be:

- **Phase 0 — Foundations & scaffolding**: repo skeleton, docker-compose stack (web, frontend, db, redis), Django + DRF + Celery wiring, React + Vite scaffold, CI. Done = `docker compose up` gives a running admin plus a React page hitting `/api/health/`, and CI is green.
- **Phase 1 — Auth & core domain models**: custom User model, JWT auth, the `Task` and `List` models + migrations, Django admin registration. Done = migrations apply cleanly, admin CRUD works, auth endpoints are tested.
- **Phase 2 — Task CRUD API**: DRF serializers/viewsets for creating, listing, updating, completing, and deleting tasks, scoped per-user. Done = full CRUD test coverage, including permission checks (a user can't touch another user's tasks).
- **Phase 3 — Business logic**: due dates, recurring tasks, reminders via Celery beat, list sharing/collaborators. Done = scheduled jobs tested with time-freezing, sharing permissions enforced and tested.
- **Phase 4 — React frontend**: list/task views, create/edit/complete flows, optimistic updates. Done = component tests plus at least one end-to-end flow (e.g. RTL/Playwright) covering create → complete.
- **Phase 5 — Hardening & full-suite pass**: coverage gaps closed, error states handled, rate limiting, observability (e.g. Sentry). Done = coverage target hit, full suite green.
- **Phase 6 - Style Review**: Reviews the css and styling of the application to ensure the style matches the features.  
- **Phase 7 - Mobile**: Port features for mobile using PWA.
- **Phase 8 — Deployment**: prod Docker images, managed Postgres/Redis, migrations run safely as part of CI/CD.

Each phase should be committed and opened as a pull request per "Committing & pull requests" above, with an independent review before merging, before moving to the next phase.
