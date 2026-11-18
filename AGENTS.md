# Working on saasPlug

- Start with the [project overview](README.md), [architecture](docs/architecture-summary.md), and the README and tests for the affected service or package.
- Keep service data ownership explicit. The frontend calls the Nginx gateway. Provider adapters own external contracts, and stateful services own their databases. Preserve stable event IDs and idempotent consumers.
- Use [the deployment guide](deploy/README.md) and `.env.example` for local configuration. Keep real credentials in local configuration. Live provider access requires authorization.
- Run the relevant [offline tests](tests/README.md) before changing service or API behavior. Keep mock billing and provider fixtures separate from live operations.
- The [AI development record](docs/ai/README.md) preserves the original coursework instructions and run logs as historical evidence.
