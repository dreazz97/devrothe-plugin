# Devrothe

Plugin of **development-support skills** for [Claude Code](https://code.claude.com).
Distributed as a marketplace from GitHub (`dreazz97/devrothe-plugin`).

Current state: 3 skills — [`create-application`](#skill-create-application),
[`test-application`](#skill-test-application) and [`refactor-application`](#skill-refactor-application).

---

## Contents

```
Devrothe Plugin/
├── .claude-plugin/marketplace.json     # marketplace manifest
└── devrothe/                           # plugin
    ├── .claude-plugin/plugin.json
    └── skills/
        ├── create-application/
        │   ├── SKILL.md                # interview + plan + scaffold + implementation
        │   └── references/
        │       ├── web-stack.md        # Next.js stack
        │       ├── app-stack.md        # React+Vite + backend stack
        │       ├── testing.md          # testing strategy
        │       └── modules.md          # conditional modules
        ├── test-application/
        │   ├── SKILL.md                # detect/run tests or analyze + create + report
        │   └── references/
        │       ├── app-analysis.md     # classify the application type
        │       ├── detect-and-run.md   # detect and run tests per ecosystem
        │       └── test-plan.md        # what to test per facet
        └── refactor-application/
            ├── SKILL.md                # analyze + restructuring plan + execute + validate
            └── references/
                ├── assessment.md       # analysis: app type, deviations and bugs
                ├── restructure-plan.md # phased restructuring plan
                └── execution.md        # safe execution and validation
```

---

## Installation

From Claude Code:

```
/plugin marketplace add dreazz97/devrothe-plugin
/plugin install devrothe@devrothe
```

The install format is `plugin@marketplace` — here both are called `devrothe`.

### Updating after changes

After new commits in the repository:

```
/plugin marketplace update devrothe
```

### Local development (optional)

To iterate on the plugin without going through GitHub, add the marketplace by local path:

```
/plugin marketplace add "/Volumes/Samsung PSSD T7 Shield Media/Novlok/Devrothe Plugin"
```

---

## Commands / Skills

| Command | What it does |
|---------|--------------|
| `/create-application` | Interviews, picks the stack, plans, scaffolds and implements the project with real tests. |
| `/test-application` | Detects and runs the app's tests and reports; if there are none, analyzes the app, creates tests per facet and reports. |
| `/refactor-application` | Analyzes an existing app, plans the restructuring toward the `create-application` stack/practices, executes (with approval) and validates with tests. |

The skills also trigger via natural language (see the triggers below) — using the slash command is not
mandatory.

---

## Skill: `create-application`

Starts a new project end to end: asks questions, resolves the stack, plans the features, confirms with
you and then **initializes and implements the project from the plan — with real tests**. It does not
stop at the scaffold: it builds the app feature by feature and only considers the task done with a
green test suite.

### How to invoke

- Command: `/create-application`
- Trigger phrases: *"new project"*, *"start development"*, *"start a project"*, *"what stack should I
  use"*, *"scaffold"* (plus the Portuguese equivalents).

### What it will ask you

First the **functional scope** — what the app does: main features, core entities/data, key flows and
mandatory pages/endpoints (this feeds the plan and the implementation). Then the stack decisions:

1. **Project type** (decides the base stack):
   - PoC, landing page, e-commerce, institutional site/blog → **Next.js**
   - Interactive web application (dashboard, SaaS, internal tool) → **React + Vite + backend**
2. **Data persistence?** → PostgreSQL (on by default)
3. **Image/video storage?** → MinIO
4. **User authentication?** → choice (radio): Keycloak (OIDC), own JWT in httpOnly cookies, or let the AI choose
5. **Deploy on Kubernetes?** → Prometheus + OpenTelemetry
6. **Structured logging?** → Pino (Node) / structlog (Python)
7. **Payments?** → Stripe
8. **Transactional email?** → Resend

### Stacks it chooses

**Next.js** (PoC / traditional sites):
Next.js · TypeScript · Tailwind CSS + shadcn/ui + lucide-react · framer-motion · next-themes · sonner ·
Zod + React Hook Form · TanStack Query · Prisma + PostgreSQL.

**React + Vite + backend** (web applications):
React + Vite with the same UI stack above, plus an **Express/Node** or **FastAPI/Python** backend
(decided based on the project). ORM: **Prisma** (Node) or **SQLAlchemy + Alembic** (Python).

### Conditional modules

Activated based on the answers: `auth` (Keycloak or JWT httpOnly), `storage` (MinIO/S3),
`observability` (Prometheus + OpenTelemetry for Kubernetes), `logging` (Pino/structlog), `payments`
(Stripe), `email` (Resend) and `compose` (dev services in Docker Compose).

### Cross-cutting defaults (always)

TypeScript · pnpm · ESLint + Prettier · Docker Compose for dev · Vitest + Playwright (pytest in Python).

Every solution is generated with a **robust, feature-first folder structure** (organized by domain,
separation of UI/server/data layers and predictable locations for schemas, types, config and tests),
defined in each stack's reference files. Tests are not just scaffolding: the implementation writes
**real tests** per feature (see `devrothe/skills/create-application/references/testing.md`).

### Workflow

1. Interview (functional scope + stack decisions)
2. Stack resolution
3. Feature plan (recorded as tasks)
4. Confirmation with you (stack + plan; nothing is written/installed before the "yes")
5. Scaffold (project + dependencies + configs + `README.md`)
6. Feature-by-feature implementation, with real tests
7. Verification — lint, build, startup **and a green test suite** (completion gate)

> **Rules:** the generated project's `README.md` is kept in sync with the stack/dependencies/env/
> commands; and every implemented feature ships with real tests, with the green suite as a completion
> gate.

---

## Skill: `test-application`

Validates and runs an existing application's tests and, if there are none, analyzes the app, plans and
creates real tests per facet, runs them (with authorization) and presents a report.

### How to invoke

- Command: `/test-application`
- Trigger phrases: *"test the application"*, *"run the tests"*, *"does the app have tests?"*,
  *"validate the tests"*, *"create tests"*, *"test coverage"* (plus the Portuguese equivalents).

### Flow

1. **Detects** the existing tests and **maps coverage per facet** (UI, backend/API, microservice,
   service, CLI/library, fullstack, mobile, data/ML).
2. Picks the branch based on coverage:
   - **All covered** → runs the suite and presents the report.
   - **None covered** → reports the app type and the absence of tests; asks (Yes/No) whether you want
     to create them; if yes, plans per facet, creates (after approval), asks (Yes/No) and runs
     everything.
   - **Partial coverage** → reports which facets have and do not have tests; creates (with
     authorization) **only the missing ones**; then runs **all tests — existing + just created** — and
     reports.
3. **Final report** with totals, failures and coverage. Created tests are real (they exercise behavior)
   — no placeholders or empty snapshots.

---

## Skill: `refactor-application`

Takes an existing application — typically built by someone without experience — and restructures it to
the fundamentals, stack and organization of `create-application`, fixing along the way the bugs/errors
that were already there. Reuses the target methodology of `create-application` and the analysis/tests
of `test-application`.

### How to invoke

- Command: `/refactor-application`
- Trigger phrases: *"restructure the app"*, *"align with create-application"*, *"normalize the
  project"*, *"refactor the organization"*, *"fix up this application"* (plus the Portuguese
  equivalents).

### Flow

1. **Analysis** — app type and facets, deviation map against the target methodology, and bug/error
   detection.
2. **Phased restructuring plan** (low risk first), with a section dedicated to the bugs to fix.
3. **Confirmation** — presents the plan, effort and risks; nothing is changed before the "yes" (heavy
   migrations can be approved phase by phase).
4. **Safety net** — branch/commit (or backup) before touching anything.
5. **Execution** incrementally, preserving behavior and validating at each phase; README kept in sync.
6. **Validation** — runs the existing tests and creates the missing ones; green suite + lint + build +
   startup, and a final report (before → after, bugs resolved, test results).

> Restructures without changing functionality — the only allowed behavior change is fixing the bugs
> listed in the plan.

---

## Plugin development

- Marketplace manifest: `.claude-plugin/marketplace.json`
- Plugin manifest: `devrothe/.claude-plugin/plugin.json`
- The skills follow Anthropic's official best practices: third-person description with triggers,
  *progressive disclosure* (lean SKILL.md + references one level deep), imperative language and
  justification of the *why* behind the rules.
- **Language convention**: skill bodies, descriptions and docs are written in English, with **bilingual
  triggers (EN + PT)** so they still fire for Portuguese phrasing. See `CLAUDE.md`.

After editing any file, reload with `/plugin marketplace update devrothe`.
