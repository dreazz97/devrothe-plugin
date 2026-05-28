# Stack Next.js (PoC, landing pages, e-commerce, sites institucionais)

Stack-base para projetos com forte componente de SSR/SEO e interatividade moderada no cliente.

## Contents
- Inicialização
- Dependências
- Configuração
- Estrutura de pastas
- Base de dados (Prisma + PostgreSQL)
- Verificação

## Inicialização

Criar a app (App Router, TypeScript, Tailwind, ESLint, `src/`, alias `@/*`):

```bash
pnpm create next-app@latest <nome> --ts --tailwind --eslint --app --src-dir --import-alias "@/*"
```

Inicializar o shadcn/ui dentro da pasta do projeto:

```bash
pnpm dlx shadcn@latest init
```

Adicionar componentes à medida que forem precisos, p.ex.: `pnpm dlx shadcn@latest add button input form sonner`.

## Dependências

```bash
# UI e UX
pnpm add lucide-react framer-motion next-themes sonner
# Formulários e validação
pnpm add zod react-hook-form @hookform/resolvers
# Server-state no cliente
pnpm add @tanstack/react-query
# Lint/format
pnpm add -D prettier prettier-plugin-tailwindcss
# Testes
pnpm add -D vitest @testing-library/react @testing-library/jest-dom jsdom @vitejs/plugin-react
pnpm create playwright@latest
```

Nota: `framer-motion` é o pacote pedido; o seu sucessor publica-se também como `motion`. Usar
`framer-motion` salvo indicação em contrário.

## Configuração

- **next-themes**: envolver o `app/layout.tsx` num `ThemeProvider` (`attribute="class"`,
  `defaultTheme="system"`). Garantir `suppressHydrationWarning` no `<html>`.
- **sonner**: montar `<Toaster />` no layout.
- **TanStack Query**: criar um `QueryProvider` (client component) com `QueryClientProvider` e envolver
  os ramos que fazem fetch no cliente. Em App Router, preferir Server Components / Server Actions para
  dados de servidor — reservar o TanStack Query para dados client-side (mutations, dados que mudam por
  interação). Não duplicar fetching.
- **Prettier**: criar `.prettierrc` com `"plugins": ["prettier-plugin-tailwindcss"]`.
- **Vitest**: criar `vitest.config.ts` com `environment: "jsdom"` e o plugin de React.

## Estrutura de pastas

Estrutura robusta e feature-first. Criar estas pastas no scaffold (usar `.gitkeep` nas que comecem
vazias). As rotas vivem em `app/`; o código de cada domínio vive em `features/<feature>/` e é
importado pelas rotas — manter as páginas finas.

```
.
├── src/
│   ├── app/                      # App Router: layouts, pages, loading/error, route handlers
│   │   ├── (marketing)/          # route groups por área (público vs autenticado)
│   │   ├── (app)/
│   │   └── api/                  # route handlers (REST/webhooks)
│   ├── components/
│   │   ├── ui/                   # primitivos shadcn/ui (gerado)
│   │   └── shared/               # componentes partilhados de alto nível (layout, nav)
│   ├── features/                 # um diretório por domínio/feature
│   │   └── <feature>/
│   │       ├── components/       # UI específica da feature
│   │       ├── hooks/
│   │       ├── actions.ts        # Server Actions
│   │       ├── queries.ts        # leituras de dados (server)
│   │       ├── schemas.ts        # Zod
│   │       └── types.ts
│   ├── server/                   # lógica de servidor partilhada (services, auth, db helpers)
│   ├── lib/                      # clientes e utilitários (prisma.ts, utils.ts, env.ts)
│   ├── hooks/                    # hooks globais
│   ├── config/                   # site config, navegação, constantes
│   ├── types/                    # tipos globais
│   └── styles/                   # globals.css
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── tests/
│   ├── unit/                     # Vitest
│   └── e2e/                      # Playwright
└── public/
```

- Validar `env.ts` com Zod no arranque (falhar cedo se faltar uma variável).
- Cada feature expõe a sua API interna pelas pastas acima; evitar importar internals de uma feature
  noutra — passar pelo que está em `server/` ou `lib/` quando for partilhado.
- Testes: `tests/unit/` (Vitest + Testing Library) e `tests/e2e/` (Playwright). Ver `testing.md` para
  o que cobrir em cada camada e o gate de testes verde.

## Base de dados (Prisma + PostgreSQL)

Ativar quando o projeto precisa de persistência (default, exceto sites 100% estáticos).

```bash
pnpm add @prisma/client
pnpm add -D prisma
pnpm dlx prisma init --datasource-provider postgresql
```

- Definir `DATABASE_URL` no `.env` a apontar para o Postgres do Docker Compose (ver `modules.md` para
  o serviço, ou criar um `docker-compose.yml` mínimo com `postgres:16`).
- Criar `src/lib/prisma.ts` com um singleton do `PrismaClient` (evita esgotar conexões em dev com
  hot-reload).
- Modelar o schema em `prisma/schema.prisma` e correr `pnpm dlx prisma migrate dev --name init`.

## Verificação

```bash
pnpm install
pnpm lint
pnpm build   # ou: pnpm dev  para arranque local
```
