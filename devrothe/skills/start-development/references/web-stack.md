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

```
src/
├── app/                # rotas (App Router)
├── components/         # componentes partilhados
│   └── ui/             # gerado pelo shadcn/ui
├── lib/                # utilitários (ex.: lib/utils.ts, lib/prisma.ts)
├── hooks/              # hooks reutilizáveis
└── server/             # Server Actions e lógica de servidor
```

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
