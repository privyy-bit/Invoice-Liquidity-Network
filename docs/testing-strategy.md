# Testing Strategy

This document is the single reference for testing Invoice Liquidity Network (ILN). It explains which test type belongs where, when it runs, and how to choose tests for a change.

The repository is a multi-workspace project. Tests should be placed as close as possible to the code they protect, while cross-service and network-dependent behavior belongs in integration or end-to-end suites.

## Test-type overview

| Test type | Typical location | Typical trigger | Owning workspace(s) |
| --- | --- | --- | --- |
| Unit tests | `packages/*/src/**/*.test.ts`, `cli/src/**/*.test.ts`, `cli/src/__tests__/**/*.test.ts`, `indexer/tests/`, `notifications/tests/`, and package-local test directories | Every PR and push through workspace test tasks | The workspace containing the implementation: SDK, CLI, indexer, notifications, shared packages, and examples |
| Contract unit tests | Smart-contract repository, in Rust test modules and contract test files | Every PR affecting contract code | `ILN-Smart-Contract` |
| Integration tests | `cli/tests/integration.test.ts`, workspace integration test directories, and service/database test suites | Every PR when dependencies are available; otherwise skipped or run in dedicated CI jobs | CLI, SDK, indexer, notifications, and the smart-contract repository |
| SDK E2E tests | `packages/sdk/e2e/` and SDK E2E workflow configuration | Every PR for relevant SDK changes; local-node coverage in a dedicated workflow | SDK workspace and local Soroban fixture |
| CLI E2E tests | CLI integration tests and CLI smoke workflows | Every PR for CLI changes; smoke workflows for command-line behavior | `cli` workspace |
| Cross-service E2E tests | `e2e/` or the relevant E2E workspace in this repository and the frontend repository | Nightly or scheduled runs; selected scenarios may run on PRs | Project-level repository, `ILN-Frontend`, SDK, indexer, notifications, and smart contracts |
| Frontend E2E tests | Frontend repository Playwright suites | Every PR for selected browser coverage and nightly for the full suite | `ILN-Frontend` |
| Load tests | `scripts/load-test.ts`, `scripts/load-test-indexer.ts`, `scripts/load-test-notifications.ts`, and `scripts/lib/load-test-harness.ts` | Scheduled or manually dispatched; not every PR | Project-level scripts, indexer, and notifications |
| Benchmark tests | `indexer/benchmark.ts` and benchmark files in the owning workspace | Scheduled, manually dispatched, or before performance-sensitive releases | `indexer` and any workspace containing a benchmark |
| Migration tests | `indexer/tests/migrations.test.ts`, migration-specific tests in the smart-contract repository, and migration workflows | Every PR changing migrations; scheduled deployment checks where a live environment is required | Indexer and `ILN-Smart-Contract` |
| Schema-snapshot tests | `indexer/tests/graphql-schema.test.ts` and generated/schema snapshot locations in the owning workspace | Every PR changing a schema; snapshot changes require review | Indexer, SDK/API consumers, and other schema-owning workspaces |
| Configuration/schema drift tests | `cli/tests/config-schema-drift.test.ts` and equivalent generated-schema tests | Every PR changing configuration types or schemas | CLI and the workspace that owns the schema |
| Compatibility tests | Root `scripts/__tests__/check-compatibility.test.ts` and the `test:compatibility` script | Every PR that changes public interfaces or package compatibility | Project-level tooling and SDK/CLI consumers |
| Smoke tests | CLI smoke workflow, SDK browser workflow, deployment and release workflows | Every PR or release workflow, depending on the surface | CLI, SDK, deployment tooling, and release tooling |
| Mutation tests | `packages/sdk/src/errors.test.ts`, `packages/sdk/stryker.config.mjs`, and `.github/workflows/mutation-testing.yml` | Advisory PR runs for mutation targets and weekly scheduled runs | SDK; Rust mutation testing remains proposed for `ILN-Smart-Contract` |
| Documentation tests | Documentation build, generated API/CLI documentation tests, link checks, and `packages/sdk/scripts/__tests__/generate-docs.test.ts` | Every documentation PR and documentation workflow | `docs`, SDK, CLI, and project-level documentation tooling |
| Security and dependency tests | CodeQL, dependency audit scripts, secret scanning, and security workflows | Every PR or scheduled security run | Project-level CI and each affected workspace |

