# Stack React + Vite + backend (web applications)

Stack-base para SPAs interativas (dashboards, SaaS, ferramentas internas) com um backend próprio
desacoplado.

## Contents
- Escolher o backend
- Frontend (React + Vite)
- Backend Node (Express)
- Backend Python (FastAPI)
- Base de dados
- Verificação

## Escolher o backend

Decidir entre **Express/Node** e **FastAPI/Python**:

- **Express/Node** — defeito quando a equipa/projeto é maioritariamente TypeScript, quando se quer
  partilhar tipos e validação (Zod) entre cliente e servidor, ou quando o domínio é I/O-bound (APIs,
  tempo real, integrações web).
- **FastAPI/Python** — quando o projeto envolve dados/ML, processamento numérico, bibliotecas
  científicas, ou quando a equipa já trabalha em Python. OpenAPI/Swagger automático e validação com
  Pydantic são vantagens.

Se o utilizador não tiver preferência, recomendar Express/Node para manter um só linguagem (TypeScript
ponta a ponta) salvo se houver um requisito de dados/ML claro.

## Frontend (React + Vite)

```bash
pnpm create vite@latest <nome> -- --template react-ts
cd <nome> && pnpm install
```

UI e dependências (mesmo stack visual do `web-stack.md`):

```bash
pnpm add lucide-react framer-motion next-themes sonner
pnpm add zod react-hook-form @hookform/resolvers
pnpm add @tanstack/react-query
pnpm add react-router-dom
pnpm add -D prettier prettier-plugin-tailwindcss
pnpm add -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

- Configurar Tailwind v4 e inicializar o shadcn/ui: `pnpm dlx shadcn@latest init`.
- `next-themes` funciona em apps React puras (sem Next): usar para alternância de tema com classe no
  `<html>`.
- Montar `<QueryClientProvider>` (TanStack Query) e `<Toaster />` (sonner) na raiz da app.
- Definir a base URL da API por variável de ambiente (`VITE_API_URL`).

Estrutura robusta e feature-first (criar as pastas no scaffold, `.gitkeep` nas vazias):

```
.
├── src/
│   ├── app/                  # bootstrap: providers (query, theme, toaster), router, App.tsx
│   ├── routes/               # definição de rotas (react-router) ou pages/
│   ├── components/
│   │   ├── ui/               # primitivos shadcn/ui (gerado)
│   │   └── shared/           # layout, navegação, componentes de alto nível
│   ├── features/             # um diretório por domínio/feature
│   │   └── <feature>/
│   │       ├── api/          # chamadas HTTP + hooks de query/mutation (TanStack Query)
│   │       ├── components/
│   │       ├── hooks/
│   │       ├── schemas.ts    # Zod
│   │       ├── types.ts
│   │       └── index.ts      # API pública da feature
│   ├── lib/                  # http client, queryClient, utils, env
│   ├── hooks/                # hooks globais
│   ├── config/               # constantes, config de runtime
│   ├── types/                # tipos globais
│   ├── assets/
│   └── styles/
└── tests/
    ├── unit/                 # Vitest + Testing Library (lógica e componentes)
    └── e2e/                  # Playwright (fluxos críticos)
