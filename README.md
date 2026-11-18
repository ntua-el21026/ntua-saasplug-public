# saasPlug — EV charging service platform

saasPlug is an NTUA Electrical and Computer Engineering coursework prototype for discovering EV charging points, checking availability, making reservations, and managing provider operations. The project demonstrates service-owned data, a synchronous external reservation boundary, and asynchronous billing and analytics over RabbitMQ.

## Engineering contribution

- **Service boundaries and data ownership.** A React and TypeScript frontend uses one Nginx gateway. Eleven FastAPI services each own a PostgreSQL database. Three internal adapters translate redPlug, greenPlug, and bluePlug provider contracts into the platform's catalog, status, and reservation workflows.
- **Reservation consistency.** The reservation service records idempotency keys and attempts, calls the provider synchronously through its adapter, and records the result. The outbox carries confirmed success and failure events to downstream services. Ambiguous outcomes remain in the reservation journal as `unknown`. Billing defers any failure charge.
- **Asynchronous processing.** RabbitMQ delivers reservation events to billing and analytics with at-least-once semantics. The consumers use event identifiers to recognize repeated delivery. A scheduler dispatches separate service-owned jobs for catalog imports, outbox publication, and aggregate refresh. Dashboards can lag the reservation record.
- **Role-based workflows.** EV users browse the map and manage reservations. Provider administrators see data scoped to their provider. Operators manage imports, pricing, invoices, scheduler runs, and system health. Billing records simulated payments. The tracked revenue metric estimates charging revenue from reservation snapshots.

Read the [implemented architecture](docs/architecture-summary.md), [service inventory](services/README.md), and [final UML](docs/diagrams/final/saasplug-final-uml.md). The [project-owned OpenAPI contract](openapi.json) and [Postman suites](docs/postman/README.md) document the application interfaces.

## Validation and scope

The [offline tests](tests/README.md) cover example configuration, reservation timeouts, confirmed outcomes, and idempotent billing with nonzero failure fees. The example configuration passes Compose parsing. The [verification record](docs/final-verification.md) covers earlier service, scheduler, and live-provider checks.

The live-provider workflow needs approved NTUA network access and provider keys. Starting the stack with the example configuration leaves automatic imports and the scheduler loop disabled. An empty catalog produces an empty map/list until an authorized import. The local stack uses role-based demo accounts and supports optional Google OAuth. The background map needs internet access.

## Local setup

The [deployment guide](deploy/README.md) explains Compose prerequisites, gateway routes, local demo accounts, and configuration safeguards. Use a disposable local configuration for demonstrations. The [frontend guide](apps/frontend/README.md) gives the development and build commands.

## Course context and attribution

Michail Athanasios Peppas developed saasPlug for NTUA coursework. The [official course references](docs/assignment/README.md) identify the original brief, requirements specification and diagram exports. The provider contracts are available [through NTUA](docs/assignment/providers/README.md). The [reading index](docs/lectures/README.md) lists the course references and the licenses that apply to retained readings.

The [documentation index](docs/README.md) separates implemented architecture, verification evidence, historical submission records, and the preserved [AI development record](docs/ai/README.md). The [MIT notice](LICENSE) applies to original project contributions. Third-party materials retain their own terms and attribution.
