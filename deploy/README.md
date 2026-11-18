# Deploy

This folder contains deployment and gateway configuration for the local
Dockerized saasPlug runtime.

## Purpose

The MVP uses Docker Compose for local orchestration and Nginx as the single
browser-facing gateway. Compose starts the frontend, canonical backend
services, one PostgreSQL container per stateful service, and RabbitMQ.

## Contents

| Path | Purpose |
|---|---|
| `deploy/nginx/default.conf` | Nginx reverse-proxy routes for frontend and `/api/...` service prefixes |
| `deploy/nginx/README.md` | Gateway route documentation |
| `docker-compose.yml` | Local runtime topology at repository root |
| `.env.example` | Local demonstration configuration with placeholder secrets |

## Host bindings

| Component | Host port | Purpose |
|---|---:|---|
| `nginx-gateway` | `${GATEWAY_HTTP_PORT:-8080}` | Frontend and API gateway on loopback |
| `rabbitmq` | `${RABBITMQ_MANAGEMENT_PORT:-15672}` on `127.0.0.1` | RabbitMQ management UI for local inspection |

Backend services, frontend port `5173`, PostgreSQL containers, and RabbitMQ
AMQP port are Docker-internal for the MVP.

## Gateway Routes

```text
/                       -> frontend:5173
/health                 -> nginx-gateway health response
/api/auth/...           -> auth-service:8000
/api/providers/...      -> provider-registry-service:8000
/api/provider-registry/... -> provider-registry-service:8000
/api/catalog/...        -> catalog-service:8000
/api/status/...         -> status-service:8000
/api/reservations/...   -> reservation-service:8000
/api/analytics/...      -> analytics-service:8000
/api/billing/...        -> billing-service:8000
/api/scheduler/...      -> scheduler-service:8000
```

Provider services are intentionally not exposed through Nginx. They remain
internal to provider sync, status lookup, and reservation flows.
Service-owned `/internal/jobs/...` endpoints are also not exposed through
Nginx; they are called only over the Docker network by `scheduler-service`.

## Commands

Docker Engine or Docker Desktop with the Compose plugin is required. The initial build needs access to the configured container images and package registries. Run the following from the repository root.

```bash
test -f .env || cp .env.example .env
docker compose config --quiet
docker compose up -d --build
docker compose ps
```

Smoke checks:

```bash
curl "http://localhost:${GATEWAY_HTTP_PORT:-8080}/health"
curl "http://localhost:${GATEWAY_HTTP_PORT:-8080}/api/auth/health"
curl "http://localhost:${GATEWAY_HTTP_PORT:-8080}/api/providers/health"
curl "http://localhost:${GATEWAY_HTTP_PORT:-8080}/api/catalog/health"
curl "http://localhost:${GATEWAY_HTTP_PORT:-8080}/api/status/health"
curl "http://localhost:${GATEWAY_HTTP_PORT:-8080}/api/reservations/health"
curl "http://localhost:${GATEWAY_HTTP_PORT:-8080}/api/analytics/health"
curl "http://localhost:${GATEWAY_HTTP_PORT:-8080}/api/billing/health"
curl "http://localhost:${GATEWAY_HTTP_PORT:-8080}/api/scheduler/health"
```

Stop the stack with `docker compose down`. The optional `-v` flag deletes the local PostgreSQL and RabbitMQ volumes and is appropriate only for deliberate disposal of the local environment.

## Provider and demo configuration

- Use `RED_PROVIDER_BASE_URL`, `GREEN_PROVIDER_BASE_URL`, `BLUE_PROVIDER_BASE_URL` and their corresponding `*_PROVIDER_API_KEY` values for authorized course providers. The [provider setup guide](../docs/assignment/providers/README.md) describes NTUA access. Automatic scheduler jobs and startup imports default to off. The operator can initiate an authorized catalog import.
- Optional Google OAuth uses `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GOOGLE_REDIRECT_URI`, and `GOOGLE_FRONTEND_CALLBACK_URI`. Match the callback exactly. The backend's gateway auth URLs use the repeated `/api/auth/auth/` prefix.
- The auth service seeds demonstration accounts using `DEMO_USER_PASSWORD`. The frontend build can expose that value through `VITE_DEMO_LOGIN_PASSWORD` for demo prefill. Use disposable local credentials. The sample roles include `operator@saasplug.local`, `evuser@saasplug.local`, and `red-admin@saasplug.local`, `green-admin@saasplug.local`, `blue-admin@saasplug.local`.
- For a shared demonstration, configure distinct credentials, use `APP_ENV=demo` or `SAASPLUG_STRICT_SECRETS=true` as applicable, and align the service URLs with them. Strict mode rejects placeholder authentication and internal secrets. Existing volumes retain database and broker account state after environment edits, and existing demo accounts change only through the documented reset behavior.
- When a reservation outcome is `unknown`, inspect the stored attempt before submitting with a new key. If billing or analytics lag, inspect outbox and job status. The [architecture summary](../docs/architecture-summary.md) describes the ordered maintenance sequence.

The [test guide](../tests/README.md) links the frontend, service, and gateway tests. For the event package's local tests, install the requirements in an isolated environment and run `PYTHONPATH=packages/events python -m pytest packages/events/tests -q` from the repository root.

## Deployment boundaries

Keep `.env` and real credentials outside the public project. The gateway is the
host entry point for the application. Backend services use the private Docker
network. The scheduler dispatches maintenance jobs, while user reservations
stay on the synchronous provider path and publish their terminal outcomes through
the reservation outbox.

## References

- [Project specifications](../docs/specs/SPECIFICATIONS.md)
- [Architecture summary](../docs/architecture-summary.md)
- [Deployment diagram](../docs/diagrams/final/deployment.puml)
