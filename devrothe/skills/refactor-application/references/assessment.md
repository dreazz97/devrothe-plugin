# Project analysis

Goal: understand the current state without changing it and produce these outputs — app type, deviation
map against the target methodology, a list of bugs/errors and (for a real app) a list of security
findings. **Do not edit anything in this phase.**

## Contents
- Current state and app type
- Deviation map against the target methodology
- Bug/error detection
- Security findings (real app only)
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
| Dev infra | Docker Compose (Postgres, and MinIO/Keycloak if applicable); note the current local run mode — services-only vs a `Dockerfile`/`app` service that containerizes the app |
| Modules | auth (Keycloak/JWT httpOnly), storage (MinIO), observability, logging, payments, email |
| Security (real app) | secure-by-default baseline: validation, authz, headers, rate limiting, secrets hygiene (see security.md) |

Target details in `../create-application/references/web-stack.md`, `app-stack.md`, `modules.md`,
`testing.md`, `design.md` and `security.md`.

## Bug/error detection

Look for and record (with severity):
- **Blocking**: build fails, type errors (`tsc`), the app does not start, missing/broken dependencies.
- **Tests**: existing tests failing.
- **Quality/anti-patterns**: lint errors, dead code, duplicated logic, giant components/files, missing
  error handling at the boundaries.

Security red flags are recorded separately, in the next section.

## Security findings (real app only)

Gated on the security posture (see `../create-application/references/security.md` — for a PoC/demo skip
this beyond the repo-hygiene floor). Without modifying anything, surface the security deviations and
red flags so the plan can fix them like the bugs. This is **flagging during a refactor, not a full
security audit** — record what is plainly visible while reading the code. Look for:

- **Secrets**: hardcoded credentials/keys/tokens in code; secrets committed to git or its history;
  `.env`/`*.pem`/`*.key` not in `.gitignore`.
- **Injection**: SQL via string concat/interpolation; `eval`/`pickle`/unsafe `yaml.load`;
  `shell=True`/`os.system` on user input.
- **Access control**: protected routes without auth; `:id` access without an ownership/role check
  (IDOR); admin paths without a role gate.
- **Auth/crypto**: tokens in `localStorage`; JWT without `exp`/signature verification; passwords with
  MD5/SHA1 or unhashed; `Math.random()` for tokens.
- **Config**: open/`*` CORS; missing security headers; DEBUG on in prod; default credentials; stack
  traces returned to clients.
- **Input/SSRF/uploads**: missing boundary validation; server-side fetch of a user-controlled URL
  without an allowlist; uploads without type/size checks.
- **Dependencies**: known-vulnerable packages (quick `pnpm audit`/`pip-audit`); missing/committed
  lockfile issues.

Record each finding with a **severity** (CRITICAL/HIGH/MEDIUM/LOW — see `security.md`) and note whether
fixing it **changes observable behavior** (e.g., adding auth, locking CORS) — those need the deliberate
-change treatment in the plan.

## Analysis output

Summarize to the user: app type + facets, the deviation table (OK/deviation), the list of bugs/errors
by severity and — for a real app — the security findings by severity (marking the behavior-changing
ones). This output feeds the plan (restructure-plan.md).
