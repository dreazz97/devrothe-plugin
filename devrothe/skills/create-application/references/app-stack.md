# React + Vite + backend stack (web applications)

Base stack for interactive SPAs (dashboards, SaaS, internal tools) with their own decoupled backend.

## Contents
- Choosing the backend
- Frontend (React + Vite)
- Node backend (Express)
- Python backend (FastAPI)
- Database
- Verification

## Choosing the backend

Decide between **Express/Node** and **FastAPI/Python**:

- **Express/Node** — default when the team/project is mostly TypeScript, when you want to share types
  and validation (Zod) between client and server, or when the domain is I/O-bound (APIs, real-time,
  web integrations).
- **FastAPI/Python** — when the project involves data/ML, numerical processing, scientific libraries,
  or when the team already works in Python. Automatic OpenAPI/Swagger and Pydantic validation are
  advantages.

If the user has no preference, recommend Express/Node to keep a single language (TypeScript end to
end) unless there is a clear data/ML requirement.

## Frontend (React + Vite)

```bash
pnpm create vite@latest <name> -- --template react-ts
cd <name> && pnpm install
```

UI and dependencies (same visual stack as `web-stack.md`):

```bash
pnpm add lucide-react framer-motion next-themes sonner
pnpm add zod react-hook-form @hookform/resolvers
pnpm add @tanstack/react-query
pnpm add react-router-dom
pnpm add -D prettier prettier-plugin-tailwindcss
pnpm add -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

- Configure Tailwind v4 and initialize shadcn/ui: `pnpm dlx shadcn@latest init`.
- `next-themes` works in plain React apps (no Next): use it for theme switching with a class on
  `<html>`.
- Mount `<QueryClientProvider>` (TanStack Query) and `<Toaster />` (sonner) at the app root.
- Set the API base URL via an environment variable (`VITE_API_URL`).

Robust, feature-first structure (create the folders during scaffold, `.gitkeep` in the empty ones):

```
.
├── src/
│   ├── app/                  # bootstrap: providers (query, theme, toaster), router, App.tsx
│   ├── routes/               # route definitions (react-router) or pages/
│   ├── components/
│   │   ├── ui/               # shadcn/ui primitives (generated)
│   │   └── shared/           # layout, navigation, high-level components
│   ├── features/             # one directory per domain/feature
│   │   └── <feature>/
│   │       ├── api/          # HTTP calls + query/mutation hooks (TanStack Query)
│   │       ├── components/
│   │       ├── hooks/
│   │       ├── schemas.ts    # Zod
│   │       ├── types.ts
│   │       └── index.ts      # the feature's public API
│   ├── lib/                  # http client, queryClient, utils, env
│   ├── hooks/                # global hooks
│   ├── config/               # constants, runtime config
│   ├── types/                # global types
│   ├── assets/
│   └── styles/
└── tests/
    ├── unit/                 # Vitest + Testing Library (logic and components)
    └── e2e/                  # Playwright (critical flows)
```

- From the outside, import only each feature's `index.ts`; do not reach into another feature's
  internals. Anything shared moves up to `lib/`, `components/shared/` or `hooks/`.
- Component tests can be colocated (`*.test.tsx` next to the component) or in `tests/unit/`; see
  `testing.md` for the convention and what to cover.

## Node backend (Express)

```bash
pnpm add express cors helmet
pnpm add zod
pnpm add @prisma/client
pnpm add -D typescript tsx @types/express @types/cors prisma
pnpm add -D vitest supertest @types/supertest
```

- TypeScript with `tsx` for dev (`tsx watch src/server.ts`).
- Validate input at the boundaries with Zod; share schemas with the frontend when possible.
- `helmet` + `cors` configured for the frontend's origin.

Robust, feature-first structure with per-module layers (create the folders during scaffold):

```
.
├── src/
│   ├── app.ts                    # creates the Express app (middleware, routes) — no listen
│   ├── server.ts                 # bootstrap: listen, graceful shutdown
│   ├── config/                   # env (validated with Zod), constants
│   ├── modules/                  # one directory per domain
│   │   └── <module>/
│   │       ├── <module>.routes.ts
│   │       ├── <module>.controller.ts   # HTTP in/out
│   │       ├── <module>.service.ts      # business rules
│   │       ├── <module>.repository.ts   # data access (Prisma)
│   │       ├── <module>.schema.ts       # Zod (request validation)
│   │       └── <module>.types.ts
│   ├── middleware/               # error handler, auth, validation, rate-limit
│   ├── lib/                      # prisma client, logger, helpers
│   └── utils/
├── prisma/
│   ├── schema.prisma
│   └── migrations/
└── tests/
    ├── unit/                     # Vitest (services, utils — no I/O)
    └── integration/              # supertest against app.ts (routes + test DB)
```

- Route flow: `routes → controller → service → repository`. The controller does not talk to Prisma
  directly; business rules live in the service. Why: testability and clear boundaries.
- See `testing.md` for what to cover at each layer and the green test gate.

## Python backend (FastAPI)

```bash
python -m venv .venv && source .venv/bin/activate
pip install fastapi "uvicorn[standard]" pydantic pydantic-settings
pip install sqlalchemy alembic psycopg[binary]
pip install -U pytest httpx
```

- Validation and settings with Pydantic; automatic OpenAPI/Swagger at `/docs`.
- ORM: **SQLAlchemy + Alembic** (Prisma is not idiomatic in Python). Initialize migrations with
  `alembic init alembic`.
- Serve in dev: `uvicorn app.main:app --reload`.
- Manage dependencies in a `requirements.txt` or `pyproject.toml`.

Robust, feature-first structure with per-module layers (create the folders during scaffold):

```
.
├── app/
│   ├── main.py                   # creates the FastAPI app, registers routers and middleware
│   ├── core/                     # config (Pydantic settings), security, logging
│   │   ├── config.py
│   │   └── security.py
│   ├── api/
│   │   ├── deps.py               # shared dependencies (auth, DB session)
│   │   └── v1/                   # API version; aggregates the modules' routers
│   ├── modules/                  # one directory per domain
│   │   └── <module>/
│   │       ├── router.py         # endpoints (HTTP in/out)
│   │       ├── service.py        # business rules
│   │       ├── repository.py     # data access (SQLAlchemy)
│   │       ├── models.py         # SQLAlchemy models
│   │       └── schemas.py        # Pydantic schemas (request/response)
│   └── db/
│       ├── base.py               # Declarative Base + model imports
│       └── session.py            # engine + SessionLocal
├── alembic/                      # migrations
├── alembic.ini
└── tests/
    ├── unit/                     # pytest (services, domain logic)
    └── integration/              # httpx (TestClient/AsyncClient) against the app + test DB
```

- Endpoint flow: `router → service → repository`. The router does not touch the SQLAlchemy session
  directly; it injects dependencies via `api/deps.py` and delegates business rules to the service.
- See `testing.md` for what to cover at each layer and the green test gate.

## Database

PostgreSQL (default). Bring it up via Docker Compose (see `modules.md` or create a minimal compose
with `postgres:16`). ORM depending on the backend:
- Node → **Prisma** (`pnpm dlx prisma init --datasource-provider postgresql`).
- Python → **SQLAlchemy + Alembic**.

## Verification

- Frontend: `pnpm install && pnpm lint && pnpm build`.
- Node backend: `pnpm install && pnpm test && pnpm dev`.
- Python backend: `pytest && uvicorn app.main:app --reload`.
- Confirm the frontend talks to the backend (correct CORS and base URL).
