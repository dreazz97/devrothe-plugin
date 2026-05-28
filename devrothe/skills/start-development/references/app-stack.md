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

Estrutura sugerida:

```
src/
├── components/   (ui/ gerado pelo shadcn)
├── pages/ ou routes/
├── lib/          (cliente HTTP, query client, utils)
├── hooks/
└── features/     (organização por domínio)
```

## Backend Node (Express)

```bash
pnpm add express cors helmet
pnpm add zod
pnpm add @prisma/client
pnpm add -D typescript tsx @types/express @types/cors prisma
pnpm add -D vitest supertest @types/supertest
```

- TypeScript com `tsx` para dev (`tsx watch src/index.ts`).
- Validar input nas fronteiras com Zod; partilhar schemas com o frontend quando possível.
- `helmet` + `cors` configurado para a origem do frontend.
- Estrutura por camadas: `routes/ → controllers/ → services/ → repositories (Prisma)`.

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
- Estrutura: `app/ (main.py, api/, models/, schemas/, services/, db/)`.
- Gerir dependências num `requirements.txt` ou `pyproject.toml`.

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
