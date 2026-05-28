# Conditional modules

Read only the sections of the modules activated in the interview. Each module applies to the base
stack already chosen (Next.js or React+Vite+backend).

## Contents
- auth — Authentication (Keycloak or own JWT in httpOnly cookies)
- storage — File storage (MinIO / S3)
- observability — Observability on Kubernetes (Prometheus + OpenTelemetry)
- logging — Structured logging (Pino / structlog)
- payments — Payments (Stripe)
- email — Transactional email (Resend)
- compose — Dev services (Docker Compose)

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

## email — Transactional email (Resend)

For account verification, password reset, receipts/orders.

- **Node**: `pnpm add resend`. Optional: `react-email` to compose templates in React.
- **Python**: call the Resend API over HTTP (`pip install resend`).
- Keep `RESEND_API_KEY` in env. Configure a verified domain for the sender.

## compose — Dev services (Docker Compose)

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