```

- Importar apenas o `index.ts` de cada feature a partir de fora; não alcançar internals de outra
  feature. O que for partilhado sobe para `lib/`, `components/shared/` ou `hooks/`.
- Testes de componente podem ficar colocados (`*.test.tsx` junto ao componente) ou em `tests/unit/`;
  ver `testing.md` para a convenção e o que cobrir.

## Backend Node (Express)

```bash
pnpm add express cors helmet
pnpm add zod
pnpm add @prisma/client
pnpm add -D typescript tsx @types/express @types/cors prisma
pnpm add -D vitest supertest @types/supertest
```

- TypeScript com `tsx` para dev (`tsx watch src/server.ts`).
- Validar input nas fronteiras com Zod; partilhar schemas com o frontend quando possível.
- `helmet` + `cors` configurado para a origem do frontend.

Estrutura robusta, feature-first com camadas por módulo (criar as pastas no scaffold):

```
.
├── src/
│   ├── app.ts                    # cria a app Express (middleware, rotas) — sem listen
│   ├── server.ts                 # bootstrap: listen, graceful shutdown
│   ├── config/                   # env (validado com Zod), constantes
│   ├── modules/                  # um diretório por domínio
│   │   └── <module>/
│   │       ├── <module>.routes.ts
│   │       ├── <module>.controller.ts   # HTTP in/out
│   │       ├── <module>.service.ts      # regras de negócio
│   │       ├── <module>.repository.ts   # acesso a dados (Prisma)
│   │       ├── <module>.schema.ts       # Zod (validação de request)
│   │       └── <module>.types.ts
│   ├── middleware/               # error handler, auth, validação, rate-limit
│   ├── lib/                      # prisma client, logger, helpers
│   └── utils/
├── prisma/
│   ├── schema.prisma
│   └── migrations/
└── tests/
    ├── unit/                     # Vitest (services, utils — sem I/O)
    └── integration/              # supertest sobre app.ts (rotas + BD de teste)
```

- Fluxo de uma rota: `routes → controller → service → repository`. O controller não fala com o Prisma
  diretamente; a regra de negócio vive no service. Porquê: testabilidade e fronteiras claras.
- Ver `testing.md` para o que cobrir em cada camada e o gate de testes verde.

## Backend Python (FastAPI)

```bash
python -m venv .venv && source .venv/bin/activate
pip install fastapi "uvicorn[standard]" pydantic pydantic-settings
pip install sqlalchemy alembic psycopg[binary]
pip install -U pytest httpx
```

- Validação e settings com Pydantic; OpenAPI/Swagger automático em `/docs`.
- ORM: **SQLAlchemy + Alembic** (Prisma não é idiomático em Python). Inicializar migrations com
  `alembic init alembic`.
- Servir em dev: `uvicorn app.main:app --reload`.
- Gerir dependências num `requirements.txt` ou `pyproject.toml`.

Estrutura robusta, feature-first com camadas por módulo (criar as pastas no scaffold):

```
.
├── app/
│   ├── main.py                   # cria a app FastAPI, regista routers e middleware
│   ├── core/                     # config (Pydantic settings), security, logging
│   │   ├── config.py
│   │   └── security.py
│   ├── api/
│   │   ├── deps.py               # dependências partilhadas (auth, sessão DB)
│   │   └── v1/                   # versão da API; agrega os routers dos módulos
│   ├── modules/                  # um diretório por domínio
│   │   └── <module>/
│   │       ├── router.py         # endpoints (HTTP in/out)
│   │       ├── service.py        # regras de negócio
│   │       ├── repository.py     # acesso a dados (SQLAlchemy)
│   │       ├── models.py         # modelos SQLAlchemy
│   │       └── schemas.py        # schemas Pydantic (request/response)
│   └── db/
│       ├── base.py               # Declarative Base + import dos models
│       └── session.py            # engine + SessionLocal
├── alembic/                      # migrations
├── alembic.ini
└── tests/
    ├── unit/                     # pytest (services, lógica de domínio)
    └── integration/              # httpx (TestClient/AsyncClient) sobre a app + BD de teste
```

- Fluxo de um endpoint: `router → service → repository`. O router não toca na sessão SQLAlchemy
  diretamente; injeta dependências via `api/deps.py` e delega a regra de negócio ao service.
- Ver `testing.md` para o que cobrir em cada camada e o gate de testes verde.

## Base de dados

PostgreSQL (default). Subir via Docker Compose (ver `modules.md` ou criar um compose mínimo com
`postgres:16`). ORM conforme o backend:
- Node → **Prisma** (`pnpm dlx prisma init --datasource-provider postgresql`).
- Python → **SQLAlchemy + Alembic**.

## Verificação

- Frontend: `pnpm install && pnpm lint && pnpm build`.
- Backend Node: `pnpm install && pnpm test && pnpm dev`.
- Backend Python: `pytest && uvicorn app.main:app --reload`.
- Confirmar que o frontend comunica com o backend (CORS e base URL corretos).
