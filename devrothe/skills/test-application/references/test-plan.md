# Plano de testes por vertente

O que cobrir em cada vertente identificada na análise (app-analysis.md). Construir o plano só para as
vertentes presentes. Testes reais que exercem comportamento — caminho feliz + erros/edge cases.

## Contents
- O que testar por vertente
- Estrutura do plano
- Princípios

## O que testar por vertente

- **Frontend / UI**
  - Componentes com lógica → testes de componente (Testing Library): interação do utilizador, estados,
    render condicional — não a estrutura interna nem snapshots vazios.
  - Fluxos críticos (login, checkout, criar/editar recurso) → e2e (Playwright/Cypress).
- **Backend / API**
  - Endpoints → testes de integração (supertest/httpx) contra uma BD de teste: status, payload,
    validação, autorização.
  - Lógica de negócio / serviços / utils → testes unitários (sem I/O; mockar fronteiras).
- **Microserviço**
  - Contratos de API/mensagens (request/response, schemas de eventos).
  - Handlers de mensagens (consumir → processar → produzir/efeito).
  - Integração com dependências via testcontainers (broker, BD) quando aplicável.
  - Health/readiness endpoints.
- **Serviço / monólito** — como backend, com ênfase nas fronteiras entre módulos.
- **CLI / biblioteca** — testes unitários da API pública; testes de invocação da CLI (args, exit codes,
  output).
- **Mobile** — testes de componente/widget + e2e (Detox/Maestro).
- **Dados / ML** — transformações e funções puras; validação de contratos/shapes de dados.

## Estrutura do plano

Apresentar uma tabela por vertente antes de pedir aprovação:

| Vertente | Alvo / fatia | Tipo de teste |
|----------|--------------|---------------|
| API | `POST /orders` (criação + validação) | integração |
| API | `OrderService.calcTotal` | unitário |
| UI | fluxo de checkout | e2e |

Registar as fatias como tarefas (`TaskCreate`) e implementar uma a uma.

## Princípios

- Usar o framework já presente no projeto; se não houver, instalar o idiomático (ver
  `detect-and-run.md`) e configurá-lo.
- Para integração com BD, usar uma base de dados de teste descartável e limpar o estado entre testes;
  nunca correr contra a BD de desenvolvimento.
- Priorizar comportamento que importa em vez de perseguir 100% de cobertura.
- Não desligar testes com `skip`/`xfail`/`.only` sem justificação explícita ao utilizador.
