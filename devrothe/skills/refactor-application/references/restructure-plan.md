# Restructuring plan

Turn the analysis output (assessment.md) into a phased plan, from low to high risk, preserving the
application's behavior.

## Contents
- Principles
- Phase order
- Structure of each phase
- Mandatory bugs/errors section
- Mandatory security findings section (real app)

## Principles

- **Preserve behavior**: refactors are functional equivalence; the only allowed behavior change is
  fixing the listed bugs.
- **Incremental**: small, verifiable slices; no "big bang".
- **Low risk first**: start with what is safe and unblocks the rest.
- **Heavy migrations are opt-in**: a framework swap (e.g., CRA → Vite, mass JS → TS) is flagged
  separately and only proceeds with explicit approval; prefer a phased (strangler) migration over
  rewriting everything at once.

## Phase order

Suggestion (adjust to the project and the detected facets):

1. **Baseline + safety net** — make sure you know how to start the app; create a branch/commit or
   backup (see execution.md).
2. **Blocking bugs** — fix what prevents build/startup, so you refactor over code that runs.
3. **Base tooling** — TypeScript, pnpm, ESLint + Prettier, `dev/build/lint/test` scripts.
4. **Stack/dependency alignment** — UI (Tailwind + shadcn/ui + lucide-react, framer-motion,
   next-themes, sonner), Zod + React Hook Form, TanStack Query, ORM (Prisma/SQLAlchemy) over Postgres.
   When there is UI, also align it to professional UI/UX and remove AI-slop tells (see
   `../create-application/references/design.md`); flag notable visual changes as deliberate plan items.
5. **Folder structure** — migrate to robust feature-first (see `../create-application/references/`
   `web-stack.md` or `app-stack.md`), moving the code by domain.
6. **Modules** — align auth (Keycloak/JWT httpOnly), storage (MinIO), observability, logging, payments,
   email as applicable (see `../create-application/references/modules.md`). Also align the **local run
   mode** (`compose` section): the dev services belong in Docker Compose; whether to containerize the
   **app** itself (a `Dockerfile` + `app` service) is the user's choice — offer the same three options
   as create-application (services-only/native, fully containerized, or both) and only add/keep the
   Dockerfile if the user wants it. Adding a Dockerfile is behavior-preserving (it does not change how
   the code runs), but make sure service hosts come from env so the app runs both natively and
   in-container.
7. **Security alignment (real app)** — close the security findings and align to the baseline (see
   `../create-application/references/security.md`): boundary validation, authorization/ownership
   checks, parameterized data access, security headers, locked CORS, rate limiting, secrets→env. Note:
   the CRITICAL secrets-in-code/history findings are pulled forward (rotate the secret; `.gitignore`
   does not remove history), and behavior-changing fixes carry the deliberate-change flag. Skip this
   phase entirely for a PoC/demo (beyond the repo-hygiene floor).
8. **Tests** — align/add the test layer (see `../create-application/references/testing.md`), including
   the security-relevant tests for what was hardened (real app).
9. **Remaining bugs/quality** — fix non-blocking bugs and remove dead code/anti-patterns.
10. **Docs** — update the `README.md` to reflect the final state (record the security posture).

## Structure of each phase

For each phase, record (and create a task with `TaskCreate`):
- **Goal** and files/areas affected.
- Concrete **changes**.
- **Validation** (how to confirm it still works: build/lint/tests/startup).
- **Risk / rollback** (what to do if it goes wrong).

## Mandatory bugs/errors section

The plan always includes a section with **all bugs/errors detected in the analysis**, each with:
severity, symptom, likely cause, the phase in which it will be fixed and how it will be validated.
Blocking ones go to phase 2; the rest to phase 9 (or to the phase of the module they belong to).

## Mandatory security findings section (real app)

For a real app (gated — skip for a PoC/demo), the plan includes a section listing **all security
findings from the analysis**, each with: severity (CRITICAL/HIGH/MEDIUM/LOW), the issue, the phase that
fixes it, how it will be validated, and **whether the fix changes observable behavior**.

- **Behavior-preserving fixes** (add headers, lock CORS down to the real origin, move a secret to env,
  parameterize a query) go in phase 7 (or the relevant alignment phase).
- **Behavior-changing fixes** (add auth where there was none, enforce validation that previously let
  bad input through, invalidate sessions on logout) are flagged as **deliberate changes** — like the
  bug fixes, they are explicit, approved, and may be offered for phase-by-phase approval.
- **CRITICAL secrets in code/history** are pulled forward and paired with a rotation note (a refactor
  cannot un-leak a committed secret).

This is the security counterpart of the bugs section: nothing security-relevant is changed silently.
