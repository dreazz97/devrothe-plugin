# Test detection and execution

How to find out whether there are tests, how to run them and how to interpret the results. **Always
prefer the command defined by the project** (script in the manifest) over guessing a command.

## Contents
- What to look for
- Commands per ecosystem
- Coverage
- Interpreting results

## What to look for

Three signals (any one counts as "has tests"):

1. **Test scripts** — `scripts.test`/`test:*` in `package.json`; a `test` target in a `Makefile`;
   shell test scripts; `[tool.pytest]`/`[tool.poetry.scripts]` in `pyproject.toml`.
2. **Test frameworks** in the dependencies — see the table below.
3. **Test files** — patterns: `*.test.*`, `*.spec.*`, `__tests__/`, `tests/`, `test/`, `test_*.py`,
   `*_test.py`, `*_test.go`, `*Tests.cs`, `*Test.java`.

## Commands per ecosystem

| Ecosystem | Common frameworks | Detect | Run |
|-----------|-------------------|--------|-----|
| Node/JS/TS | Vitest, Jest, Mocha, Playwright, Cypress | devDeps + `scripts.test` | `pnpm test` / `npm test` (or the defined script) |
| Python | pytest, unittest | `pytest` installed, `pytest.ini`/`pyproject` | `pytest` |
| Go | testing (stdlib) | `*_test.go` | `go test ./...` |
| Java/Kotlin | JUnit | `src/test/...` | `mvn test` / `gradle test` |
| Rust | testing (stdlib) | `#[test]` | `cargo test` |
| .NET | xUnit/NUnit/MSTest | `*Tests.csproj` | `dotnet test` |
| PHP | PHPUnit/Pest | `phpunit.xml` | `vendor/bin/phpunit` |
| Ruby | RSpec/Minitest | `spec/`/`test/` | `bundle exec rspec` / `rake test` |

If a framework needs to be **installed** (Branch B, project without tests), pick the idiomatic one for
the stack:
- Node/TS frontend: Vitest + Testing Library (+ Playwright for e2e).
- Node/TS backend: Vitest + supertest.
- Python: pytest (+ httpx for APIs).
- Go/Rust/.NET/Java: use the ecosystem's standard test framework.

## Coverage

Enable coverage when available, to enrich the report:
- Vitest: `--coverage` · Jest: `--coverage` · pytest: `--cov` (pytest-cov) · Go: `-cover` ·
  .NET: `--collect:"XPlat Code Coverage"`.

## Interpreting results

Capture from the output: total tests, passed/failed/skipped, duration and coverage (if any). List each
failure with the test name and a short reason. Do not infer "green" if the command exited with an error
code — use the exit code as the source of truth.
