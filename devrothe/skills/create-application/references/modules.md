# Conditional modules

Read only the sections of the modules activated in the interview. Each module applies to the base
stack already chosen (Next.js or React+Vite+backend).

## Contents
- auth — Authentication (Keycloak or own JWT in httpOnly cookies)
- storage — File storage (MinIO / S3)
- observability — Observability on Kubernetes (Prometheus + OpenTelemetry)
- logging — Structured logging (Pino / structlog)
- payments — Payments (Stripe)
- email — Email send/receive (SMTP, Microsoft Graph or Resend)
- secrets — Encrypted credential vault (DB + Fernet) — real apps
- compose — Dev services + local run mode (Docker Compose, optional app containerization)

## auth — Authentication (Keycloak or own JWT in httpOnly cookies)

The user chooses the approach in the interview: **Keycloak (OIDC)**, **own JWT (httpOnly cookies)** or
**let the AI decide**. In any option, the tokens/session live in `httpOnly`, `Secure`, `SameSite=Lax`
cookies (BFF pattern) — Keycloak does not waive good cookie practices. Why: an httpOnly cookie is not
accessible from JavaScript, which mitigates token theft via XSS, unlike storing tokens in
`localStorage`. `SameSite` mitigates CSRF; for cross-site flows, add an anti-CSRF token.

### When the AI decides

- **Keycloak** when: SSO / centralized identity is needed, several apps/services share login, social
  login or federation (OIDC/SAML), user and role management outside the app (admin UI),
  enterprise/compliance requirements, or the organization already uses Keycloak.
- **Own JWT** when: a single, simpler app, no external IdP, minimal infrastructure and full control of
  the flow, with no need for SSO/federation.

### Option A — Own JWT (httpOnly cookies)

- **Node/Next.js**: `pnpm add jose bcrypt` (`jose` to sign/verify JWT; `bcrypt` or `argon2` to hash
  passwords). In Next.js, set the cookie via `cookies()` in Server Actions / Route Handlers. Never
  return the token in the response body.
- **Express**: `pnpm add jsonwebtoken cookie-parser bcrypt`. Set the cookie with
  `res.cookie(name, token, { httpOnly: true, secure: true, sameSite: "lax" })`.
- **FastAPI**: `pip install pyjwt "passlib[bcrypt]"`. Set the cookie with
  `response.set_cookie(..., httponly=True, secure=True, samesite="lax")`.

Keep the signing secret in an environment variable. Set a short expiry on the access token and, if
needed, a rotating refresh token.

### Option B — Keycloak (OIDC)

Keycloak is an open-source Identity Provider (OIDC/OAuth2/SAML). It runs as its own service (add it to
Docker Compose in dev — see the `compose` section) with Postgres as its DB. Create a *realm* and a
*client*; use the **Authorization Code + PKCE** flow. Always validate tokens against Keycloak's JWKS
(`/realms/<realm>/protocol/openid-connect/certs`) — never trust a token without verifying the
signature.

- **Next.js**: use Auth.js (`pnpm add next-auth`) with the Keycloak provider; the session lives in an
  httpOnly cookie. Low-level alternative: `openid-client`.
- **React + Vite (SPA)**: `pnpm add oidc-client-ts` (or `keycloak-js`) with Authorization Code + PKCE.
  Prefer the BFF pattern (the backend stores the tokens in an httpOnly cookie) over storing tokens in
  the browser.
- **Express**: validate the access token against the JWKS with `jose` (`createRemoteJWKSet`);
  authorize by the token's role/scope.
- **FastAPI**: validate the JWT against the JWKS with `authlib` (or `python-jose` + `httpx` to fetch
  the keys); read roles/claims for authorization.

Configure via env: `KEYCLOAK_ISSUER` (realm URL), `KEYCLOAK_CLIENT_ID` and, for confidential clients,
`KEYCLOAK_CLIENT_SECRET`. Do not use Keycloak's default admin credentials in production.

## storage — File storage (MinIO / S3)

MinIO is open-source and S3-API compatible, so you use the AWS S3 SDK — no lock-in, and in production
it can be swapped for any S3.

- Add the `minio` service to Docker Compose (see the `compose` section).
- **Node**: `pnpm add @aws-sdk/client-s3 @aws-sdk/s3-request-presigner`. Configure the client with
  `endpoint` pointing at MinIO, `forcePathStyle: true` and credentials via env.
