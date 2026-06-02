# Devrothe

Plugin of **development-support skills** for [Claude Code](https://code.claude.com).
Distributed as a marketplace from GitHub (`dreazz97/devrothe-plugin`).

Current state: 5 skills — [`plan-strategy`](#skill-plan-strategy),
[`create-application`](#skill-create-application), [`test-application`](#skill-test-application),
[`refactor-application`](#skill-refactor-application) and
[`implement-feature`](#skill-implement-feature).

---

## Contents

```
Devrothe Plugin/
├── .claude-plugin/marketplace.json     # marketplace manifest
└── devrothe/                           # plugin
    ├── .claude-plugin/plugin.json
    └── skills/
        ├── plan-strategy/
        │   ├── SKILL.md                # context + frame + research + clarify + strategy + handoff
        │   └── references/
        │       ├── research-agent.md   # brief/run the web-research subagent (multi-source, cited)
        │       └── strategy-and-handoff.md # strategy layout + routing to the other skills
        ├── create-application/
        │   ├── SKILL.md                # interview + plan + scaffold + implementation
        │   └── references/
        │       ├── web-stack.md        # Next.js stack
        │       ├── app-stack.md        # React+Vite + backend stack
        │       ├── testing.md          # testing strategy
        │       ├── design.md           # professional UI/UX + anti-AI-slop rules
        │       ├── security.md         # PoC/demo gate + secure-by-default baseline
        │       └── modules.md          # conditional modules
        ├── test-application/
        │   ├── SKILL.md                # detect/run tests or analyze + create + report
        │   └── references/
        │       ├── app-analysis.md     # classify the application type
        │       ├── detect-and-run.md   # detect and run tests per ecosystem
        │       └── test-plan.md        # what to test per facet
        ├── refactor-application/
        │   ├── SKILL.md                # analyze + restructuring plan + execute + validate
        │   └── references/
        │       ├── assessment.md       # analysis: app type, deviations and bugs
        │       ├── restructure-plan.md # phased restructuring plan
        │       └── execution.md        # safe execution and validation
        └── implement-feature/
            ├── SKILL.md                # clarify + plan + impact statement + implement + tests
            └── references/
                ├── analysis-and-plan.md # analyze the codebase and structure the plan
                └── impact-analysis.md   # blast-radius assessment before coding
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
| `/plan-strategy` | Turns an idea into a researched, decision-ready strategy: grounds in the project, dispatches a web-research subagent, brainstorms with you, and on approval hands off to the right skill. |
| `/create-application` | Interviews, picks the stack, plans, scaffolds and implements the project with real tests. |
| `/test-application` | Detects and runs the app's tests and reports; if there are none, analyzes the app, creates tests per facet and reports. |
| `/refactor-application` | Analyzes an existing app, plans the restructuring toward the `create-application` stack/practices, executes (with approval) and validates with tests. |
| `/implement-feature` | Implements one or more requested features in an existing project: clarifies, plans the whole set ordered by dependency, states the impact, then builds them one at a time with real tests following the project's conventions. |

The skills also trigger via natural language (see the triggers below) — using the slash command is not
mandatory.

---

## Skill: `plan-strategy`

Turns a raw idea into a **researched, decision-ready strategy** — a more autonomous alternative to the
built-in plan mode. It acts as the planning agent: grounds itself in the project, delegates the web
research to a subagent, brainstorms with you, synthesizes a strategy you approve, and then hands that
strategy off to the right execution skill. It **plans, it does not build** — no files are changed in
this skill (beyond optionally recording the decided strategy when you ask).

### How to invoke

- Command: `/plan-strategy`
- Trigger phrases: *"help me plan"*, *"research and plan"*, *"brainstorm this idea"*, *"plan this"*
  (plus the Portuguese equivalents: *"planeia isto"*, *"ajuda-me a planear"*, *"pesquisa e planeia"*,
  *"brainstorm desta ideia"*).

### What it does

1. **Context** — reads `CLAUDE.md`, the project memory, the `README`/docs and analyzes the codebase
   (a full read for a broad idea), and notes the security posture and constraints.
2. **Frame** — restates the idea and the goal, then lists the open unknowns split into *"ask the user"*
   vs *"research the web"*.
3. **Research** — dispatches a dedicated **web-research subagent** (one brief per independent topic, in
   parallel) that consults **multiple independent sources**, prefers current/dated information, flags
   conflicts and returns a **cited briefing** with a confidence level.
4. **Clarify** — brainstorms with you: presents the options and trade-offs (now backed by evidence) and
   asks the questions needed to remove ambiguity.
5. **Strategy** — synthesizes context + research + answers into a concrete strategy (goal, constraints,
   options considered, **recommendation + rationale**, scope, risks, phases, success criteria),
   presented for approval and iterated until you're satisfied.
6. **Hand off** — on approval, triggers the matching skill, passing the strategy as input so it doesn't
   redo discovery:

   | The strategy is about… | Hands off to |
   |---|---|
   | A brand-new project | `create-application` |
   | Adding/building feature(s) in an existing app | `implement-feature` |
   | Restructuring/normalizing an existing app | `refactor-application` |
   | Running or creating tests | `test-application` |

> **Plans, doesn't build.** The planning phase stays free of side effects so you can iterate freely;
> the execution skills own their own detailed planning, impact analysis and tests — `plan-strategy`
> feeds them the strategy, it doesn't replace those steps. The **security posture** and UX weight are
> carried into the plan so the downstream skill applies the right defaults. If you'd rather stop at the
> plan, it stays recorded and nothing is triggered.

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
9. **How to run locally?** → choice (radio): services in Docker + app native (default), fully
   containerized (`Dockerfile` + `app` service, `docker compose up` runs everything), or both

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
(Stripe), `email` (Resend) and `compose` (dev services in Docker Compose, plus the chosen local run
mode — app native, fully containerized, or both).

### Cross-cutting defaults (always)

TypeScript · pnpm · ESLint + Prettier · Docker Compose for dev services (the app runs natively by
default; full containerization is opt-in) · Vitest + Playwright (pytest in Python).

Every solution is generated with a **robust, feature-first folder structure** (organized by domain,
separation of UI/server/data layers and predictable locations for schemas, types, config and tests),
defined in each stack's reference files. Tests are not just scaffolding: the implementation writes
**real tests** per feature (see `devrothe/skills/create-application/references/testing.md`).

UI is held to a **professional UI/UX bar** (hierarchy, spacing, type scale, WCAG AA contrast,
focus/keyboard states) and deliberately avoids the generic AI-generated look — no emojis as UI, no
cliché purple→blue gradients, no untouched stock defaults or buzzword filler (see
`devrothe/skills/create-application/references/design.md`).

For a real application it is also **secure-by-default**: input validation at the boundary,
authorization/ownership checks, parameterized data access, safe auth/secrets, security headers + HTTPS,
rate limiting on sensitive endpoints and OWASP Top 10 awareness (see
`devrothe/skills/create-application/references/security.md`). This is **gated on a PoC/demo decision** —
a throwaway prototype skips the hardening (only the repo-hygiene floor, never committing secrets,
always applies), and when the posture is ambiguous the AI asks you. The decision is recorded in the
project so the other skills honor it.

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

> **Manual testing:** when reporting test results, the AI always tells you which checks are still
> recommended to do by hand — especially UI/UX interaction in a real browser — since graphical aspects
> are hard to verify reliably with AI.

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
   — no placeholders or empty snapshots. The report also recommends **manual checks** (UI/UX in a real
   browser), since graphical aspects are hard to verify reliably with AI.

For a real app, the coverage map and the plan also include **security cases** — access control (IDOR /
unauthenticated access rejected), input validation, auth flows and webhook signatures — and treat the
critical paths (auth, payments, PII, tenancy) as must-cover. This is gated: skipped for a PoC/demo.

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

1. **Analysis** — app type and facets, deviation map against the target methodology, bug/error
   detection and, for a real app, **security findings** (red flags + deviations from the baseline, with
   severities).
2. **Phased restructuring plan** (low risk first), with a section dedicated to the bugs to fix and (for
   a real app) one for the security findings — fixes that change observable behavior are flagged as
   deliberate, like the bugs.
3. **Confirmation** — presents the plan, effort and risks; nothing is changed before the "yes" (heavy
   migrations can be approved phase by phase).
4. **Safety net** — branch/commit (or backup) before touching anything.
5. **Execution** incrementally, preserving behavior and validating at each phase; README kept in sync.
6. **Validation** — runs the existing tests and creates the missing ones; green suite + lint + build +
   startup, and a final report (before → after, bugs resolved, test results). The report also flags
   **manual checks** (UI/UX in a real browser), since graphical aspects are hard to verify with AI.

> Restructures without changing functionality — the only allowed behavior change is fixing the bugs
> listed in the plan.

---

## Skill: `implement-feature`

Implements one or more features you request into an existing project, respecting its conventions and
reusing the methodology of the other skills (`create-application` for structure/tests,
`test-application` for the test flow). Handles a single feature or a whole list — understands first,
plans the set, discloses impact, then builds with real tests.

### How to invoke

- Command: `/implement-feature`
- Trigger phrases: *"implement a feature"*, *"implement these features"*, *"add a feature"*, *"build
  this feature"* (plus the Portuguese equivalents).

### Flow

1. **Context** — reads CLAUDE.md, memory, READMEs/docs and the codebase area the feature(s) live in.
2. **Clarify** — lists out the requested feature(s) to confirm scope and asks you questions about any
   that are ambiguous (skipped if already clear).
3. **Plan** — runs an extensive codebase analysis when the conversation lacks context, then one
   consolidated plan of small slices; when several features are requested they are ordered by
   dependency and grouped per feature. Presented for approval.
4. **Impact** — after approval and before coding, states the blast radius per feature: whether it can
   break the app, which existing features it touches, or that it is isolated — plus the cross-feature
   interactions and, for a real app, the **security impact** (new attack surface and its severity, or
   none).
5. **Implement** — one feature at a time in dependency order, each slice by slice, following the
   project's stack/structure/conventions (professional UI/UX, no AI-slop tells; secure-by-default for a
   real app), with a safety net for changes to existing code and a checkpoint between features.
6. **Tests & validate** — real tests per feature (including security-relevant ones for a real app), a
   green suite + lint/build/startup after each feature and at the end, and the manual-testing
   disclaimer (browser UI/UX).

> Preserves existing behavior — a new feature must not break or change other features (including ones
> delivered earlier in the same batch). Security hardening is gated on the project's PoC/demo posture
> (skipped for a throwaway, applied for a real app).

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