The table describes expected ownership rather than an exhaustive list of every test file. A new test should normally follow the nearest existing convention in the workspace being changed.

## Standard test commands

Run the smallest useful check first, then broaden validation as the change warrants:

```bash
# All workspace tests
pnpm test

# Coverage-enabled workspace tests
pnpm test:coverage

# Root repository checks
pnpm test:checklist
pnpm test:compatibility
pnpm test:audit

# Load tests (manual or scheduled use)
pnpm test:load
pnpm test:load:indexer
pnpm test:load:notifications

# Build, lint, and type checking
pnpm build
pnpm lint
pnpm type-check
```

Workspace-specific commands can be run with the package manager's workspace selector or from the workspace directory. Check that workspace's `package.json` and test configuration before adding a new command.

## Decision guide: adding a feature

Use the following process when adding a feature to `X`:

1. **Write unit tests for the local behavior.** Test normal behavior, boundary values, invalid input, and failure handling. Put tests beside the implementation or in the workspace's established test directory. Keep external networks, real databases, browsers, and other services out of unit tests.

2. **Add an integration test if the feature crosses a boundary.** Add one when the feature depends on a real or in-memory database, RPC client, generated contract bindings, filesystem configuration, event stream, email/webhook adapter, or another service. Put it in the owning workspace's integration-test directory and use deterministic fixtures.

3. **Add E2E coverage for a user journey or cross-service contract.** SDK and CLI protocol flows belong in their respective E2E or smoke suites. Browser journeys belong in `ILN-Frontend`. A flow spanning the frontend, SDK, indexer, notifications, and contracts belongs in the cross-service E2E suite. Keep the PR scenario focused; leave the full matrix for scheduled runs.

4. **Add contract tests for on-chain behavior.** Changes to contract logic, authorization, storage, events, or upgrades need Rust contract tests in `ILN-Smart-Contract`. If an off-chain consumer changes too, add the corresponding SDK, CLI, or integration coverage in this repository.

5. **Add migration tests for data or contract migrations.** Test upgrades from representative prior versions, including rollback or failure behavior where supported. Update migration notes and snapshots when the migration changes a public representation.

6. **Update schema snapshots and compatibility tests for public shapes.** For GraphQL, API, configuration, generated bindings, or other public schemas, update the owning snapshot deliberately and review the diff. Add compatibility coverage when consumers could observe the change.

7. **Add a benchmark or load test when performance is part of the feature.** Use a benchmark to measure a focused operation and a load test to measure system behavior under representative concurrency or volume. Do not make either suite a prerequisite for every PR unless the workspace explicitly requires it.

8. **Consider mutation testing for critical logic.** Mutation testing is advisory and currently targeted primarily at SDK error handling. Improve assertions or add tests when an important mutation survives; do not replace ordinary unit and integration tests with mutation runs.

### Quick examples

- **New SDK helper:** unit tests beside the helper; integration or SDK E2E coverage if it calls a live RPC or contract; compatibility tests if the public API changes.
- **New CLI command:** command unit tests, configuration/schema drift coverage when applicable, and CLI integration or smoke coverage for parsing and execution.
- **New indexer event or query:** unit tests for mapping and validation, integration tests against the database or event source, and schema snapshots if the public GraphQL shape changes.
- **New notification provider:** unit tests with mocked delivery, integration tests for persistence and retry behavior, and a scheduled or manually dispatched load test if throughput matters.
- **New frontend flow:** component and state tests in `ILN-Frontend`, plus focused Playwright coverage for the critical browser journey. Add cross-service E2E coverage when the flow depends on deployed services.
- **New contract entry point or storage change:** Rust contract unit/integration tests, migration tests for upgrades, and consumer coverage in SDK/CLI or E2E suites as appropriate.

## Test design and CI principles

- Prefer deterministic tests with isolated fixtures and no reliance on shared environments.
- Keep fast unit tests independent of networks, wall-clock time, and external services.
- Use integration tests to verify real boundaries, not to duplicate every unit-test case.
- Keep E2E tests focused on observable behavior and clean up created resources.
- Treat schema, migration, snapshot, and generated-file changes as reviewable API changes.
- Document required services, credentials, seeds, and cleanup for tests that cannot run locally by default.
- A scheduled or advisory workflow supplements, but does not replace, focused tests required in a pull request.

When a test does not fit one of these categories, follow the closest existing workspace convention and explain the choice in the pull request.