- **Python**: `pip install boto3`. Client `boto3.client("s3", endpoint_url=...)`.

Prefer **presigned URLs**: the backend generates a signed URL and the client uploads/downloads
directly to MinIO. Why: avoids passing large files through the backend (less memory/latency) and keeps
the credentials on the server. Validate file type and size before signing.

## observability — Observability on Kubernetes (Prometheus + OpenTelemetry)

Enable when the deploy is on Kubernetes. OpenTelemetry for traces/metrics/logs; Prometheus scrapes a
`/metrics` endpoint.

- **Node**: `pnpm add @opentelemetry/sdk-node @opentelemetry/auto-instrumentations-node prom-client`.
  Initialize the OTel SDK before the rest of the app; expose `/metrics` with `prom-client`.
- **Python/FastAPI**: `pip install opentelemetry-distro opentelemetry-instrumentation-fastapi
  prometheus-client` (and `opentelemetry-bootstrap -a install`). Expose `/metrics`.
- Configure the OTLP exporter to point at the collector via env (`OTEL_EXPORTER_OTLP_ENDPOINT`).
- On Kubernetes, create a `ServiceMonitor` (Prometheus Operator) or scrape annotations for the
  `/metrics` endpoint. Add liveness/readiness probes.

## logging — Structured logging (Pino / structlog)

Structured logs (JSON) are searchable and correlatable with traces; prefer them over `console.log`.

- **Node**: `pnpm add pino` and `pnpm add -D pino-pretty`. Use JSON in production and `pino-pretty` in
  dev only. Include a request-id per request and never log secrets/PII.
- **Python/FastAPI**: `pip install structlog`. Configure a JSON processor in production.
- If the `observability` module is active, propagate the trace/span id into the logs for correlation.

## payments — Payments (Stripe)

- **Node**: `pnpm add stripe` (and `@stripe/stripe-js` + `@stripe/react-stripe-js` on the frontend if
  using Elements). **Python**: `pip install stripe`.
- All sensitive logic (create PaymentIntent/Checkout Session) runs on the server; the secret key never
  reaches the client.
- Implement the **webhook** and **verify the signature** (`stripe.webhooks.constructEvent`). Why:
  without verification, anyone could forge payment events. Handle idempotency (Stripe may resend
  events).
- Keep `STRIPE_SECRET_KEY` and `STRIPE_WEBHOOK_SECRET` in env.

## email — Email send/receive (SMTP, Microsoft Graph or Resend)

Covers transactional email (account verification, password reset, receipts/orders) and — via Microsoft
Graph — reading/receiving mail. The provider choice and where the credentials live **depend on the
PoC/demo gate** (see `security.md`), because they trade simplicity for production fit.

### PoC/demo — keep it simple

One transport, credentials in `.env`. Default to **Resend** (or SMTP if the user already has a relay).
No vault, no DB config — speed over flexibility.

- **Node**: `pnpm add resend`. Optional: `react-email` to compose templates in React.
- **Python**: call the Resend API over HTTP (`pip install resend`).
- Keep `RESEND_API_KEY` (or the SMTP creds) in env; configure a verified sender domain.

### Real app — pick the provider, store creds in the vault

Ask the user (`AskUserQuestion`, single choice) which provider: **SMTP**, **Microsoft Graph** or
**Resend**. For a real app the provider's credentials are **not** kept in `.env` — they live in the
**encrypted credential vault** (DB + Fernet; see the `secrets` section below). Why: a real app usually
needs the mail config to be changeable at runtime (an admin screen) and, in multi-tenant apps,
different credentials per tenant — neither fits a redeploy-to-change `.env`. The one exception is the
Fernet key itself, which stays in env (it bootstraps the vault).

Build the integration behind a small `EmailProvider` interface (`send(...)`, and for Graph optionally
`listMessages(...)`) so the chosen provider is swappable and testable with a fake.

**SMTP** — works with any relay (Postfix, SES SMTP, Mailgun, a corporate server).
- **Node**: `pnpm add nodemailer`. **Python**: `aiosmtplib` (async) or stdlib `smtplib`.
- Vault keys: host, port, username, password, from-address, TLS mode.

