---
name: create-application
description: Starts a new project end to end — interviews the user about requirements, picks the right tech stack, scaffolds it, plans the features and implements the project from the plan with real tests. Use when the user wants to start a new project, decide technologies/stack, or initialize and bootstrap a codebase. Triggers on "create-application", "start development", "new project", "start a project", "what stack should I use", "scaffold", and in Portuguese "iniciar desenvolvimento", "novo projeto", "começar um projeto", "que stack uso". Chooses between a Next.js stack (PoC, landing pages, e-commerce, marketing sites) and a React+Vite stack with an Express/Node or FastAPI/Python backend (web applications), with optional modules for authentication, storage (MinIO), observability (Kubernetes), logging, payments (Stripe) and email (Resend).
---

# create-application

Interview the user, resolve the stack, plan the features and implement the project from the plan —
with real tests.

The stack is not a free choice: there is a default stack per project type (defined below) to keep
projects consistent and avoid ad-hoc decisions. Only propose a different technology if the user
explicitly asks for it or a requirement makes the default unworkable — and in that case, explain why.

**Rule: keep `README.md` up to date.** The scaffold generates a README and, from then on, any change
to the stack, dependencies, environment variables or run commands must be reflected in the README in
the same task. Why: the README is the single source of truth for onboarding and for running the
project — an outdated README is worse than none.

**Rule: robust, feature-first folder structure.** Every generated solution follows the structure
defined in its stack reference file — organized by feature/domain, with clear separation of layers
(UI, server logic, data access) and predictable locations for schemas (Zod/Pydantic), types, config
and tests. Why: predictable locations and explicit boundaries keep the project navigable as it grows,
avoid giant files and prevent accidental coupling. Do not use ad-hoc structures nor scatter domain
logic across generic technical folders when an obvious feature exists. The reference structures are
the minimum — create the folders during scaffold even if they start empty (with a `.gitkeep`), to
lock in the convention from the start.

**Rule: real tests accompany every feature.** Every implemented feature ships with tests that
exercise its behavior (see `references/testing.md`) — the test config and folders created during
scaffold are not enough. Why: the test layer provisioned by the scaffold is just scaffolding; without
tests that assert behavior there is no safety net, and a "green" but empty test gate is theater. A
green test suite is a completion gate (step 7).

**Rule: professional UI/UX, not the generic AI look.** For any UI, follow `references/design.md` —
respect UI/UX fundamentals (visual hierarchy, consistent spacing scale, deliberate type scale, WCAG AA
contrast, focus/keyboard states, responsiveness, purposeful motion) and avoid the tells of
AI-generated design (emojis as UI, cliché purple→blue gradients, blindly-applied boilerplate hero +
identical 3-card grids, untouched stock defaults, buzzword filler copy). Why: the default "AI slop"
aesthetic looks mass-produced and unprofessional — deliberate, restrained design is the baseline.

**Rule: secure-by-default — unless it is a PoC/demo.** For a real application, build in the security
baseline from `references/security.md` (input validation at the boundary, authorization/ownership
checks, parameterized data access, safe auth/secrets, security headers + HTTPS, rate limiting on
sensitive endpoints, OWASP Top 10 awareness). This is **gated**: if the project is a throwaway
PoC/demo, skip the hardening (the gate in `references/security.md` says how to decide, and to **ask the
user when it is ambiguous**); only the minimum repo-hygiene floor (never commit real secrets/data)
still applies. Why: hardening is cheap up front and ruinous to retrofit on a real app, but pure
friction on a disposable prototype — so the PoC/demo decision drives it, and a wrong guess is costly
either way.

## Workflow

Copy this checklist into the response and tick items off:

```
- [ ] 0. Context — read CLAUDE.md, memory and READMEs/docs of the project (if any)
- [ ] 1. Interview — stack requirements + functional scope (features, entities, flows) + security posture (PoC/demo or real app)
- [ ] 2. Resolve the stack from the answers (+ the security baseline if it is a real app)
- [ ] 3. Plan — design the features/tasks to build
- [ ] 4. Confirm — stack + plan + security posture with the user (nothing is written/installed before the "yes")
- [ ] 5. Scaffold — structure, dependencies, configs and README (security baseline if applicable)
- [ ] 6. Implementation — feature by feature, driven by the plan, with real tests (secure-by-default if applicable)
- [ ] 7. Verify — lint, build, startup AND a green test suite (completion gate); security checkpoint if applicable
```

### 0. Project context

Before moving on, look for and read metadata that helps understand the context and preferences:
- `CLAUDE.md` at the root and in subfolders, and other agent instruction files (`AGENTS.md`,
  `.cursor/rules/`, `.github/copilot-instructions.md`).
