---
name: test-application
description: Valida, executa e (se faltarem) cria testes para a aplicação atual. Deteta os testes existentes e mapeia a cobertura por vertente. Se tudo estiver coberto, corre os testes e reporta. Se faltarem testes — em todas ou só nalgumas vertentes (UI/frontend, backend/API, microserviço, serviço, CLI/biblioteca, fullstack, mobile, dados/ML) — informa o utilizador, propõe e (após aprovação) cria os testes em falta e, no fim e com autorização, executa todos (existentes + criados) e apresenta um relatório. Usar quando o utilizador quer correr os testes, saber se a app tem testes, validar a suíte, ou criar testes. Dispara em "test-application", "testa a aplicação", "corre os testes", "a app tem testes?", "valida os testes", "criar testes", "executar testes", "cobertura de testes".
---

# test-application

Validar e executar os testes da aplicação atual e, onde faltarem, analisar a app, planear e criar
testes reais por vertente, executá-los (com autorização) e apresentar um relatório.

**Regra: testes reais, não placeholders.** Os testes criados exercem comportamento real (caminho feliz
+ erros/edge cases). Evitar asserts triviais, snapshots vazios ou mocks que devolvem o resultado
esperado sem exercer a lógica. Porquê: uma suíte verde mas vazia dá falsa confiança.

## Fluxo

Copiar este checklist para a resposta e ir marcando. O ramo decide-se pela **cobertura por vertente**.

```
- [ ] 0. Contexto — ler CLAUDE.md, memória e READMEs/docs do projeto (se existirem)
- [ ] 1. Detetar testes + analisar vertentes → mapa de cobertura por vertente
- [ ] 2. Ramo A (todas as vertentes cobertas) → executar tudo e reportar (passo 8)
- [ ] 3. Ramos B/C (falta cobertura, total ou parcial) → informar tipo de app + vertentes com/sem testes
- [ ] 4. Perguntar (opção única Sim/Não) se quer criar os testes em falta
- [ ] 5. Plano de testes para as vertentes SEM testes → aprovação do utilizador
- [ ] 6. Criar os testes reais em falta
- [ ] 7. Perguntar (opção única Sim/Não) se pode executar
- [ ] 8. Executar TODOS (existentes + criados) e apresentar o relatório
```

### 0. Contexto do projeto

Antes de detetar, procurar e ler os metadados que ajudem a perceber a aplicação:
- `CLAUDE.md` na raiz e em subpastas, e outros ficheiros de instruções de agentes (`AGENTS.md`,
  `.cursor/rules/`, `.github/copilot-instructions.md`).
- A **memória** do projeto (preferências e decisões já registadas, incluindo as de testes).
- Ficheiros de texto: `README*`, `docs/`, `CONTRIBUTING.md`, `ARCHITECTURE.md`, ADRs e notas.

Usar este contexto para informar a análise das vertentes e o plano/criação de testes. Porquê: o README
e o CLAUDE.md costumam indicar como correr a app e os testes e que convenções seguir.

### 1. Detetar e mapear cobertura

Detetar frameworks, scripts e ficheiros de teste (ver `references/detect-and-run.md`) **e** classificar
as vertentes da aplicação (ver `references/app-analysis.md`). Cruzar os dois para construir um **mapa de
cobertura por vertente**: para cada vertente presente (UI, backend/API, microserviço, etc.), tem testes
ou não. **Não escrever nem instalar nada nesta fase** — só observar.

Decidir o ramo a partir do mapa:
- **Todas** as vertentes têm testes → **Ramo A** (passo 2).
- **Nenhuma** vertente tem testes → **Ramo B**.
- **Algumas** têm, outras não → **Ramo C** (cobertura parcial).

Os Ramos B e C partilham os passos 3–8. A única diferença é o **âmbito da criação**: no Ramo B criam-se
testes para todas as vertentes; no Ramo C apenas para as vertentes **sem** testes (as restantes já
existem e são reutilizadas).

### 2. Ramo A — tudo coberto

Correr a suíte completa usando o comando definido pelo projeto (preferir o script do projeto a
adivinhar) e seguir para o passo 8 (relatório). Não é preciso pedir autorização: o utilizador invocou a
skill para validar e executar.

Se a execução falhar por falta de serviços (ex.: base de dados), indicá-lo e sugerir como prepará-los
(ex.: `docker compose up -d`) em vez de marcar a suíte como partida.

### 3. Informar (Ramos B/C)

Apresentar ao utilizador, em poucas linhas: o **tipo de aplicação** e, do mapa de cobertura, as
vertentes **com** testes (se houver) e as vertentes **sem** testes. Exemplos:
- Ramo B: "App fullstack (React + Vite e FastAPI/REST). Sem testes em nenhuma vertente."
- Ramo C: "App fullstack: o backend/API tem testes; o frontend não tem."

### 4. Perguntar se quer criar os testes em falta

Usar `AskUserQuestion` (opção única **Sim / Não**) a perguntar se quer que sejam criados testes para as
vertentes em falta.

Se **Não**:
- Ramo C → executar os testes **existentes** e ir ao relatório (passo 8).
- Ramo B → terminar com um resumo (tipo de app + ausência de testes + recomendação); não há nada para
  correr.

Se **Sim** → continuar para o passo 5.

### 5. Plano de testes

Criar um plano **apenas para as vertentes sem testes** (ver `references/test-plan.md` para o que cobrir
em cada uma). Para cada vertente, listar os alvos/fatias e o tipo de teste (unitário, integração,
componente, e2e, contrato). Registar como tarefas com `TaskCreate`. Apresentar o plano e pedir
aprovação **antes** de escrever qualquer teste.

### 6. Criar os testes

Implementar os testes do plano aprovado. Reutilizar o framework de teste já presente no projeto (no
Ramo C há quase sempre um); se faltar, instalar o idiomático para o stack (ver
`references/detect-and-run.md`) e configurá-lo. Escrever testes reais (ver a regra acima) e manter a
suíte a compilar.

### 7. Pedir autorização para executar

Usar `AskUserQuestion` (opção única **Sim / Não**) a perguntar se pode executar agora os testes.

Se **Não**: terminar, indicando o(s) comando(s) para o utilizador correr os testes manualmente.
Se **Sim**: seguir para o passo 8.

### 8. Relatório

Executar **todos** os testes e apresentar um relatório:
- Ramo A → a suíte existente.
- Ramos B/C → **existentes + acabados de criar** (no Ramo C, os dois conjuntos juntos).

```
# Relatório de testes
Comando(s): <comando(s) executado(s)>
Total: X · Passou: Y · Falhou: Z · Ignorado: W · Duração: <t>
Cobertura: <%, se disponível>

## Falhas
- <teste> — <motivo resumido>

## Conclusão
<verde/vermelho + próximos passos sugeridos>
```

Quando os testes acabados de criar falham, distinguir no relatório se a falha é do teste ou um bug real
da aplicação.

## Referências

- **`references/app-analysis.md`** — como classificar o tipo de aplicação e as vertentes (sinais a
  procurar por ecossistema). Ler no passo 1.
- **`references/detect-and-run.md`** — deteção de frameworks/scripts/ficheiros de teste e os comandos
  para correr por ecossistema; interpretação de resultados. Ler nos passos 1, 2, 6 e 8.
- **`references/test-plan.md`** — o que testar em cada vertente da aplicação e como estruturar o plano.
  Ler nos passos 5 e 6.