**Microsoft Graph** — Microsoft 365 / Exchange Online; supports both send and receive.
- App registration in Entra ID: tenant id, client id, client secret (or certificate), using the
  **client-credentials** flow (app-only). Grant `Mail.Send` (and `Mail.Read` for inbound) application
  permissions with admin consent — scope down to specific mailboxes with Application Access Policies.
- **Node**: `pnpm add @azure/msal-node @microsoft/microsoft-graph-client`. **Python**: `pip install
  msal httpx` (or `msgraph-sdk`).
- Send: `POST /users/{mailbox}/sendMail`. **Receiving (capability note):** read with
  `GET /users/{mailbox}/messages`, or subscribe to change notifications
  (`POST /subscriptions` → your webhook) for near-real-time inbound. Treat receive as opt-in — wire it
  only when the feature needs inbound mail; validate the subscription `clientState` and renew the
  subscription before it expires.
- Vault keys: tenant id, client id, client secret, default from/mailbox.

**Resend** — same SDK as the PoC path, but read `RESEND_API_KEY` from the vault, not `.env`.

Validate the active provider's config at startup (or on first send) and fail with a clear message if it
is missing — same fail-fast spirit as env validation.

## secrets — Encrypted credential vault (DB + Fernet)

A reusable place to keep **runtime/integration credentials encrypted at rest in the database** instead
of `.env`. Introduced for the `email` module (real-app path) but written to be reused by any module
that holds admin-configurable or per-tenant credentials (e.g. `payments`, `storage`). Gated on the
PoC/demo decision — a PoC keeps its few secrets in `.env`; a real app uses the vault for credentials
that must be changeable at runtime.

**What goes where.** This does not replace env wholesale — it splits secrets by lifecycle:
- **Env (unchanged):** bootstrap/infra secrets the app needs to start — `DATABASE_URL`, the JWT signing
  secret, and the **Fernet key** itself. These are deploy-time, not user-editable.
- **Vault (DB + Fernet):** credentials configured *inside* the running app or differing per tenant —
  email provider creds, third-party API keys managed from an admin UI. Encrypted at rest; the row holds
  ciphertext, never plaintext.

**Fernet.** Symmetric authenticated encryption (AES-128-CBC + HMAC-SHA256) — encrypt on write, decrypt
in memory only when the secret is used. The key is a single bootstrap secret in env
(`APP_ENCRYPTION_KEY`), generated with `Fernet.generate_key()` and validated at startup (fail fast if
missing). Support rotation with `MultiFernet` (new key first for encrypt; old keys still decrypt until
everything is re-encrypted).

- **Python**: `pip install cryptography` → `from cryptography.fernet import Fernet, MultiFernet`.
- **Node**: `pnpm add fernet` (Fernet-compatible, so tokens interoperate with the Python lib). Keep the
  same key format if both stacks ever touch the same vault.

**Schema** (Prisma/SQLAlchemy) — one row per stored secret:
- `id`, optional `tenantId`/`scope` (per-tenant or global), `name` (e.g. `email.smtp.password`),
  `ciphertext` (the Fernet token), `createdAt`/`updatedAt`.
- Wrap it in a small service: `setSecret(name, plaintext)` encrypts and upserts; `getSecret(name)`
  decrypts. Cache decrypted values in memory only (short TTL), never on disk.

**Rules.** Never log or return decrypted values; the admin UI is **write-only** (show "configured",
allow replace, never reveal). Plaintext credentials in the DB are a finding, same as secrets in code
(see `security.md`). The Fernet key is the crown jewel — losing it means re-entering every secret,
leaking it means the vault is readable; keep it out of git and rotate the stored secrets if it ever
leaks.

## compose — Dev services + local run mode (Docker Compose)

This module covers two things: the **dev services** (always, when the project has runtime
dependencies) and the **local run mode** chosen in the interview, which decides whether the application
*itself* is containerized.

### Local run mode (interview question 8)

| Mode | What is generated | `docker compose up` brings up | App runs via |
|------|-------------------|-------------------------------|--------------|
| **A — Services in Docker, app native** (default) | only the services compose | Postgres / MinIO / Keycloak | `pnpm dev` / `uvicorn --reload` on the host |
| **B — Fully containerized** | services compose **+ app `Dockerfile` + `app` service** | everything (app + services) | the `app` container |
| **C — Both** | the Dockerfile + `app` service, *and* native dev stays working | services by default; everything with the `full` profile | either — host or container |

