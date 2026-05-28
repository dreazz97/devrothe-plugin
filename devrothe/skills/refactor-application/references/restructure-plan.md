# Plano de reestruturação

Transformar a saída da análise (assessment.md) num plano faseado, de baixo risco para alto, preservando
o comportamento da aplicação.

## Contents
- Princípios
- Ordem das fases
- Estrutura de cada fase
- Secção obrigatória de bugs/erros

## Princípios

- **Preservar comportamento**: refactors são equivalência funcional; a única mudança de comportamento
  permitida é a correção dos bugs listados.
- **Incremental**: fatias pequenas e validáveis; nada de "big bang".
- **Baixo risco primeiro**: começar pelo que é seguro e desbloqueia o resto.
- **Migrações pesadas são opt-in**: uma troca de framework (ex.: CRA → Vite, JS → TS em massa) é
  assinalada à parte e só avança com aprovação explícita; preferir migração faseada (strangler) a
  reescrever tudo de uma vez.

## Ordem das fases

Sugestão (ajustar ao projeto e às vertentes detetadas):

1. **Baseline + rede de segurança** — garantir que se sabe arrancar a app; criar branch/commit ou
   backup (ver execution.md).
2. **Bugs bloqueantes** — corrigir o que impede build/arranque, para refatorar sobre código que corre.
3. **Tooling base** — TypeScript, pnpm, ESLint + Prettier, scripts de `dev/build/lint/test`.
4. **Alinhamento de stack/dependências** — UI (Tailwind + shadcn/ui + lucide-react, framer-motion,
   next-themes, sonner), Zod + React Hook Form, TanStack Query, ORM (Prisma/SQLAlchemy) sobre Postgres.
5. **Estrutura de pastas** — migrar para feature-first robusta (ver `../create-application/references/`
   `web-stack.md` ou `app-stack.md`), movendo o código por domínio.
6. **Módulos** — alinhar auth (Keycloak/JWT httpOnly), storage (MinIO), observability, logging,
   payments, email conforme aplicável (ver `../create-application/references/modules.md`).
7. **Testes** — alinhar/adicionar a camada de testes (ver `../create-application/references/testing.md`).
8. **Restantes bugs/qualidade** — corrigir bugs não-bloqueantes e remover dead code/anti-padrões.
9. **Docs** — atualizar o `README.md` para refletir o estado final.

## Estrutura de cada fase

Para cada fase, registar (e criar tarefa com `TaskCreate`):
- **Objetivo** e ficheiros/áreas afetadas.
- **Mudanças** concretas.
- **Validação** (como confirmar que continua a funcionar: build/lint/testes/arranque).
- **Risco / reversão** (o que fazer se correr mal).

## Secção obrigatória de bugs/erros

O plano inclui sempre uma secção com **todos os bugs/erros detetados na análise**, cada um com:
severidade, sintoma, causa provável, fase em que será corrigido e como será validado. Bloqueantes vão
para a fase 2; os restantes para a fase 8 (ou para a fase do módulo a que pertencem).