- The project **memory** (preferences and decisions already recorded).
- Text files: `README*`, `docs/`, `CONTRIBUTING.md`, `ARCHITECTURE.md`, ADRs and notes.

Use this context to inform the interview, the plan and the implementation. Why: respecting conventions
and decisions already documented avoids contradicting what the user/team has set. In an empty folder
there is usually nothing to read — move on; if there is already a `CLAUDE.md` with preferences,
respect it.

### 1. Interview

Use the `AskUserQuestion` tool. Gather two blocks:

**(a) Functional scope** — what the application does: main features, core entities/data, key user
flows and mandatory pages/endpoints. This feeds the plan (step 3) and the implementation (step 6);
without functional scope, the skill can only scaffold.

**(b) Stack decisions** — start with the project-type question (decides the base stack) and then the
conditional ones. Group related questions in the same batch to reduce back-and-forth.

**Key question (decides the base stack):**

| Answer | Base stack |
|--------|-----------|
| PoC, landing page, e-commerce, institutional/marketing site, blog, site with little client-side interactivity | **Next.js** — see `references/web-stack.md` |
| Interactive web application: dashboard, SaaS, app with rich client state, internal tool | **React + Vite + backend** — see `references/app-stack.md` |

If the boundary is ambiguous (e.g., e-commerce with a complex customer area), prefer Next.js — the App
Router covers SSR/SEO and interactivity. Reserve React+Vite for cases where the frontend is clearly a
SPA decoupled from its own backend.

**Conditional questions** (each "yes" activates a module from `references/modules.md`):

1. Does it need data persistence? → **PostgreSQL** (on by default; only turn off for 100% static
   sites). ORM: **Prisma** if the backend is Node/Next.js; **SQLAlchemy + Alembic** if Python/FastAPI.
2. Will it store images or videos? → **MinIO** (`storage` module).
3. Does it have user authentication? If so, ask the approach with `AskUserQuestion` (single choice):
   **Keycloak (OIDC)**, **own JWT in httpOnly cookies**, or **let the AI choose**. See the `auth`
   module in `references/modules.md` (includes the decision criteria when the AI chooses).
4. Will it be deployed on Kubernetes? → **Prometheus + OpenTelemetry** (`observability` module).
5. Want structured/advanced logging? → **Pino** (Node) or **structlog** (Python) (`logging` module).
6. Does it process payments? → **Stripe** (`payments` module).
7. Does it send transactional email (account verification, password reset, orders)? → **Resend**
   (`email` module).

If the backend is Python/FastAPI, also ask whether Express/Node is preferred — otherwise decide based
on the project (see `references/app-stack.md` for the criteria).

**Security posture** — determine whether this is a real application or a throwaway PoC/demo, because it
decides whether the security baseline is built in (see the gate in `references/security.md`). Infer it
from the functional scope first: real users, authentication, payments, PII or a production deployment
mean "real app — apply security"; an explicit prototype/demo/spike with only fake data and no exposure
means "PoC — skip hardening". **If it is ambiguous, ask** with `AskUserQuestion` (single choice:
*"Real application — apply the security baseline"* vs *"PoC/demo — skip it"*). Note: "PoC" as the
*project type* (Next.js base stack) does not by itself mean PoC for security — an e-commerce or a site
handling real data is a real app even on the Next.js base.

### 2. Resolve the stack

Combine the base stack with the activated modules. Always apply the cross-cutting defaults below. If
the security posture is "real app", also fold in the security baseline from `references/security.md`
(headers, input validation, authorization, rate limiting, secrets hygiene) — woven across the
features, not bolted on. Record the posture decision in the README/CLAUDE.md (step 5) so the other
skills honor it later.

### 3. Plan

From the functional scope, design a development plan: split it into **small vertical slices** and
order them by dependency (typically: data model → authentication → domain features → UI/integrations).
For each slice, note what goes in and which tests cover it. Record the plan as tasks with `TaskCreate`
to track progress during implementation.

For a real app, treat security as **cross-cutting** rather than a separate slice: each slice that
handles input, persistence, auth or external calls carries its own validation, authorization and tests
(see `references/security.md`). Call out in the plan any slice with notable security weight (auth,
payments, uploads, anything touching PII) so its hardening and tests are explicit.

### 4. Confirm

Present to the user the resolved stack (language, framework, DB/ORM, UI, modules) **and** the feature
plan, as a short summary, and ask for confirmation **before** creating or installing anything. The
scaffold and the implementation write files and install dependencies — do not proceed without the
"yes".

### 5. Scaffold

Follow the base-stack reference file (`web-stack.md` or `app-stack.md`) and, for each activated module,
the matching section of `modules.md`. Run the init commands, install the dependencies (including the
test ones), generate the base configs and create the robust folder structure.

