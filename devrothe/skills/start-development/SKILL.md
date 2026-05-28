---
name: start-development
description: Arranca um projeto novo entrevistando o utilizador sobre requisitos, escolhendo o stack tecnológico adequado e fazendo o scaffold inicial. Usar quando o utilizador quer começar um projeto novo, decidir tecnologias/stack, ou inicializar uma base de código. Dispara em "start-development", "iniciar desenvolvimento", "novo projeto", "começar um projeto", "arranca um projeto", "que stack uso", "setup inicial", "scaffold". Decide entre stack Next.js (PoC, landing pages, e-commerce, sites institucionais) e stack React+Vite com backend Express/Node ou FastAPI/Python (web applications), com módulos condicionais para autenticação, armazenamento (MinIO), observabilidade (Kubernetes), logging, pagamentos (Stripe) e email (Resend).
---

# start-development

Entrevistar o utilizador, resolver o stack tecnológico e fazer o scaffold do projeto.

A escolha do stack não é livre: existe um stack-padrão por tipo de projeto (definido abaixo) para
garantir consistência entre projetos e evitar decisões ad-hoc. Propor outra tecnologia apenas se o
utilizador a pedir explicitamente ou se um requisito a tornar inviável — e nesse caso, explicar porquê.

**Regra: o `README.md` está sempre atualizado.** O scaffold gera um README e, daí em diante, qualquer
alteração ao stack, dependências, variáveis de ambiente ou comandos de arranque tem de ser refletida
no README na mesma tarefa. Porquê: o README é a fonte única de verdade para onboarding e para correr o
projeto — um README desatualizado é pior do que nenhum.

## Workflow

Copiar este checklist para a resposta e ir marcando:

```
- [ ] 1. Entrevista — recolher requisitos
- [ ] 2. Resolver o stack a partir das respostas
- [ ] 3. Apresentar o stack resolvido e confirmar com o utilizador
- [ ] 4. Scaffold — inicializar projeto, instalar dependências, gerar configs e o README
- [ ] 5. Verificar — o projeto instala, faz lint e arranca sem erros; o README reflete o estado real
```

### 1. Entrevista

Usar a ferramenta `AskUserQuestion` para fazer as perguntas. Começar pela pergunta de tipo de
projeto (decide o stack-base) e depois fazer as condicionais. Agrupar perguntas relacionadas no
mesmo batch para reduzir idas e voltas.

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
3. Tem autenticação de utilizadores? → **JWT em cookies httpOnly** (módulo `auth`).
4. Vai ser deployed em Kubernetes? → **Prometheus + OpenTelemetry** (módulo `observability`).
5. Quer logging estruturado/sofisticado? → **Pino** (Node) ou **structlog** (Python) (módulo `logging`).
6. Processa pagamentos? → **Stripe** (módulo `payments`).
7. Envia email transacional (verificação de conta, recuperação de password, encomendas)? →
   **Resend** (módulo `email`).

Se o backend for Python/FastAPI, perguntar também se prefere Express/Node — caso contrário, decidir
em função do projeto (ver `references/app-stack.md` para o critério).

### 2. Resolver o stack

Combinar o stack-base com os módulos ativados. Aplicar sempre os defaults transversais abaixo.

### 3. Confirmar

Apresentar ao utilizador a lista final resolvida (linguagem, framework, BD/ORM, UI, módulos) num
resumo curto e pedir confirmação **antes** de criar ou instalar o que quer que seja. O scaffold
escreve ficheiros e instala dependências — não avançar sem o "sim".

### 4. Scaffold

Seguir o ficheiro de referência do stack-base (`web-stack.md` ou `app-stack.md`) e, para cada módulo
ativado, a secção correspondente de `modules.md`. Executar os comandos de inicialização, instalar as
dependências e gerar as configs base.

Gerar um `README.md` na raiz do projeto com, no mínimo: descrição curta, stack escolhido, pré-requisitos,
variáveis de ambiente (`.env`), como subir os serviços de dev (Docker Compose), como instalar e arrancar,
e como correr testes e lint. Manter este README sincronizado em qualquer alteração futura (ver regra acima).

### 5. Verificar

Correr a instalação de dependências, o lint e o comando de build/dev. Confirmar que arranca sem
erros. Se falhar, corrigir antes de dar a tarefa por concluída.

## Defaults transversais (sempre, sem perguntar)

- **TypeScript** em todo o código JS.
- **pnpm** como package manager.
- **ESLint + Prettier** (com `prettier-plugin-tailwindcss`) para lint e formatação.
- **Tailwind CSS + shadcn/ui + lucide-react** para UI; **framer-motion** (animações), **next-themes**
  (tema claro/escuro) e **sonner** (toasts).
- **Zod + React Hook Form** (`@hookform/resolvers`) para validação de schemas e formulários.
- **TanStack Query** para server-state no cliente.
- **Docker Compose** para os serviços de dev (PostgreSQL e, se aplicável, MinIO).
- **Vitest + Playwright** para testes (Node/JS); **pytest** para Python.

## Referências

- **`references/web-stack.md`** — scaffold completo do stack Next.js (PoC, landing pages, e-commerce,
  sites institucionais).
- **`references/app-stack.md`** — scaffold do stack React + Vite com backend Express/Node ou
  FastAPI/Python, e o critério para escolher o backend.
- **`references/modules.md`** — módulos condicionais: `auth`, `storage` (MinIO), `observability`
  (Kubernetes), `logging`, `payments` (Stripe), `email` (Resend). Ler apenas as secções dos módulos
  ativados na entrevista.
