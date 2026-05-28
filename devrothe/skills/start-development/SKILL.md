---
name: start-development
description: Arranca um projeto novo de ponta a ponta — entrevista o utilizador sobre requisitos, escolhe o stack tecnológico adequado, faz o scaffold, planeia as features e implementa o projeto a partir do plano com testes reais. Usar quando o utilizador quer começar um projeto novo, decidir tecnologias/stack, ou inicializar e arrancar uma base de código. Dispara em "start-development", "iniciar desenvolvimento", "novo projeto", "começar um projeto", "arranca um projeto", "que stack uso", "setup inicial", "scaffold". Decide entre stack Next.js (PoC, landing pages, e-commerce, sites institucionais) e stack React+Vite com backend Express/Node ou FastAPI/Python (web applications), com módulos condicionais para autenticação, armazenamento (MinIO), observabilidade (Kubernetes), logging, pagamentos (Stripe) e email (Resend).
---

# start-development

Entrevistar o utilizador, resolver o stack, planear as features e implementar o projeto a partir do
plano — com testes reais.

A escolha do stack não é livre: existe um stack-padrão por tipo de projeto (definido abaixo) para
garantir consistência entre projetos e evitar decisões ad-hoc. Propor outra tecnologia apenas se o
utilizador a pedir explicitamente ou se um requisito a tornar inviável — e nesse caso, explicar porquê.

**Regra: o `README.md` está sempre atualizado.** O scaffold gera um README e, daí em diante, qualquer
alteração ao stack, dependências, variáveis de ambiente ou comandos de arranque tem de ser refletida
no README na mesma tarefa. Porquê: o README é a fonte única de verdade para onboarding e para correr o
projeto — um README desatualizado é pior do que nenhum.

**Regra: estrutura de pastas robusta e feature-first.** Cada solução gerada segue a estrutura definida
no ficheiro de referência do seu stack — organização por feature/domínio, separação clara de camadas
(UI, lógica de servidor, acesso a dados), e localizações previsíveis para schemas (Zod/Pydantic),
tipos, configuração e testes. Porquê: localizações previsíveis e fronteiras explícitas mantêm o
projeto navegável à medida que cresce, evitam ficheiros gigantes e travam o acoplamento acidental.
Não usar estruturas ad-hoc nem dispersar lógica de domínio por pastas técnicas genéricas quando existe
uma feature óbvia. As estruturas dos ficheiros de referência são o mínimo — criar as pastas no
scaffold mesmo que comecem vazias (com um `.gitkeep`), para fixar a convenção desde o início.

**Regra: testes reais acompanham cada feature.** Cada feature implementada traz testes que exercem o
seu comportamento (ver `references/testing.md`) — não basta a configuração e as pastas de teste
criadas no scaffold. Porquê: a camada de testes provisionada no scaffold é só andaime; sem testes que
afirmem comportamento não há rede de segurança e um gate de testes "verde" mas vazio é teatro. A suíte
de testes verde é um gate de conclusão (passo 7).

## Workflow

Copiar este checklist para a resposta e ir marcando:

```
- [ ] 0. Contexto — ler CLAUDE.md, memória e READMEs/docs do projeto (se existirem)
- [ ] 1. Entrevista — requisitos do stack + scope funcional (features, entidades, fluxos)
- [ ] 2. Resolver o stack a partir das respostas
- [ ] 3. Plano — desenhar as features/tarefas a construir
- [ ] 4. Confirmar — stack + plano com o utilizador (nada é escrito/instalado antes do "sim")
- [ ] 5. Scaffold — estrutura, dependências, configs e README
- [ ] 6. Implementação — feature a feature, guiada pelo plano, com testes reais
- [ ] 7. Verificar — lint, build, arranque E suíte de testes verde (gate de conclusão)
```

### 0. Contexto do projeto

Antes de avançar, procurar e ler os metadados que ajudem a perceber o contexto e as preferências:
- `CLAUDE.md` na raiz e em subpastas, e outros ficheiros de instruções de agentes (`AGENTS.md`,
  `.cursor/rules/`, `.github/copilot-instructions.md`).
- A **memória** do projeto (preferências e decisões já registadas).
- Ficheiros de texto: `README*`, `docs/`, `CONTRIBUTING.md`, `ARCHITECTURE.md`, ADRs e notas.

Usar este contexto para informar a entrevista, o plano e a implementação. Porquê: respeitar convenções
e decisões já documentadas evita contrariar o que o utilizador/equipa já definiu. Numa pasta vazia
normalmente não há nada a ler — seguir em frente; se já houver um `CLAUDE.md` com preferências,
respeitá-lo.

### 1. Entrevista

Usar a ferramenta `AskUserQuestion`. Recolher dois blocos:

**(a) Scope funcional** — o que a aplicação faz: features principais, entidades/dados centrais,
fluxos-chave de utilizador e páginas/endpoints obrigatórios. É isto que alimenta o plano (passo 3) e
a implementação (passo 6); sem scope funcional, a skill só consegue fazer scaffold.

**(b) Decisões de stack** — começar pela pergunta de tipo de projeto (decide o stack-base) e depois
as condicionais. Agrupar perguntas relacionadas no mesmo batch para reduzir idas e voltas.

**Pergunta-chave (decide o stack-base):**

| Resposta | Stack-base |
|----------|-----------|
| PoC, landing page, e-commerce, site institucional/apresentação, blog, site com pouca interatividade no cliente | **Next.js** — ver `references/web-stack.md` |
| Web application interativa: dashboard, SaaS, app com estado rico no cliente, ferramenta interna | **React + Vite + backend** — ver `references/app-stack.md` |

Se a fronteira for ambígua (ex.: e-commerce com área de cliente complexa), preferir Next.js — o App
Router cobre SSR/SEO e interatividade. Reservar React+Vite para casos onde o frontend é claramente
uma SPA desacoplada de um backend próprio.