Generate a `README.md` at the project root with, at minimum: a short description, the chosen stack,
prerequisites, environment variables (`.env`), how to bring up the dev services (Docker Compose), how
to install and run, and how to run tests and lint. Record the **security posture** here (or in
CLAUDE.md) so later work honors it. Keep it in sync on any future change (see rule above).

For a real app, set up the security baseline during scaffold (see `references/security.md`, "Per-stack
setup"): security-headers config (Next.js `headers()`/middleware; `helmet` on Express; a headers
middleware on FastAPI), CORS locked to the frontend origin, a rate-limit dependency for auth endpoints,
boundary validation wiring (Zod/Pydantic), and `.gitignore` covering `.env`/`*.pem`/`*.key`/`secrets/`
**before the first commit**. The minimum repo-hygiene floor (no committed secrets/real data) applies
even to PoCs.

### 6. Implementation

Implement the project from the plan (step 3), slice by slice, following the stack's folder structure.
For each slice:

1. Implement the feature's code in the right folders, respecting layers and boundaries.
2. Write **real tests** at the appropriate level (see `references/testing.md`): unit for
   logic/services, integration for endpoints, component for UI with logic, e2e for critical flows.
3. Run the slice's tests and iterate (write → run → fix) until they pass.
4. Update the tasks (`TaskUpdate`) and the README as needed before moving to the next slice.

For any UI built in a slice, follow `references/design.md` (professional UI/UX, no AI-slop tells). For
a real app, build each slice secure-by-default (`references/security.md`): validate input at the
boundary, enforce authorization/ownership on protected routes, use parameterized data access, and add
the security-relevant tests (e.g., a denied cross-user access, a rejected malformed payload).

Do not mark a slice as done with its tests failing. Cover the happy path and the relevant
errors/edge cases of each feature — do not leave features half-done.

### 7. Verify

The task is only done when, in the generated project: dependency install passes, lint passes, build
passes, the app starts, **and the full test suite is green**. Run the tests as the final gate; if any
fails (or is improperly skipped/xfail), fix it before calling it done. Also confirm the README
reflects the real state.

For a real app, add a quick **security checkpoint** before calling it done (see `references/security.md`):
the baseline is in place (headers, locked CORS, boundary validation, authz on protected routes, rate
limiting on auth), no secrets are committed and `.gitignore` covers them, the dependency audit
(`pnpm audit`/`pip-audit`) surfaces no unresolved known-vulnerable packages, and the security-relevant
tests pass. For a PoC/demo, skip this checkpoint beyond the repo-hygiene floor.

**Manual testing disclaimer (always tell the user).** A green suite is not the whole picture: some
checks are still recommended to be done manually — especially **UI/UX interaction in a real browser**
(visual layout, responsiveness, animations, accessibility, real flows), since graphical aspects are
hard to verify reliably with AI. List the concrete manual checks worth doing (see
`references/testing.md`).

## Cross-cutting defaults (always, without asking)

- **TypeScript** in all JS code.
- **pnpm** as the package manager.
- **ESLint + Prettier** (with `prettier-plugin-tailwindcss`) for lint and formatting.
- **Tailwind CSS + shadcn/ui + lucide-react** for UI; **framer-motion** (animations), **next-themes**
  (light/dark theme) and **sonner** (toasts).
- **Zod + React Hook Form** (`@hookform/resolvers`) for schema and form validation.
- **TanStack Query** for client-side server-state.
- **Docker Compose** for the dev services (PostgreSQL and, if applicable, MinIO).
- **Tests**: Vitest + @testing-library/react + Playwright (Node/JS) and pytest + httpx (Python). The
  layer is installed and configured during scaffold and **filled with real tests** during
  implementation — see `references/testing.md`.

## References

- **`references/web-stack.md`** — full scaffold of the Next.js stack (PoC, landing pages, e-commerce,
  marketing sites).
- **`references/app-stack.md`** — scaffold of the React + Vite stack with an Express/Node or
  FastAPI/Python backend, and the criteria for choosing the backend.
- **`references/testing.md`** — testing strategy: what to test per layer, location conventions, tools
  per stack and the green test gate. Read during implementation (step 6).
- **`references/design.md`** — professional UI/UX rules and anti-AI-slop guidance. Read when building
  any UI (step 6).
- **`references/security.md`** — the PoC/demo gate and the secure-by-default baseline (OWASP Top 10,
  secrets/deps, per-stack setup, security testing, severities). Read in steps 1–2 (gate) and 5–7
  (apply) for a real app.
- **`references/modules.md`** — conditional modules: `auth`, `storage` (MinIO), `observability`
  (Kubernetes), `logging`, `payments` (Stripe), `email` (Resend). Read only the sections of the
  modules activated in the interview.
