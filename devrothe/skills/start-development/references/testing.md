# Estratégia de testes

Ler na implementação (passo 6 do SKILL.md). Cada feature construída traz testes reais que exercem o
seu comportamento. Um teste que não falha quando o comportamento parte não é um teste — evitar
snapshots vazios, asserts triviais (`expect(true).toBe(true)`) e mocks que devolvem o resultado
esperado sem exercer a lógica real.

## Contents
- O que testar por camada
- Convenções e localização
- Ferramentas por stack
- Loop de implementação
- Gate de conclusão

## O que testar por camada

- **Lógica de negócio / services / utils** → testes **unitários**: rápidos, sem I/O; mockar apenas as
  fronteiras (BD, rede). Cobrir caminho feliz + erros/edge cases relevantes.
- **Endpoints / route handlers** → testes de **integração**: chamam a app real (supertest/httpx)
  contra uma BD de teste. Verificar status, payload, validação e autorização.
- **Componentes de UI com lógica** → testes de **componente** (Testing Library): interação do
  utilizador e mudanças de estado/render, não a estrutura interna.
- **Fluxos críticos de utilizador** (login, checkout, criar/editar recurso) → **e2e** (Playwright).

Priorizar comportamento que importa; não perseguir 100% de cobertura. Cada feature do plano (passo 3)
deve sair da implementação com pelo menos os testes da sua camada principal a passar.

## Convenções e localização

- **Frontend (Next.js, React+Vite)**: unitários/componente em `tests/unit/` (ou colocados como
  `*.test.tsx` junto ao código); e2e em `tests/e2e/`.
- **Backend (Express)**: unitários em `tests/unit/`; integração (supertest sobre `app.ts`) em
  `tests/integration/`.
- **Backend (FastAPI)**: unitários em `tests/unit/`; integração (httpx sobre a app) em
  `tests/integration/`.
- Nomes: `<alvo>.test.ts` / `<alvo>.spec.ts` (JS/TS); `test_<alvo>.py` (Python).

## Ferramentas por stack

| Stack | Unit / componente | Integração | E2E |
|-------|-------------------|-----------|-----|
| Next.js / React+Vite | Vitest + @testing-library/react + jsdom | — | Playwright |
| Express/Node | Vitest | supertest | (e2e no frontend) |
| FastAPI/Python | pytest | httpx (TestClient/AsyncClient) | (e2e no frontend) |

Para integração com BD, usar uma base de dados de teste descartável (ex.: serviço Postgres dedicado do
Docker Compose ou schema isolado) e limpar o estado entre testes. Não correr testes contra a BD de dev.

## Loop de implementação

Para cada fatia do plano: escrever o código → escrever/ajustar os testes → correr → corrigir →
repetir até passar. Só depois passar à fatia seguinte. Manter a suíte sempre a compilar.

## Gate de conclusão

A implementação de uma feature só está concluída quando os seus testes passam. A tarefa global só está
concluída quando a **suíte completa está verde**, além de lint e build. Nunca marcar como feito com
testes a falhar, nem desligá-los com `skip`/`xfail`/`.only` sem justificação explícita ao utilizador.
