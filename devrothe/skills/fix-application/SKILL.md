---
name: fix-application
description: Analisa uma aplicação existente e reestrutura-a para seguir o stack tecnológico, os fundamentos e a organização da skill start-development. Pensada para projetos feitos por pessoas sem experiência: identifica o tipo de app e as vertentes, mapeia os desvios à metodologia-alvo, deteta bugs/erros e cria um plano de reestruturação faseado (incluindo as correções de bugs). Depois de aprovado pelo utilizador, executa o plano de forma incremental e preservando o comportamento, e no fim valida com testes (corre os existentes e cria os que faltarem) mais lint/build/arranque. Usar quando o utilizador quer alinhar/normalizar um projeto existente com as práticas da start-development, refatorar a organização/stack, ou "arranjar" uma app mal estruturada. Dispara em "fix-application", "reestrutura a app", "alinha com a start-development", "normaliza o projeto", "refatora a organização", "arranja esta aplicação".
---

# fix-application

Pegar numa aplicação existente — tipicamente feita por alguém sem experiência — e reestruturá-la para
os fundamentos, o stack e a organização da skill `start-development`, corrigindo pelo caminho os
bugs/erros que já lá estavam.

**Regra: preservar o comportamento.** A reestruturação não altera funcionalidades — é equivalência
funcional (mover, renomear, reorganizar, alinhar tecnologias). A única exceção são os bugs/erros, que
são corrigidos e têm de constar do plano. Porquê: o utilizador confia que a app continua a fazer o
mesmo; mudar features "à boleia" de um refactor torna impossível saber o que partiu.

**Regra: rede de segurança antes de mexer.** Não alterar nada sem um caminho de reversão (ver passo 4).
Porquê: refactor automatizado em código alheio é arriscado; sem histórico/backup não há como voltar
atrás.

**Regra: incremental e validado.** Avançar fase a fase e validar (lint/build/testes/arranque) ao fim
de cada uma; só passar à seguinte com a anterior verde. Porquê: erros apanham-se cedo e ficam isolados
à fase que os introduziu.

**Metodologia-alvo.** O destino é definido pela `start-development`. Consultar:
`../start-development/references/web-stack.md` e `../start-development/references/app-stack.md` (stack e
estrutura de pastas), `../start-development/references/modules.md` (auth, storage, etc.) e
`../start-development/references/testing.md` (testes). Para classificar a app e detetar/correr testes,
usar `../test-application/references/app-analysis.md` e `../test-application/references/detect-and-run.md`.

## Fluxo

Copiar este checklist para a resposta e ir marcando:

```
- [ ] 0. Contexto — ler CLAUDE.md, memória e READMEs/docs do projeto (se existirem)
- [ ] 1. Análise — stack atual, tipo de app/vertentes, desvios à metodologia-alvo e bugs/erros
- [ ] 2. Plano de reestruturação faseado, incluindo as correções de bugs detetados
- [ ] 3. Confirmar o plano com o utilizador (nada é alterado antes do "sim")
- [ ] 4. Rede de segurança — garantir git/branch ou backup
- [ ] 5. Executar o plano — fase a fase, preservando comportamento, atualizando o README
- [ ] 6. Validar — testes verdes (correr existentes; criar onde faltarem) + lint/build/arranque
```

### 0. Contexto do projeto

Antes de analisar, procurar e ler os metadados que ajudem a perceber a aplicação e a intenção original:
- `CLAUDE.md` na raiz e em subpastas, e outros ficheiros de instruções de agentes (`AGENTS.md`,
  `.cursor/rules/`, `.github/copilot-instructions.md`).
- A **memória** do projeto (preferências e decisões já registadas).
- Ficheiros de texto: `README*`, `docs/`, `CONTRIBUTING.md`, `ARCHITECTURE.md`, ADRs e notas.

Usar este contexto para informar a análise, o mapa de desvios e o plano de reestruturação. Porquê:
perceber o que a app pretende fazer (e que convenções já existem) é essencial para reestruturar sem
mudar comportamento nem contrariar decisões deliberadas.

### 1. Análise

Analisar o projeto sem o alterar. Produzir: o **tipo de aplicação** e vertentes, um **mapa de desvios**
à metodologia-alvo (linguagem, tooling, UI, ORM, estrutura de pastas, testes, módulos) e a **lista de
bugs/erros** já existentes (build/type/lint partidos, runtime, testes a falhar, anti-padrões, red flags
de segurança). Ver `references/assessment.md`.

### 2. Plano de reestruturação

A partir da análise, construir um plano **faseado** (baixo-risco primeiro), preservando comportamento.
O plano tem de incluir uma secção explícita de **bugs/erros a corrigir**. Registar as fases/tarefas com
`TaskCreate`. Ver `references/restructure-plan.md`.

### 3. Confirmar

Apresentar ao utilizador o plano completo: tipo de app, desvios, bugs a corrigir, fases, esforço e
riscos. Pedir aprovação **antes** de alterar o que quer que seja. Para migrações de elevado esforço
(ex.: troca de framework), assinalá-las e oferecer aprovação fase a fase. Não avançar sem o "sim".

### 4. Rede de segurança

Antes de qualquer edição, garantir reversibilidade (gate — ver `references/execution.md`): se o projeto
usa git, criar um branch e commitar o estado atual; se não usa, sugerir `git init` + commit inicial ou
uma cópia de segurança. Não prosseguir sem isto.

### 5. Executar

Executar o plano fase a fase, seguindo `references/execution.md`. Em cada fase: aplicar as mudanças,
preservar o comportamento, validar (lint/build/testes/arranque) e atualizar o `README.md` e as tarefas
(`TaskUpdate`). Se uma fase partir algo difícil de resolver, parar e reportar em vez de forçar.

### 6. Validar

Confirmar que tudo funciona: correr os testes existentes e **criar os que faltarem** para as vertentes
afetadas (alinhar com `../start-development/references/testing.md`); a suíte tem de ficar **verde**,
além de lint, build e arranque. Terminar com um relatório do que mudou, dos bugs corrigidos e do
resultado dos testes.

## Referências

- **`references/assessment.md`** — como analisar o projeto: tipo de app, mapa de desvios à
  metodologia-alvo e deteção de bugs/erros. Ler no passo 1.
- **`references/restructure-plan.md`** — como estruturar o plano faseado (ordem, preservação de
  comportamento, secção de bugs). Ler no passo 2.
- **`references/execution.md`** — execução segura: rede de segurança, fases, validação e relatório.
  Ler nos passos 4–6.
