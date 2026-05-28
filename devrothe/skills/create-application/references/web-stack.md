# Next.js stack (PoC, landing pages, e-commerce, marketing sites)

Base stack for projects with a strong SSR/SEO component and moderate client-side interactivity.

## Contents
- Initialization
- Dependencies
- Configuration
- Folder structure
- Database (Prisma + PostgreSQL)
- Verification

## Initialization

Create the app (App Router, TypeScript, Tailwind, ESLint, `src/`, alias `@/*`):

```bash
pnpm create next-app@latest <name> --ts --tailwind --eslint --app --src-dir --import-alias "@/*"
```

Initialize shadcn/ui inside the project folder:

```bash
pnpm dlx shadcn@latest init
```

Add components as needed, e.g.: `pnpm dlx shadcn@latest add button input form sonner`.

## Dependencies

```bash
# UI and UX
pnpm add lucide-react framer-motion next-themes sonner
# Forms and validation
pnpm add zod react-hook-form @hookform/resolvers
# Client-side server-state
pnpm add @tanstack/react-query
# Lint/format
pnpm add -D prettier prettier-plugin-tailwindcss
# Tests
pnpm add -D vitest @testing-library/react @testing-library/jest-dom jsdom @vitejs/plugin-react
pnpm create playwright@latest
```

Note: `framer-motion` is the requested package; its successor is also published as `motion`. Use
`framer-motion` unless told otherwise.

## Configuration

- **next-themes**: wrap `app/layout.tsx` in a `ThemeProvider` (`attribute="class"`,
  `defaultTheme="system"`). Ensure `suppressHydrationWarning` on `<html>`.
- **sonner**: mount `<Toaster />` in the layout.
- **TanStack Query**: create a `QueryProvider` (client component) with `QueryClientProvider` and wrap
  the branches that fetch on the client. In App Router, prefer Server Components / Server Actions for
  server data — reserve TanStack Query for client-side data (mutations, data that changes on
  interaction). Do not duplicate fetching.
- **Prettier**: create `.prettierrc` with `"plugins": ["prettier-plugin-tailwindcss"]`.
- **Vitest**: create `vitest.config.ts` with `environment: "jsdom"` and the React plugin.

## Folder structure

Robust, feature-first structure. Create these folders during scaffold (use `.gitkeep` in the ones that
start empty). Routes live in `app/`; each domain's code lives in `features/<feature>/` and is imported
by the routes — keep the pages thin.

```
.
├── src/
│   ├── app/                      # App Router: layouts, pages, loading/error, route handlers
│   │   ├── (marketing)/          # route groups by area (public vs authenticated)
│   │   ├── (app)/
│   │   └── api/                  # route handlers (REST/webhooks)
│   ├── components/
│   │   ├── ui/                   # shadcn/ui primitives (generated)
│   │   └── shared/               # high-level shared components (layout, nav)
│   ├── features/                 # one directory per domain/feature
│   │   └── <feature>/
│   │       ├── components/       # feature-specific UI
│   │       ├── hooks/
│   │       ├── actions.ts        # Server Actions
│   │       ├── queries.ts        # data reads (server)
│   │       ├── schemas.ts        # Zod
│   │       └── types.ts
│   ├── server/                   # shared server logic (services, auth, db helpers)
│   ├── lib/                      # clients and utilities (prisma.ts, utils.ts, env.ts)
│   ├── hooks/                    # global hooks
│   ├── config/                   # site config, navigation, constants
│   ├── types/                    # global types
│   └── styles/                   # globals.css
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── tests/
│   ├── unit/                     # Vitest
│   └── e2e/                      # Playwright
└── public/
```

- Validate `env.ts` with Zod at startup (fail fast if a variable is missing).
- Each feature exposes its internal API through the folders above; avoid importing one feature's
  internals from another — go through what lives in `server/` or `lib/` when it is shared.
- Tests: `tests/unit/` (Vitest + Testing Library) and `tests/e2e/` (Playwright). See `testing.md` for
  what to cover at each layer and the green test gate.

## Database (Prisma + PostgreSQL)

Enable when the project needs persistence (default, except 100% static sites).

```bash
pnpm add @prisma/client
pnpm add -D prisma
pnpm dlx prisma init --datasource-provider postgresql
```

- Set `DATABASE_URL` in `.env` pointing at the Docker Compose Postgres (see `modules.md` for the
  service, or create a minimal `docker-compose.yml` with `postgres:16`).
- Create `src/lib/prisma.ts` with a `PrismaClient` singleton (avoids exhausting connections in dev
  with hot-reload).
- Model the schema in `prisma/schema.prisma` and run `pnpm dlx prisma migrate dev --name init`.

## Verification

```bash
pnpm install
pnpm lint
pnpm build   # or: pnpm dev  for local startup
```
