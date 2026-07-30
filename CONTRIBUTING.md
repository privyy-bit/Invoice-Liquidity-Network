# Contributing to Invoice Liquidity Network

Thank you for your interest in contributing. Invoice Liquidity Network (ILN) is a multi-repository project. This guide explains how the three-repo structure works, where to open issues, how PRs are reviewed, how decisions are made, and how the Drips Wave model works.

---

## Project structure

- [Ways to contribute](#ways-to-contribute)
- [Applying to work on an issue](#applying-to-work-on-an-issue)
- [Project board](#project-board)
- [Development setup](#development-setup)
- [CI/CD pipeline reference](#cicd-pipeline-reference)
- [Pre-commit security scan](#pre-commit-security-scan)
- [Submitting a pull request](#submitting-a-pull-request)
- [Branch protection](#branch-protection)
- [Code standards](#code-standards)
- [Automated dependency updates](#automated-dependency-updates)
- [Getting help](#getting-help)

| Repository | Purpose | Typical contributions |
|------------|---------|-----------------------|
| `Invoice-Liquidity-Network` | Project-level repo: shared docs, SDK, CLI, indexer, notifications, repo tooling, developer guides | SDK, CLI, docs, indexer improvements, notifications, repo workflows, shared tests |
| `ILN-Frontend` | Frontend dApp: freelancer dashboard, LP analytics, governance UI, visual polish | UI, UX, styles, React components, frontend integration |
| `ILN-Smart-Contract` | Soroban / Rust smart contracts, on-chain invoice lifecycle, contract tests | contract logic, on-chain validations, Rust tests, protocol security |

This document is the entry point for first-time contributors and for anyone who wants to work across repos.

---

## Where to contribute

Start by choosing the right repo for the issue or improvement.

- **Bug in contract behavior or on-chain logic** → `ILN-Smart-Contract`
- **Visual issue, layout bug, or frontend flow problem** → `ILN-Frontend`
- **SDK, CLI, docs, indexer, notifications, or shared repository tooling** → `Invoice-Liquidity-Network`
- **Governance process, roadmap, coordination, or project-level policy** → `Invoice-Liquidity-Network`

| Label | Meaning |
| --- | --- |
| `help wanted` | High priority, no funding attached |
| `good first issue` | Well-scoped, good entry point |
| `in progress` | Already claimed, do not apply |

If you are unsure or the work spans multiple repos, open the issue in `Invoice-Liquidity-Network` and clearly explain the affected repo(s). Maintainers will help route it.

---

## Drips Wave contribution model

The Drips Wave system is our project prioritization and complexity model. Every issue is assigned a Wave point value during triage.

### How points are assigned

- `1 point` — small docs updates, typo fixes, minor test cleanups
- `2 points` — small bug fixes, minor frontend polish, SDK/CLI small improvements
- `3 points` — medium bug fixes, new helper behavior, contract interface updates, documentation with code changes
- `4 points` — new feature in one repo, significant UX flow changes, contract + SDK coordination
- `5+ points` — large cross-repo work, major architecture changes, governance or protocol enhancements

Maintainers assign points during issue triage and use them to group work in Waves. If you are new, ask for “Drips Wave points” in the issue comment and maintainers will assign the appropriate complexity level.

### Why it matters

- It helps contributors choose work at the right size
- It makes review and planning easier
- It keeps PRs focused and aligned with project priorities

When you open or apply to work on an issue, include the Wave points if available.

---

## Getting started (first-time contributor)

### Prerequisites

- Node.js 18+
- `pnpm` 9+
- Rust 1.74+
- Docker
- Stellar CLI

### Clone the project with submodules

```bash
git clone --recurse-submodules https://github.com/Invoice-Liquidity-Network/Invoice-Liquidity-Network.git
cd Invoice-Liquidity-Network
git submodule update --init --recursive
pnpm install
```

### Package manager policy

This repository is a single **pnpm workspace** (see `pnpm-workspace.yaml`). The root `pnpm-lock.yaml` is the only lockfile that should ever exist in this repo.

- Always use `pnpm install` / `pnpm add` — never `npm install` or `yarn add`, even inside an individual package such as `cli/`, `sdk/`, or `indexer/`.
- CI runs `pnpm run validate:lockfiles` (`scripts/check-no-foreign-lockfiles.mjs`) on every PR and fails the build if any `package-lock.json` or `yarn.lock` is found anywhere in the repo. If you hit this, delete the stray lockfile and re-run `pnpm install` from the repo root.
- CI runs `pnpm syncpack:check` on every PR to enforce consistent dependency version ranges across all workspaces. If this check fails, run `pnpm syncpack:fix` locally to align versions.

The install step runs the repository's `prepare` script, which installs Husky's Git hooks. If hooks are not installed after setup, run:

```bash
pnpm prepare
```

### Start local development

- Use `README.md` and `docs/local-development.md` in this repo for the root development setup.
- The frontend and smart contract repositories each have their own setup instructions once their submodules are initialized.
- Run the root test suite with:

```bash
pnpm test
```

### Local repo basics

- `sdk/` — TypeScript SDK and client helpers
- `cli/` — command-line interface for contract interactions
- `indexer/` — event indexer service for frontend data
- `notifications/` — webhook notification service
- `docs/` — shared documentation and contribution guides

---

## Pre-commit security scan

A Husky `pre-commit` hook runs Gitleaks against the staged changes before every commit. It uses the staged snapshot rather than scanning the entire working tree, keeping the check fast during normal development.

If Gitleaks reports a possible secret:

1. Remove the secret from the staged changes and rotate it if it may already have been exposed.
2. Confirm that the secret is not present in the repository history or any remote.
3. Re-stage the corrected files and commit again.

In a genuine emergency, the hook may be bypassed for one commit with:

```bash
git commit --no-verify
```

Bypassing the hook is an exception, not a way to avoid remediation. Any bypass requires immediate follow-up: remove or rotate the secret, run the Gitleaks scan manually, and notify the maintainers if sensitive data may have been exposed.

Never commit real credentials, private keys, tokens, or other secrets. Use environment variables and local, untracked configuration files for development credentials.

---

## Formatting

This repository uses a shared root Prettier configuration to keep formatting consistent across packages.

- Run `pnpm format` to apply formatting.
- Run `pnpm format:check` to verify formatting without changing files.
- Generated outputs and Markdown files are excluded by the root `.prettierignore`.

## Test conventions

Tests are colocated with source files in `src/` using the repository's existing `*.test.ts` conventions. Run focused tests for the package you change and the complete suite before opening a pull request.

---

## Ways to contribute

Contributions include bug fixes, documentation improvements, tests, SDK and CLI enhancements, indexer and notification changes, frontend work, and smart-contract improvements.

Before starting substantial work:

1. Search open issues for an existing task.
2. Comment on the issue with your proposed approach.
3. Confirm the affected repository and scope.
4. Keep the implementation focused on the agreed issue.

## Applying to work on an issue

Comment on the issue with your proposed approach, relevant experience, and an estimate of the work. Wait for maintainer confirmation before beginning substantial implementation.

## Project board

Use the project board and issue labels to understand priorities, ownership, and current work. Do not begin work on issues marked `in progress` without coordinating with the existing assignee.

## CI/CD pipeline reference

Run the relevant formatting, linting, type-checking, and test commands locally before submitting a pull request. CI is the final validation of the repository and may include additional integration, dependency, lockfile, and deployment checks.

---

## Submitting a pull request

1. Create a focused branch from the current default branch.
2. Make the smallest appropriate change.
3. Add or update tests and documentation as needed.
4. Run formatting, linting, type checks, and relevant tests.
5. Ensure the pre-commit Gitleaks hook passes.
6. Open a pull request with a clear description, testing notes, and any deployment or migration considerations.

Use conventional commit messages, for example:

```text
security: add gitleaks pre-commit hook via Husky
```

Package source changes may require a Changeset. Follow the repository's existing Changeset conventions and CI feedback.

## Branch protection

Keep changes in pull requests until they have been reviewed and all required checks have passed. Do not force-push shared branches or bypass required repository checks.

## Code standards

Follow the existing conventions in the package you are changing. Prefer small, focused changes with tests and clear documentation. Do not commit generated secrets, credentials, private keys, or local configuration files.

## Automated dependency updates

Review automated dependency updates carefully and run the same validation required for a normal pull request. Dependency changes must comply with the pnpm workspace and lockfile policy.

## Code of conduct and security

Contributors are expected to follow `CODE_OF_CONDUCT.md`. For vulnerability reports, follow the instructions in `SECURITY.md` rather than opening a public issue.

## Getting help

Consult the repository README and relevant documentation under `docs/` first. If the answer is not available, open a focused issue or ask in the project's normal maintainer communication channel.
