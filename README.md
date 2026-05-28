# Devrothe

Plugin de **skills de apoio ao desenvolvimento** para o [Claude Code](https://code.claude.com).
Distribuído como marketplace a partir do GitHub (`dreazz97/devrothe-plugin`).

Estado atual: 3 skills — [`start-development`](#skill-start-development),
[`test-application`](#skill-test-application) e [`fix-application`](#skill-fix-application).

---

## Conteúdo

```
Devrothe Plugin/
├── .claude-plugin/marketplace.json     # manifesto do marketplace
└── devrothe/                           # plugin
    ├── .claude-plugin/plugin.json
    └── skills/
        ├── start-development/
        │   ├── SKILL.md                # entrevista + plano + scaffold + implementação
        │   └── references/
        │       ├── web-stack.md        # stack Next.js
        │       ├── app-stack.md        # stack React+Vite + backend
        │       ├── testing.md          # estratégia de testes
        │       └── modules.md          # módulos condicionais
        ├── test-application/
        │   ├── SKILL.md                # detetar/correr testes ou analisar + criar + reportar
        │   └── references/
        │       ├── app-analysis.md     # classificar o tipo de aplicação
        │       ├── detect-and-run.md   # detetar e executar testes por ecossistema
        │       └── test-plan.md        # o que testar por vertente
        └── fix-application/
            ├── SKILL.md                # analisar + plano de reestruturação + executar + validar
            └── references/
                ├── assessment.md       # análise: tipo de app, desvios e bugs
                ├── restructure-plan.md # plano faseado de reestruturação
                └── execution.md        # execução segura e validação
```

---

## Instalação

A partir do Claude Code:

```
/plugin marketplace add dreazz97/devrothe-plugin
/plugin install devrothe@devrothe
```

O formato de instalação é `plugin@marketplace` — aqui ambos se chamam `devrothe`.

### Atualizar após alterações

Depois de novos commits no repositório:

```
/plugin marketplace update devrothe
```

### Desenvolvimento local (opcional)

Para iterar sobre o plugin sem passar pelo GitHub, adiciona o marketplace por caminho local:

```
/plugin marketplace add "/Volumes/Samsung PSSD T7 Shield Media/Novlok/Devrothe Plugin"
```

---

## Comandos / Skills

| Comando | O que faz |
|---------|-----------|
| `/start-development` | Entrevista, escolhe o stack, planeia, faz o scaffold e implementa o projeto com testes reais. |
| `/test-application` | Deteta e corre os testes da app e reporta; se não houver, analisa a app, cria testes por vertente e reporta. |
| `/fix-application` | Analisa uma app existente, planeia a reestruturação para o stack/práticas da `start-development`, executa (com aprovação) e valida com testes. |

As skills também disparam por linguagem natural (ver gatilhos abaixo) — não é obrigatório usar o comando com barra.

---

## Skill: `start-development`

Arranca um projeto novo de ponta a ponta: faz perguntas, resolve o stack, planeia as features,
confirma contigo e depois **inicializa e implementa o projeto a partir do plano — com testes reais**.
Não fica pelo scaffold: constrói a app feature a feature e só dá a tarefa por concluída com a suíte de
testes verde.

### Como invocar

- Comando: `/start-development`
- Frases-gatilho: *"novo projeto"*, *"iniciar desenvolvimento"*, *"começar um projeto"*,
  *"que stack uso"*, *"setup inicial"*, *"scaffold"*.

### O que te vai perguntar

Primeiro o **scope funcional** — o que a app faz: features principais, entidades/dados centrais,
fluxos-chave e páginas/endpoints obrigatórios (é o que alimenta o plano e a implementação). Depois as
decisões de stack:

1. **Tipo de projeto** (decide o stack-base):
   - PoC, landing page, e-commerce, site institucional/blog → **Next.js**
   - Web application interativa (dashboard, SaaS, ferramenta interna) → **React + Vite + backend**
2. **Persistência de dados?** → PostgreSQL (ligado por defeito)
3. **Armazenamento de imagens/vídeos?** → MinIO
4. **Autenticação de utilizadores?** → escolha (radio): Keycloak (OIDC), JWT próprio em cookies httpOnly, ou deixar o AI escolher
5. **Deploy em Kubernetes?** → Prometheus + OpenTelemetry
6. **Logging estruturado?** → Pino (Node) / structlog (Python)
7. **Pagamentos?** → Stripe
8. **Email transacional?** → Resend

### Stacks que escolhe

**Next.js** (PoC / sites tradicionais):
Next.js · TypeScript · Tailwind CSS + shadcn/ui + lucide-react · framer-motion · next-themes · sonner ·
Zod + React Hook Form · TanStack Query · Prisma + PostgreSQL.

**React + Vite + backend** (web applications):
React + Vite com o mesmo stack de UI acima, mais backend **Express/Node** ou **FastAPI/Python**
(decidido em função do projeto). ORM: **Prisma** (Node) ou **SQLAlchemy + Alembic** (Python).

### Módulos condicionais

Ativados conforme as respostas: `auth` (Keycloak ou JWT httpOnly), `storage` (MinIO/S3), `observability`
(Prometheus + OpenTelemetry para Kubernetes), `logging` (Pino/structlog), `payments` (Stripe),
`email` (Resend) e `compose` (serviços de dev em Docker Compose).

### Defaults transversais (sempre)

TypeScript · pnpm · ESLint + Prettier · Docker Compose para dev · Vitest + Playwright (pytest em Python).

Cada solução é gerada com uma **estrutura de pastas robusta e feature-first** (organização por
domínio, separação de camadas UI/servidor/dados e localizações previsíveis para schemas, tipos,
config e testes), definida nos ficheiros de referência de cada stack. Os testes não são só andaime: a
implementação escreve **testes reais** por feature (ver `devrothe/skills/start-development/references/testing.md`).

### Workflow

1. Entrevista (scope funcional + decisões de stack)
2. Resolução do stack
3. Plano de features (registado em tarefas)
4. Confirmação contigo (stack + plano; nada é escrito/instalado antes do "sim")
5. Scaffold (projeto + dependências + configs + `README.md`)
6. Implementação feature a feature, com testes reais
7. Verificação — lint, build, arranque **e suíte de testes verde** (gate de conclusão)

> **Regras:** o `README.md` do projeto gerado é mantido sempre sincronizado com o stack/dependências/
> env/comandos; e cada feature implementada traz testes reais, sendo a suíte verde um gate de conclusão.

---

## Skill: `test-application`

Valida e executa os testes de uma aplicação existente e, se não houver, analisa a app, planeia e cria
testes reais por vertente, executa-os (com autorização) e apresenta um relatório.

### Como invocar

- Comando: `/test-application`
- Frases-gatilho: *"testa a aplicação"*, *"corre os testes"*, *"a app tem testes?"*,
  *"valida os testes"*, *"criar testes"*, *"executar testes"*.

### Fluxo

1. **Deteta** os testes existentes e **mapeia a cobertura por vertente** (UI, backend/API, microserviço,
   serviço, CLI/biblioteca, fullstack, mobile, dados/ML).
2. Escolhe o ramo conforme a cobertura:
   - **Tudo coberto** → executa a suíte e apresenta o relatório.
   - **Nada coberto** → informa o tipo de app e a ausência de testes; pergunta (Sim/Não) se queres
     criar; se sim, planeia por vertente, cria (após aprovação), pergunta (Sim/Não) e executa tudo.
   - **Cobertura parcial** → informa que vertentes têm e não têm testes; cria (com autorização) **só as
     que faltam**; depois executa **todos os testes — existentes + acabados de criar** — e reporta.
3. **Relatório final** com totais, falhas e cobertura. Os testes criados são reais (exercem
   comportamento) — sem placeholders nem snapshots vazios.

---

## Skill: `fix-application`

Pega numa aplicação existente — tipicamente feita por alguém sem experiência — e reestrutura-a para os
fundamentos, o stack e a organização da `start-development`, corrigindo pelo caminho os bugs/erros que
já lá estavam. Reutiliza a metodologia-alvo da `start-development` e a análise/testes da
`test-application`.

### Como invocar

- Comando: `/fix-application`
- Frases-gatilho: *"reestrutura a app"*, *"alinha com a start-development"*, *"normaliza o projeto"*,
  *"refatora a organização"*, *"arranja esta aplicação"*.

### Fluxo

1. **Análise** — tipo de app e vertentes, mapa de desvios à metodologia-alvo, e deteção de bugs/erros.
2. **Plano faseado** de reestruturação (baixo risco primeiro), com uma secção dedicada aos bugs a
   corrigir.
3. **Confirmação** — apresenta plano, esforço e riscos; nada é alterado antes do "sim" (migrações
   pesadas podem ser aprovadas fase a fase).
4. **Rede de segurança** — branch/commit (ou backup) antes de mexer.
5. **Execução** incremental, preservando o comportamento e validando a cada fase; README mantido a par.
6. **Validação** — corre os testes existentes e cria os que faltarem; suíte verde + lint + build +
   arranque, e relatório final (antes → depois, bugs resolvidos, resultados dos testes).

> Reestrutura sem mudar funcionalidades — a única mudança de comportamento permitida é a correção dos
> bugs listados no plano.

---

## Desenvolvimento do plugin

- Manifesto do marketplace: `.claude-plugin/marketplace.json`
- Manifesto do plugin: `devrothe/.claude-plugin/plugin.json`
- As skills seguem as boas práticas oficiais da Anthropic: descrição em 3.ª pessoa com gatilhos,
  *progressive disclosure* (SKILL.md lean + referências a um nível de profundidade), linguagem
  imperativa e justificação do porquê das regras.

Após editar qualquer ficheiro, recarregar com `/plugin marketplace update devrothe`.