Why offer the choice: a native app has the fastest inner loop (instant hot-reload, native debugger,
no rebuild) and is the simplest default; a containerized app gives prod-parity and a single
`docker compose up` for the whole stack (good for onboarding and demos), at the cost of a heavier loop.
"Both" is the most flexible but adds maintenance surface (the Dockerfile must be kept in sync).

The one rule that makes every mode work: **read service hosts from env, never hardcode them.** Natively
the app reaches services on `localhost:<port>`; inside a container it reaches them by the Compose
service name (`postgres`, `minio`, `keycloak`). So `DATABASE_URL` / `S3_ENDPOINT` / `KEYCLOAK_ISSUER`
come from env, with `localhost` defaults for native dev and the service-name value injected into the
`app` service. Apply this whenever any containerization mode (B or C) is chosen.

### Dev services (all modes)

Bring up the runtime dependencies locally for a reproducible environment. Include only the services
needed (Postgres whenever there is a DB; MinIO if the `storage` module is active; Keycloak if
authentication is via Keycloak).

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

  # Only if the storage module is active
  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    ports: ["9000:9000", "9001:9001"]
    volumes: ["miniodata:/data"]

  # Only if authentication is via Keycloak
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    command: start-dev
    environment:
      KC_BOOTSTRAP_ADMIN_USERNAME: admin
      KC_BOOTSTRAP_ADMIN_PASSWORD: admin
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/app
      KC_DB_USERNAME: app
      KC_DB_PASSWORD: app
    ports: ["8080:8080"]
    depends_on: [postgres]

volumes:
  pgdata:
  miniodata:
```

The credentials above are for dev only. In production, use managed secrets and never the default
values.

### App containerization (modes B and C only)

Skip this whole subsection for mode A. For modes B and C, add a `Dockerfile` and an `app` service.

**`Dockerfile`** — multi-stage, with a small dev stage (hot-reload over a bind mount) and a lean,
non-root production stage. Per stack:

- **Node / Next.js / Express**: base on `node:22-slim`. Dev stage installs all deps and runs the dev
  server (`pnpm dev`); prod stage runs `pnpm build` then ships only production deps and the build
  output, started with `pnpm start` (Next.js: prefer `output: "standalone"` to shrink the image). Run
  as a non-root user.
- **React + Vite (static frontend)**: build with `pnpm build` and serve the static `dist/` behind
  **nginx** (`nginx:alpine`) in prod; in dev, run the Vite dev server (`pnpm dev --host`) so HMR works
  through the bind mount.
- **FastAPI / Python**: base on `python:3.12-slim`. Install deps into a venv; dev runs
  `uvicorn --reload`, prod runs `uvicorn` (or `gunicorn -k uvicorn.workers.UvicornWorker`) as a
  non-root user.

Always add a `.dockerignore` (at least `node_modules`, `.git`, `.env`, `dist`/`.next`, test/coverage
output) so the build context stays small and no secrets leak into the image.

**`app` service** — add it to the same Compose file. For dev, bind-mount the source for hot-reload and
point the service hosts at the Compose service names via env:

```yaml
  app:
    build:
      context: .
      target: dev            # the dev stage of the Dockerfile
    environment:
      # service names, not localhost — this container talks to the others over the compose network
      DATABASE_URL: postgresql://app:app@postgres:5432/app
      # S3_ENDPOINT: http://minio:9000        # if storage is active
      # KEYCLOAK_ISSUER: http://keycloak:8080/realms/<realm>   # if auth via Keycloak
    ports: ["3000:3000"]     # Next/Express; Vite: 5173:5173; FastAPI: 8000:8000
    volumes:
      - .:/app               # source for hot-reload
      - /app/node_modules    # keep the image's installed deps (Node)
    depends_on: [postgres]   # add minio / keycloak when active
    profiles: ["full"]       # mode C only — see below
```

**Mode B (fully containerized):** drop the `profiles` line so the `app` service comes up with a plain
`docker compose up`. The README documents `docker compose up` as the way to run everything.

**Mode C (both):** keep `profiles: ["full"]` on the `app` service. Then `docker compose up` brings up
**only the services** (native dev unaffected), and `docker compose --profile full up` brings up the app
too. Because hosts come from env (`localhost` defaults for native, service names injected into the
`app` service), the same code runs in both. The README documents both paths.
