# Análise do projeto

Objetivo: perceber o estado atual sem o alterar e produzir três saídas — tipo de app, mapa de desvios
à metodologia-alvo e lista de bugs/erros. **Não editar nada nesta fase.**

## Contents
- Estado atual e tipo de app
- Mapa de desvios à metodologia-alvo
- Deteção de bugs/erros
- Saída da análise

## Estado atual e tipo de app

- Detetar linguagem(ns), gestor de dependências, frameworks e scripts (manifestos, lockfiles).
- Classificar o tipo de app e as vertentes com `../test-application/references/app-analysis.md`
  (UI/frontend, backend/API, microserviço, serviço, CLI/biblioteca, fullstack, etc.).
- Tentar arrancar/instalar para conhecer o baseline (compila? arranca? testes existentes passam?).

## Mapa de desvios à metodologia-alvo

Comparar o projeto com o destino da `start-development`. Para cada item, marcar **OK / desvio**:

| Dimensão | Alvo |
|----------|------|
| Linguagem | TypeScript em todo o JS |
| Package manager | pnpm |
| Lint/format | ESLint + Prettier (+ prettier-plugin-tailwindcss) |
| Framework base | Next.js (sites/PoC) ou React+Vite + backend (web apps) |
| UI | Tailwind + shadcn/ui + lucide-react, framer-motion, next-themes, sonner |
| Forms/validação | Zod + React Hook Form |
| Server-state | TanStack Query |
| ORM / BD | Prisma (Node) ou SQLAlchemy+Alembic (Python) sobre PostgreSQL |
| Estrutura de pastas | feature-first robusta (ver web-stack.md / app-stack.md) |
| Testes | camada real (ver testing.md) |
| Infra de dev | Docker Compose (Postgres, e MinIO/Keycloak se aplicável) |
| Módulos | auth (Keycloak/JWT httpOnly), storage (MinIO), observability, logging, payments, email |

Detalhes do alvo em `../start-development/references/web-stack.md`, `app-stack.md`, `modules.md` e
`testing.md`.

## Deteção de bugs/erros

Procurar e registar (com severidade):
- **Bloqueantes**: build falha, erros de tipos (`tsc`), a app não arranca, dependências em falta/quebradas.
- **Testes**: testes existentes a falhar.
- **Qualidade/anti-padrões**: erros de lint, código morto, lógica duplicada, componentes/ficheiros
  gigantes, ausência de tratamento de erros nas fronteiras.
- **Segurança (red flags)**: segredos hardcoded, tokens em `localStorage`, SQL por concatenação de
  strings, CORS aberto, validação de input em falta. (Sinalizar; não é uma auditoria completa.)

## Saída da análise

Resumir ao utilizador: tipo de app + vertentes, a tabela de desvios (OK/desvio) e a lista de bugs/erros
por severidade. Esta saída alimenta o plano (restructure-plan.md).
