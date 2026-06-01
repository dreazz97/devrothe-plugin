# Test plan per facet

What to cover in each facet identified in the analysis (app-analysis.md). Build the plan only for the
facets present. Real tests that exercise behavior — happy path + errors/edge cases.

## Contents
- What to test per facet
- Security cases per facet (real app)
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

## Security cases per facet (real app)

Gated on the security posture (see `../create-application/references/security.md` — skip for a
PoC/demo). For a real app, add these alongside the functional cases, prioritizing the critical paths
(auth, payments/billing, PII, tenancy):

- **Backend / API**
  - **Access control**: an authenticated user is denied another user's resource (IDOR); an
    unauthenticated request to a protected route returns 401/403; a role-gated route rejects an
    insufficient role.
  - **Input validation**: malformed/oversized/unexpected payloads are rejected with a 4xx (not a 500
    and not a silent accept).
  - **Auth flows**: login rate limit/lockout triggers; logout invalidates the session; expired/tampered
    tokens are rejected.
  - **Webhooks** (payments/integrations): an invalid signature is rejected.
- **Frontend / UI** — a protected route/component redirects or hides when unauthenticated/unauthorized;
  forms reject invalid input and surface the error (no broken/blank state).
- **Microservice / Service** — authz on protected handlers; message/contract validation rejects
  malformed events; no secret leakage in responses.
- **CLI / library** — input validation on the public API; no injection via passed-through arguments.

These are **behavior** tests, like the rest — they assert that a bad request is actually refused, not
that a config value exists.

## Plan structure

Present a table per facet before asking for approval:

| Facet | Target / slice | Test type |
|-------|----------------|-----------|
| API | `POST /orders` (creation + validation) | integration |
| API | `OrderService.calcTotal` | unit |
| API | `GET /orders/:id` denies another user's order (IDOR) | integration (security) |
| UI | checkout flow | e2e |

Record the slices as tasks (`TaskCreate`) and implement them one by one.

## Principles

- Use the framework already present in the project; if there is none, install the idiomatic one (see
  `detect-and-run.md`) and configure it.
- For DB integration, use a disposable test database and clean the state between tests; never run
  against the development DB.
- Prioritize behavior that matters instead of chasing 100% coverage.
- Do not disable tests with `skip`/`xfail`/`.only` without explicit justification to the user.
