# Application type analysis

Classify the application from what is in the project. An app can have **multiple facets** (e.g.,
fullstack = frontend + backend). Record all that apply — the test plan (test-plan.md) depends on this.

## Contents
- Identify the ecosystem
- Signals per facet
- How to report the classification

## Identify the ecosystem

Look for manifests to find the language(s) and dependency manager:

| File | Ecosystem |
|------|-----------|
| `package.json` | Node/JS/TS |
| `pyproject.toml`, `requirements.txt`, `setup.py` | Python |
| `go.mod` | Go |
| `pom.xml`, `build.gradle(.kts)` | Java/Kotlin |
| `Cargo.toml` | Rust |
| `composer.json` | PHP |
| `*.csproj`, `*.sln` | .NET |
| `Gemfile` | Ruby |

In a monorepo, several may coexist — analyze each package/service.

## Signals per facet

- **Frontend / UI** — dependencies `react`/`vue`/`@angular/core`/`svelte`/`next`/`vite`/`astro`;
  `.tsx`/`.jsx`/`.vue`/`.svelte` files, `index.html`, `public/`, components, CSS/Tailwind, design
  system. Indicates component tests and UI e2e.
- **Backend / API** — web frameworks (`express`/`fastify`/`@nestjs`/`koa`, `fastapi`/`flask`/`django`,
  `spring-boot`, `gin`/`echo`/`fiber`, `actix`/`axum`); handlers/controllers, HTTP server bootstrap,
  routes, OpenAPI/Swagger spec. Indicates endpoint integration tests + service unit tests.
- **Microservice** — single, small responsibility, independently deployable; signals: `Dockerfile` +
  Kubernetes/Helm manifests, message broker clients (`kafka`/`rabbitmq`/`nats`), `.proto`/gRPC files,
  health/readiness endpoints, contracts with other services. Indicates contract tests, message-handler
  tests and integration (testcontainers).
- **Service / monolith** — a larger backend with multiple domains in the same deployable. Same tests as
  the backend, with more emphasis on boundaries between modules.
- **CLI / library** — `bin`/`console_scripts` entry, no server, publishable package, exported API.
  Indicates unit tests of the public API and CLI invocation tests.
- **Fullstack** — frontend + backend in the same repository. Treat as two facets.
- **Mobile** — `react-native`/`expo`, Flutter (`pubspec.yaml`). Component/widget tests + e2e
  (Detox/Maestro).
- **Data / ML** — notebooks, `pandas`/`numpy`/`scikit-learn`/`torch`, training/ETL scripts. Tests for
  data transformations and pure functions; validate data shapes/contracts.

Auxiliary signals: `docker-compose.yml` (several services → likely microservices/fullstack),
`infra/`/`k8s/`/`helm/`, CI pipelines, `Makefile`.

## How to report the classification

Summarize to the user in a few lines: language(s), application type and the list of detected facets,
followed by the observation that no tests exist. Example:
"Fullstack app (React + Vite frontend and FastAPI/REST backend). No tests in the project."
