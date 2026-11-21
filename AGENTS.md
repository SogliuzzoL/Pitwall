# Repository conventions

## Branching model: GitFlow

This repository uses GitFlow.

| Branch | Purpose | Lifetime |
|---|---|---|
| `main` | Released, deployable state. Every commit on it is a release or a hotfix. | permanent |
| `develop` | Integration. Every feature lands here first. | permanent |
| `feature/<slug>` | One unit of work. Branches from `develop`, merges back into `develop`. | short-lived |
| `release/<version>` | Stabilisation before a release. Branches from `develop`, merges into `main` **and** back into `develop`. | short-lived |
| `hotfix/<slug>` | Urgent fix on a released version. Branches from `main`, merges into `main` **and** `develop`. | short-lived |

Rules:

- Never commit directly to `main` or `develop`.
- Keep `feature/*` branches short. A branch alive for more than a few days is a branch that is too big.
- Merge with `--no-ff` so the branch topology stays visible in the history.
- Tag releases on `main` with SemVer: `vMAJOR.MINOR.PATCH`.
- `main` is the only branch that is ever deployed.
- The `git flow` CLI is optional; plain `git switch -c feature/<slug> develop` is enough.

### Versioning axes

Three versions move independently and must never be conflated:

- **code version** — the SemVer tag on `main`;
- **data version** — a DVC revision;
- **model version** — a registry entry in MLflow.

Every prediction records all three.

## Commits

[Conventional Commits](https://www.conventionalcommits.org/): `type(scope): summary`, in the imperative mood.

Allowed types: `feat`, `fix`, `refactor`, `test`, `docs`, `build`, `ci`, `chore`, `perf`.

A commit does one thing. If the subject line needs an "and", split it.

## Quality gate

Before merging into `develop`, `make check` must pass: lint, type check, tests.

## Code conventions

- Code, identifiers, comments and documentation are written in English.
- No hardcoded absolute paths and no credentials. Configuration comes from typed settings backed by environment variables.
- Business logic lives in `src/pitwall/`. Notebooks are for exploration only and must never be imported by the package.
- Every transformation that filters, normalises or aggregates race data has a test.
