# Módulos condicionais

Ler apenas as secções dos módulos ativados na entrevista. Cada módulo aplica-se ao stack-base já
escolhido (Next.js ou React+Vite+backend).

## Contents
- auth — Autenticação (JWT em cookies httpOnly)
- storage — Armazenamento de ficheiros (MinIO / S3)
- observability — Observabilidade em Kubernetes (Prometheus + OpenTelemetry)
- logging — Logging estruturado (Pino / structlog)
- payments — Pagamentos (Stripe)
- email — Email transacional (Resend)
- compose — Serviços de dev (Docker Compose)

## auth — Autenticação (JWT em cookies httpOnly)

O token JWT vive num cookie `httpOnly`, `Secure`, `SameSite=Lax` (ou `Strict`). Porquê: um cookie
httpOnly não é acessível por JavaScript, o que mitiga roubo de token por XSS — ao contrário de guardar
o token em `localStorage`. Com `SameSite` mitiga-se CSRF; para fluxos cross-site, acrescentar um token
anti-CSRF.

- **Node/Next.js**: `pnpm add jose bcrypt` (`jose` para assinar/verificar JWT; `bcrypt` ou `argon2`
  para hash de passwords). Em Next.js, definir o cookie via `cookies()` nas Server Actions / Route
  Handlers. Nunca devolver o token no corpo da resposta.
- **Express**: `pnpm add jsonwebtoken cookie-parser bcrypt`. Definir o cookie com
  `res.cookie(name, token, { httpOnly: true, secure: true, sameSite: "lax" })`.
- **FastAPI**: `pip install pyjwt "passlib[bcrypt]"`. Definir o cookie com
  `response.set_cookie(..., httponly=True, secure=True, samesite="lax")`.

Guardar o segredo de assinatura em variável de ambiente. Definir expiração curta no access token e,
se necessário, um refresh token rotativo.

## storage — Armazenamento de ficheiros (MinIO / S3)

MinIO é open-source e compatível com a API S3, por isso usa-se o SDK da AWS S3 — não há lock-in e em
produção pode trocar-se por qualquer S3.

- Adicionar o serviço `minio` ao Docker Compose (ver secção `compose`).
- **Node**: `pnpm add @aws-sdk/client-s3 @aws-sdk/s3-request-presigner`. Configurar o cliente com
  `endpoint` a apontar para o MinIO, `forcePathStyle: true` e credenciais por env.
- **Python**: `pip install boto3`. Cliente `boto3.client("s3", endpoint_url=...)`.

Preferir **presigned URLs**: o backend gera um URL assinado e o cliente faz upload/download direto ao
MinIO. Porquê: evita passar ficheiros grandes pelo backend (menos memória/latência) e mantém as
credenciais no servidor. Validar tipo e tamanho do ficheiro antes de assinar.

## observability — Observabilidade em Kubernetes (Prometheus + OpenTelemetry)

Ativar quando o deploy é em Kubernetes. OpenTelemetry para traces/métricas/logs; Prometheus faz scrape
de um endpoint `/metrics`.

- **Node**: `pnpm add @opentelemetry/sdk-node @opentelemetry/auto-instrumentations-node prom-client`.
  Inicializar o SDK do OTel antes do resto da app; expor `/metrics` com `prom-client`.
- **Python/FastAPI**: `pip install opentelemetry-distro opentelemetry-instrumentation-fastapi
  prometheus-client` (e `opentelemetry-bootstrap -a install`). Expor `/metrics`.
- Configurar o exporter OTLP a apontar para o collector via env (`OTEL_EXPORTER_OTLP_ENDPOINT`).
- Em Kubernetes, criar um `ServiceMonitor` (Prometheus Operator) ou anotações de scrape para o
  endpoint `/metrics`. Adicionar liveness/readiness probes.

## logging — Logging estruturado (Pino / structlog)

Logs estruturados (JSON) são pesquisáveis e correlacionáveis com traces; preferir a `console.log`.

- **Node**: `pnpm add pino` e `pnpm add -D pino-pretty`. Usar JSON em produção e `pino-pretty` só em
  dev. Incluir um request-id por pedido e nunca registar segredos/PII.
- **Python/FastAPI**: `pip install structlog`. Configurar processador JSON em produção.
- Se o módulo `observability` estiver ativo, propagar o trace/span id nos logs para correlação.

## payments — Pagamentos (Stripe)

- **Node**: `pnpm add stripe` (e `@stripe/stripe-js` + `@stripe/react-stripe-js` no frontend, se usar
  Elements). **Python**: `pip install stripe`.
- Toda a lógica sensível (criar PaymentIntent/Checkout Session) corre no servidor; a chave secreta
  nunca chega ao cliente.
- Implementar o **webhook** e **verificar a assinatura** (`stripe.webhooks.constructEvent`). Porquê:
  sem verificação, qualquer um podia forjar eventos de pagamento. Tratar idempotência (o Stripe pode
  reenviar eventos).
- Guardar `STRIPE_SECRET_KEY` e `STRIPE_WEBHOOK_SECRET` em env.

## email — Email transacional (Resend)

Para verificação de conta, recuperação de password, recibos/encomendas.

- **Node**: `pnpm add resend`. Opcional: `react-email` para compor templates em React.
- **Python**: chamar a API do Resend via HTTP (`pip install resend`).
- Guardar `RESEND_API_KEY` em env. Configurar um domínio verificado para o remetente.

## compose — Serviços de dev (Docker Compose)

Subir as dependências de runtime localmente para um ambiente reproduzível. Incluir apenas os serviços
necessários (Postgres sempre que houver BD; MinIO se o módulo `storage` estiver ativo).

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: app
    ports: ["5432:5432"]
    volumes: ["pgdata:/var/lib/postgresql/data"]

  # Só se o módulo storage estiver ativo
  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    ports: ["9000:9000", "9001:9001"]
    volumes: ["miniodata:/data"]

volumes:
  pgdata:
  miniodata:
```

As credenciais acima são apenas para dev. Em produção, usar segredos geridos e nunca os valores por
defeito.
