# Test plan per facet

What to cover in each facet identified in the analysis (app-analysis.md). Build the plan only for the
facets present. Real tests that exercise behavior — happy path + errors/edge cases.

## Contents
- What to test per facet
- Plan structure
- Principles

## What to test per facet

- **Frontend / UI**
  - Components with logic → component tests (Testing Library): user interaction, states, conditional
    render — not the internal structure nor empty snapshots.
  - Critical flows (login, checkout, create/edit resource) → e2e (Playwright/Cypress).
- **Backend / API**
  - Endpoints → integration tests (supertest/httpx) against a test DB: status, payload, validation,
    authorization.
  - Business logic / services / utils → unit tests (no I/O; mock boundaries).
- **Microservice**
  - API/message contracts (request/response, event schemas).
  - Message handlers (consume → process → produce/effect).
  - Integration with dependencies via testcontainers (broker, DB) when applicable.
  - Health/readiness endpoints.
- **Service / monolith** — like the backend, with emphasis on boundaries between modules.
- **CLI / library** — unit tests of the public API; CLI invocation tests (args, exit codes, output).
- **Mobile** — component/widget tests + e2e (Detox/Maestro).
- **Data / ML** — transformations and pure functions; validation of data contracts/shapes.

## Plan structure

Present a table per facet before asking for approval:

| Facet | Target / slice | Test type |
|-------|----------------|-----------|
| API | `POST /orders` (creation + validation) | integration |
| API | `OrderService.calcTotal` | unit |
| UI | checkout flow | e2e |

Record the slices as tasks (`TaskCreate`) and implement them one by one.

## Principles

- Use the framework already present in the project; if there is none, install the idiomatic one (see
  `detect-and-run.md`) and configure it.
- For DB integration, use a disposable test database and clean the state between tests; never run
  against the development DB.
- Prioritize behavior that matters instead of chasing 100% coverage.
- Do not disable tests with `skip`/`xfail`/`.only` without explicit justification to the user.