**Perguntas condicionais** (cada "sim" ativa um módulo de `references/modules.md`):

1. Precisa de persistência de dados? → **PostgreSQL** (ativo por defeito; só desativar para sites
   100% estáticos). ORM: **Prisma** se o backend for Node/Next.js; **SQLAlchemy + Alembic** se for
   Python/FastAPI.
2. Vai armazenar imagens ou vídeos? → **MinIO** (módulo `storage`).
3. Tem autenticação de utilizadores? Se sim, perguntar a abordagem com `AskUserQuestion` (opção única):
   **Keycloak (OIDC)**, **JWT próprio em cookies httpOnly**, ou **deixar o AI escolher**. Ver o módulo
   `auth` em `references/modules.md` (inclui o critério de decisão quando é o AI a escolher).
4. Vai ser deployed em Kubernetes? → **Prometheus + OpenTelemetry** (módulo `observability`).
5. Quer logging estruturado/sofisticado? → **Pino** (Node) ou **structlog** (Python) (módulo `logging`).
6. Processa pagamentos? → **Stripe** (módulo `payments`).
7. Envia email transacional (verificação de conta, recuperação de password, encomendas)? →
   **Resend** (módulo `email`).

Se o backend for Python/FastAPI, perguntar também se prefere Express/Node — caso contrário, decidir
em função do projeto (ver `references/app-stack.md` para o critério).

### 2. Resolver o stack

Combinar o stack-base com os módulos ativados. Aplicar sempre os defaults transversais abaixo.

### 3. Plano

A partir do scope funcional, desenhar um plano de desenvolvimento: dividir em **fatias verticais
pequenas** e ordená-las por dependência (tipicamente: modelo de dados → autenticação → features de
domínio → UI/integrações). Para cada fatia, anotar o que entra e que testes a cobrem. Registar o plano
como tarefas com `TaskCreate` para acompanhar o progresso durante a implementação.

### 4. Confirmar

Apresentar ao utilizador o stack resolvido (linguagem, framework, BD/ORM, UI, módulos) **e** o plano
de features, num resumo curto, e pedir confirmação **antes** de criar ou instalar o que quer que seja.
O scaffold e a implementação escrevem ficheiros e instalam dependências — não avançar sem o "sim".

### 5. Scaffold

Seguir o ficheiro de referência do stack-base (`web-stack.md` ou `app-stack.md`) e, para cada módulo
ativado, a secção correspondente de `modules.md`. Executar os comandos de inicialização, instalar as
dependências (incluindo as de teste), gerar as configs base e criar a estrutura de pastas robusta.

Gerar um `README.md` na raiz do projeto com, no mínimo: descrição curta, stack escolhido,
pré-requisitos, variáveis de ambiente (`.env`), como subir os serviços de dev (Docker Compose), como
instalar e arrancar, e como correr testes e lint. Manter sincronizado em qualquer alteração futura
(ver regra acima).

### 6. Implementação

Implementar o projeto a partir do plano (passo 3), fatia a fatia, seguindo a estrutura de pastas do
stack. Para cada fatia:

1. Implementar o código da feature nas pastas certas, respeitando as camadas e fronteiras.
2. Escrever **testes reais** ao nível adequado (ver `references/testing.md`): unitários para
   lógica/serviços, integração para endpoints, componente para UI com lógica, e2e para fluxos
   críticos.
3. Correr os testes da fatia e iterar (escrever → correr → corrigir) até passarem.
4. Atualizar as tarefas (`TaskUpdate`) e o README conforme necessário antes de passar à fatia seguinte.

Não marcar uma fatia como concluída com os seus testes a falhar. Cobrir o caminho feliz e os
erros/edge cases relevantes de cada feature — não deixar features a meio.

### 7. Verificar

A tarefa só está concluída quando, no projeto gerado: a instalação de dependências passa, o lint
passa, o build passa, a app arranca, **e a suíte de testes completa está verde**. Correr os testes
como gate final; se algum falhar (ou estiver indevidamente em skip/xfail), corrigir antes de dar por
concluído. Confirmar também que o README reflete o estado real.

## Defaults transversais (sempre, sem perguntar)

- **TypeScript** em todo o código JS.
- **pnpm** como package manager.
- **ESLint + Prettier** (com `prettier-plugin-tailwindcss`) para lint e formatação.
- **Tailwind CSS + shadcn/ui + lucide-react** para UI; **framer-motion** (animações), **next-themes**
  (tema claro/escuro) e **sonner** (toasts).
- **Zod + React Hook Form** (`@hookform/resolvers`) para validação de schemas e formulários.
- **TanStack Query** para server-state no cliente.
- **Docker Compose** para os serviços de dev (PostgreSQL e, se aplicável, MinIO).
- **Testes**: Vitest + @testing-library/react + Playwright (Node/JS) e pytest + httpx (Python). A
  camada é instalada e configurada no scaffold e **preenchida com testes reais** na implementação —
  ver `references/testing.md`.

## Referências

- **`references/web-stack.md`** — scaffold completo do stack Next.js (PoC, landing pages, e-commerce,
  sites institucionais).
- **`references/app-stack.md`** — scaffold do stack React + Vite com backend Express/Node ou
  FastAPI/Python, e o critério para escolher o backend.
- **`references/testing.md`** — estratégia de testes: o que testar por camada, convenções de
  localização, ferramentas por stack e o gate de testes verde. Ler na implementação (passo 6).
- **`references/modules.md`** — módulos condicionais: `auth`, `storage` (MinIO), `observability`
  (Kubernetes), `logging`, `payments` (Stripe), `email` (Resend). Ler apenas as secções dos módulos
  ativados na entrevista.
