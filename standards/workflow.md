# Development Workflow

## Phased delivery
- Break every non-trivial project into explicit phases (e.g. "Phase 1: models + migrations", "Phase 2: DRF serializers/viewsets", "Phase 3: frontend scaffold + auth", ...). Phases should be small enough that each is independently testable.
- Before writing any code, produce a written project plan listing the phases, what each phase delivers, and what "done" looks like for that phase (including what the tests will cover).
- **Do not start writing code until the user has explicitly confirmed the plan.** Use plan mode (or an equivalent explicit check-in) rather than assuming approval from silence or a vague go-ahead.

## Executing a phase
1. Implement the phase.
2. Write/update tests for the phase (see testing.md for the coverage bar).
3. Run the full pytest suite. All tests must pass before the phase is considered complete.
4. On failure, follow the retry protocol in testing.md (up to 3 attempts, then ask the user for help).
5. Once a phase's tests pass, briefly summarize what was completed and move to the next phase — don't silently re-plan or expand scope mid-phase.

## Mid-project changes
If new information changes the phase plan (a new requirement, a discovered blocker), surface the change and get confirmation before altering the plan, rather than quietly re-scoping.
