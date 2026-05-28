# Testing strategy

Read during implementation (step 6 of SKILL.md). Every feature built ships with real tests that
exercise its behavior. A test that does not fail when the behavior breaks is not a test — avoid empty
snapshots, trivial asserts (`expect(true).toBe(true)`) and mocks that return the expected result
without exercising the real logic.

## Contents
- What to test per layer
- Conventions and location
- Tools per stack
- Implementation loop
- Completion gate

## What to test per layer

- **Business logic / services / utils** → **unit** tests: fast, no I/O; mock only the boundaries (DB,
  network). Cover the happy path + relevant errors/edge cases.
- **Endpoints / route handlers** → **integration** tests: call the real app (supertest/httpx) against
  a test DB. Check status, payload, validation and authorization.
- **UI components with logic** → **component** tests (Testing Library): user interaction and
  state/render changes, not the internal structure.
- **Critical user flows** (login, checkout, create/edit resource) → **e2e** (Playwright).

Prioritize behavior that matters; do not chase 100% coverage. Each feature in the plan (step 3) should
come out of implementation with at least its main-layer tests passing.

## Conventions and location

- **Frontend (Next.js, React+Vite)**: unit/component in `tests/unit/` (or colocated as `*.test.tsx`
  next to the code); e2e in `tests/e2e/`.
- **Backend (Express)**: unit in `tests/unit/`; integration (supertest over `app.ts`) in
  `tests/integration/`.
- **Backend (FastAPI)**: unit in `tests/unit/`; integration (httpx over the app) in
  `tests/integration/`.
- Names: `<target>.test.ts` / `<target>.spec.ts` (JS/TS); `test_<target>.py` (Python).

## Tools per stack

| Stack | Unit / component | Integration | E2E |
|-------|------------------|-------------|-----|
| Next.js / React+Vite | Vitest + @testing-library/react + jsdom | — | Playwright |
| Express/Node | Vitest | supertest | (e2e in the frontend) |
| FastAPI/Python | pytest | httpx (TestClient/AsyncClient) | (e2e in the frontend) |

For DB integration, use a disposable test database (e.g., a dedicated Docker Compose Postgres service
or an isolated schema) and clean the state between tests. Do not run tests against the dev DB.

## Implementation loop

For each slice of the plan: write the code → write/adjust the tests → run → fix → repeat until it
passes. Only then move to the next slice. Keep the suite compiling at all times.

## Completion gate

A feature's implementation is only done when its tests pass. The overall task is only done when the
**full suite is green**, plus lint and build. Never mark as done with failing tests, nor disable them
with `skip`/`xfail`/`.only` without explicit justification to the user.
