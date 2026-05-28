# Project analysis

Goal: understand the current state without changing it and produce three outputs — app type, deviation
map against the target methodology and a list of bugs/errors. **Do not edit anything in this phase.**

## Contents
- Current state and app type
- Deviation map against the target methodology
- Bug/error detection
- Analysis output

## Current state and app type

- Detect the language(s), dependency manager, frameworks and scripts (manifests, lockfiles).
- Classify the app type and facets with `../test-application/references/app-analysis.md`
  (UI/frontend, backend/API, microservice, service, CLI/library, fullstack, etc.).
- Try to start/install to learn the baseline (does it build? does it start? do existing tests pass?).

## Deviation map against the target methodology

Compare the project with the `create-application` destination. For each item, mark **OK / deviation**:

| Dimension | Target |
|-----------|--------|
| Language | TypeScript in all JS |
| Package manager | pnpm |
| Lint/format | ESLint + Prettier (+ prettier-plugin-tailwindcss) |
| Base framework | Next.js (sites/PoC) or React+Vite + backend (web apps) |
| UI | Tailwind + shadcn/ui + lucide-react, framer-motion, next-themes, sonner |
| Forms/validation | Zod + React Hook Form |
| Server-state | TanStack Query |
| ORM / DB | Prisma (Node) or SQLAlchemy+Alembic (Python) over PostgreSQL |
| Folder structure | robust feature-first (see web-stack.md / app-stack.md) |
| UI/UX design | professional, accessible; no AI-slop tells (see design.md) |
| Tests | real layer (see testing.md) |
| Dev infra | Docker Compose (Postgres, and MinIO/Keycloak if applicable) |
| Modules | auth (Keycloak/JWT httpOnly), storage (MinIO), observability, logging, payments, email |

Target details in `../create-application/references/web-stack.md`, `app-stack.md`, `modules.md`,
`testing.md` and `design.md`.

## Bug/error detection

Look for and record (with severity):
- **Blocking**: build fails, type errors (`tsc`), the app does not start, missing/broken dependencies.
- **Tests**: existing tests failing.
- **Quality/anti-patterns**: lint errors, dead code, duplicated logic, giant components/files, missing
  error handling at the boundaries.
- **Security (red flags)**: hardcoded secrets, tokens in `localStorage`, SQL via string concatenation,
  open CORS, missing input validation. (Flag them; this is not a full audit.)

## Analysis output

Summarize to the user: app type + facets, the deviation table (OK/deviation) and the list of
bugs/errors by severity. This output feeds the plan (restructure-plan.md).
