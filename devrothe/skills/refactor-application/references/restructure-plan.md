# Restructuring plan

Turn the analysis output (assessment.md) into a phased plan, from low to high risk, preserving the
application's behavior.

## Contents
- Principles
- Phase order
- Structure of each phase
- Mandatory bugs/errors section

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
5. **Folder structure** — migrate to robust feature-first (see `../create-application/references/`
   `web-stack.md` or `app-stack.md`), moving the code by domain.
6. **Modules** — align auth (Keycloak/JWT httpOnly), storage (MinIO), observability, logging, payments,
   email as applicable (see `../create-application/references/modules.md`).
7. **Tests** — align/add the test layer (see `../create-application/references/testing.md`).
8. **Remaining bugs/quality** — fix non-blocking bugs and remove dead code/anti-patterns.
9. **Docs** — update the `README.md` to reflect the final state.

## Structure of each phase

For each phase, record (and create a task with `TaskCreate`):
- **Goal** and files/areas affected.
- Concrete **changes**.
- **Validation** (how to confirm it still works: build/lint/tests/startup).
- **Risk / rollback** (what to do if it goes wrong).

## Mandatory bugs/errors section

The plan always includes a section with **all bugs/errors detected in the analysis**, each with:
severity, symptom, likely cause, the phase in which it will be fixed and how it will be validated.
Blocking ones go to phase 2; the rest to phase 8 (or to the phase of the module they belong to).
