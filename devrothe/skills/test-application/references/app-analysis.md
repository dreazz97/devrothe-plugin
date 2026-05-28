# Análise do tipo de aplicação

Classificar a aplicação a partir do que está no projeto. Uma app pode ter **várias vertentes** (ex.:
fullstack = frontend + backend). Registar todas as que se aplicam — o plano de testes (test-plan.md)
depende disto.

## Contents
- Identificar o ecossistema
- Sinais por vertente
- Como reportar a classificação

## Identificar o ecossistema

Procurar manifestos para saber linguagem(ns) e gestor de dependências:

| Ficheiro | Ecossistema |
|----------|-------------|
| `package.json` | Node/JS/TS |
| `pyproject.toml`, `requirements.txt`, `setup.py` | Python |
| `go.mod` | Go |
| `pom.xml`, `build.gradle(.kts)` | Java/Kotlin |
| `Cargo.toml` | Rust |
| `composer.json` | PHP |
| `*.csproj`, `*.sln` | .NET |
| `Gemfile` | Ruby |

Num monorepo, podem coexistir vários — analisar cada pacote/serviço.

## Sinais por vertente

- **Frontend / UI** — dependências `react`/`vue`/`@angular/core`/`svelte`/`next`/`vite`/`astro`;
  ficheiros `.tsx`/`.jsx`/`.vue`/`.svelte`, `index.html`, `public/`, componentes, CSS/Tailwind,
  design system. Indica testes de componente e e2e de UI.
- **Backend / API** — frameworks web (`express`/`fastify`/`@nestjs`/`koa`, `fastapi`/`flask`/`django`,
  `spring-boot`, `gin`/`echo`/`fiber`, `actix`/`axum`); handlers/controllers, bootstrap de servidor
  HTTP, rotas, especificação OpenAPI/Swagger. Indica testes de integração de endpoints + unitários de
  serviços.
- **Microserviço** — responsabilidade única e pequena, deployável de forma independente; sinais:
  `Dockerfile` + manifestos Kubernetes/Helm, clientes de message broker (`kafka`/`rabbitmq`/`nats`),
  ficheiros `.proto`/gRPC, health/readiness endpoints, contratos com outros serviços. Indica testes de
  contrato, de handlers de mensagens e de integração (testcontainers).
- **Serviço / monólito** — backend maior com múltiplos domínios no mesmo deployável. Mesmos testes do
  backend, mais ênfase em fronteiras entre módulos.
- **CLI / biblioteca** — entry `bin`/`console_scripts`, sem servidor, pacote publicável, API exportada.
  Indica testes unitários da API pública e testes de invocação da CLI.
- **Fullstack** — frontend + backend no mesmo repositório. Tratar como duas vertentes.
- **Mobile** — `react-native`/`expo`, Flutter (`pubspec.yaml`). Testes de componente/widget + e2e
  (Detox/Maestro).
- **Dados / ML** — notebooks, `pandas`/`numpy`/`scikit-learn`/`torch`, scripts de treino/ETL. Testes de
  transformações de dados e de funções puras; validar shapes/contratos de dados.

Sinais auxiliares: `docker-compose.yml` (vários serviços → provável microserviços/fullstack),
`infra/`/`k8s/`/`helm/`, pipelines CI, `Makefile`.

## Como reportar a classificação

Resumir ao utilizador em poucas linhas: linguagem(ns), tipo de aplicação e a lista de vertentes
detetadas, seguido da constatação de que não existem testes. Exemplo:
"App fullstack (frontend React + Vite e backend FastAPI/REST). Sem testes no projeto."
