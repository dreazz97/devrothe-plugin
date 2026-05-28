# Deteção e execução de testes

Como descobrir se há testes, como corrê-los e como interpretar os resultados. **Preferir sempre o
comando definido pelo projeto** (script no manifesto) a adivinhar um comando.

## Contents
- O que procurar
- Comandos por ecossistema
- Cobertura
- Interpretar resultados

## O que procurar

Três sinais (qualquer um conta como "tem testes"):

1. **Scripts de teste** — `scripts.test`/`test:*` no `package.json`; alvo `test` num `Makefile`;
   scripts shell de teste; `[tool.pytest]`/`[tool.poetry.scripts]` no `pyproject.toml`.
2. **Frameworks de teste** nas dependências — ver tabela abaixo.
3. **Ficheiros de teste** — padrões: `*.test.*`, `*.spec.*`, `__tests__/`, `tests/`, `test/`,
   `test_*.py`, `*_test.py`, `*_test.go`, `*Tests.cs`, `*Test.java`.

## Comandos por ecossistema

| Ecossistema | Frameworks comuns | Detetar | Correr |
|-------------|-------------------|---------|--------|
| Node/JS/TS | Vitest, Jest, Mocha, Playwright, Cypress | devDeps + `scripts.test` | `pnpm test` / `npm test` (ou o script definido) |
| Python | pytest, unittest | `pytest` instalado, `pytest.ini`/`pyproject` | `pytest` |
| Go | testing (stdlib) | `*_test.go` | `go test ./...` |
| Java/Kotlin | JUnit | `src/test/...` | `mvn test` / `gradle test` |
| Rust | testing (stdlib) | `#[test]` | `cargo test` |
| .NET | xUnit/NUnit/MSTest | `*Tests.csproj` | `dotnet test` |
| PHP | PHPUnit/Pest | `phpunit.xml` | `vendor/bin/phpunit` |
| Ruby | RSpec/Minitest | `spec/`/`test/` | `bundle exec rspec` / `rake test` |

Se for preciso **instalar** um framework (Ramo B, projeto sem testes), escolher o idiomático do stack:
- Node/TS frontend: Vitest + Testing Library (+ Playwright para e2e).
- Node/TS backend: Vitest + supertest.
- Python: pytest (+ httpx para APIs).
- Go/Rust/.NET/Java: usar o framework de teste padrão do ecossistema.

## Cobertura

Ativar cobertura quando disponível, para enriquecer o relatório:
- Vitest: `--coverage` · Jest: `--coverage` · pytest: `--cov` (pytest-cov) · Go: `-cover` ·
  .NET: `--collect:"XPlat Code Coverage"`.

## Interpretar resultados

Capturar do output: total de testes, passou/falhou/ignorado, duração e cobertura (se houver). Listar
cada falha com o nome do teste e o motivo resumido. Não inferir "verde" se o comando saiu com código de
erro — usar o exit code como fonte de verdade.
