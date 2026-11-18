# Local Postman Suites

`docs/postman/` contains Postman collections for the local saasPlug Docker
Compose app. All app collections use the Nginx gateway base URL:

```text
{{baseUrl}} = http://localhost:8080
```

Provider-direct course collections are external references for adapter development. See the [authorized provider guide](../assignment/providers/README.md) for the hosted course contracts.

## Files

| File | Purpose |
|---|---|
| `saasPlug.local.smoke.postman_collection.json` | Login, token capture, basic role reads, catalog capture, one negative auth check |
| `saasPlug.local.regression.postman_collection.json` | Read-heavy EV/catalog/status/provider/operator cases |
| `saasPlug.local.security.postman_collection.json` | Auth, authorization, validation, and gateway-block negative cases |
| `saasPlug.local.mutation.postman_collection.json` | Small local mutations plus manual side-effect requests skipped by default |
| `saasPlug.local.postman_environment.json` | Placeholder local environment template |
| `data/*.json` | Newman iteration data |
| `newman/README.md` | Newman output notes |

## Runner Scripts

Use the wrappers in `scripts/postman/` from the repository root:

```bash
scripts/postman/run-smoke.sh --env-file .env
scripts/postman/run-regression.sh --env-file .env
scripts/postman/run-security.sh --env-file .env
scripts/postman/run-mutation.sh --env-file .env
```

The wrappers also accept `SAASPLUG_ENV_FILE=<path>`. They read the seeded demo
password from the local env file and pass it to Newman without printing the env
file. `POSTMAN_ENV_FILE` can override the Postman environment JSON path.

The wrapper creates `reports/postman/` under the working repository when it writes results. This generated location is ignored by Git because exported environments and reports may contain local tokens.

## Safety Boundaries

- Smoke is mostly read-only but login creates refresh tokens.
- Regression is read-heavy and avoids reservation creation, provider imports,
  event drains, invoice issuance, mock payment, and scheduler job execution.
- Security uses negative cases and invalid reservation bodies/idempotency
  mismatches.
- Mutation registers a disposable EV user, exercises auth flows, reverts a
  provider display-name patch, creates/patches an inactive pricing rule, and
  pauses/resumes a scheduler job.
- Broad reservation/import/event/invoice/payment/scheduler side-effect
  requests are in manual folders and are skipped unless explicitly enabled.

## Manual Postman Use

1. Start the local stack.
2. Import the four local collections and
   `saasPlug.local.postman_environment.json`.
3. Fill only local placeholder values such as `demoPassword`.
4. Run smoke first so tokens and a catalog point can be captured.

Do not commit exported environments after running authenticated requests.
