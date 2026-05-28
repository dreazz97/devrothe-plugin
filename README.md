# Devrothe

Plugin de **skills de apoio ao desenvolvimento** para o [Claude Code](https://code.claude.com).
Distribuído como um marketplace local (sem necessidade de GitHub para uso pessoal).

Estado atual: 1 skill — [`start-development`](#skill-start-development).

---

## Conteúdo

```
Devrothe Plugin/
├── .claude-plugin/marketplace.json     # manifesto do marketplace
└── devrothe/                           # plugin
    ├── .claude-plugin/plugin.json
    └── skills/
        └── start-development/
            ├── SKILL.md                # entrevista + árvore de decisão + workflow
            └── references/
                ├── web-stack.md        # stack Next.js
                ├── app-stack.md        # stack React+Vite + backend
                └── modules.md          # módulos condicionais
```

---

## Instalação

### Local (recomendado para uso/desenvolvimento)

Não é preciso GitHub. A partir do Claude Code:

```
/plugin marketplace add "/Volumes/Samsung PSSD T7 Shield Media/Novlok/Devrothe Plugin"
/plugin install devrothe@devrothe
```

O formato de instalação é `plugin@marketplace` — aqui ambos se chamam `devrothe`.

### A partir do GitHub (para partilhar/versionar)

Depois de fazer push do repositório:

```
/plugin marketplace add <owner>/<repo>
/plugin install devrothe@devrothe
```

### Atualizar após alterações

```
/plugin marketplace update devrothe
```

---

## Comandos / Skills

| Comando | O que faz |
|---------|-----------|
| `/start-development` | Entrevista o utilizador, escolhe o stack tecnológico adequado e faz o scaffold do projeto. |

As skills também disparam por linguagem natural (ver gatilhos abaixo) — não é obrigatório usar o comando com barra.

---

## Skill: `start-development`

Arranca um projeto novo de forma guiada: faz perguntas, resolve o stack a partir das respostas,
confirma contigo e só depois inicializa o projeto, instala dependências e gera as configs.

### Como invocar

- Comando: `/start-development`
- Frases-gatilho: *"novo projeto"*, *"iniciar desenvolvimento"*, *"começar um projeto"*,
  *"que stack uso"*, *"setup inicial"*, *"scaffold"*.

### O que te vai perguntar

1. **Tipo de projeto** (decide o stack-base):
   - PoC, landing page, e-commerce, site institucional/blog → **Next.js**
   - Web application interativa (dashboard, SaaS, ferramenta interna) → **React + Vite + backend**
2. **Persistência de dados?** → PostgreSQL (ligado por defeito)
3. **Armazenamento de imagens/vídeos?** → MinIO
4. **Autenticação de utilizadores?** → JWT em cookies httpOnly
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

Ativados conforme as respostas: `auth` (JWT httpOnly), `storage` (MinIO/S3), `observability`
(Prometheus + OpenTelemetry para Kubernetes), `logging` (Pino/structlog), `payments` (Stripe),
`email` (Resend) e `compose` (serviços de dev em Docker Compose).

### Defaults transversais (sempre)

TypeScript · pnpm · ESLint + Prettier · Docker Compose para dev · Vitest + Playwright (pytest em Python).

### Workflow

1. Entrevista
2. Resolução do stack
3. Confirmação contigo (nada é escrito/instalado antes do "sim")
4. Scaffold (projeto + dependências + configs + `README.md`)
5. Verificação (instala, faz lint, arranca)

> **Regra:** o `README.md` do projeto gerado é criado no scaffold e mantido sempre sincronizado com
> qualquer alteração de stack, dependências, variáveis de ambiente ou comandos de arranque.

---

## Desenvolvimento do plugin

- Manifesto do marketplace: `.claude-plugin/marketplace.json`
- Manifesto do plugin: `devrothe/.claude-plugin/plugin.json`
- As skills seguem as boas práticas oficiais da Anthropic: descrição em 3.ª pessoa com gatilhos,
  *progressive disclosure* (SKILL.md lean + referências a um nível de profundidade), linguagem
  imperativa e justificação do porquê das regras.

Após editar qualquer ficheiro, recarregar com `/plugin marketplace update devrothe`.
