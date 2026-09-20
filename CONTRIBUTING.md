# Contributing to Forge Platform

Forge Platform follows a lightweight engineering workflow designed to keep changes traceable, reviewable, and reproducible.

## Development Workflow

Changes should normally follow this lifecycle:

```text
Issue
  │
  ▼
Branch
  │
  ▼
Implementation
  │
  ▼
Local Validation
  │
  ▼
Pull Request
  │
  ▼
Automated Checks
  │
  ▼
Review
  │
  ▼
Merge
```

Direct changes to the `main` branch should be avoided.

## Issues

Use an issue when introducing:

- New platform capabilities
- Infrastructure changes
- Architectural changes
- Bugs
- Operational improvements
- Security improvements
- Significant documentation changes

Small corrections such as typos do not require an issue.

## Branch Naming

Use short-lived branches with descriptive names.

Format:

```text
<type>/<description>
```

Examples:

```text
feat/kind-bootstrap
feat/argocd-installation
fix/cluster-networking
docs/gitops-architecture
refactor/tofu-modules
ci/infrastructure-validation
```

Recommended prefixes:

| Prefix | Purpose |
|---|---|
| `feat/` | New capability |
| `fix/` | Bug fix |
| `docs/` | Documentation |
| `refactor/` | Internal restructuring |
| `ci/` | CI/CD changes |
| `test/` | Test changes |
| `chore/` | Maintenance |

## Commit Messages

Forge follows the Conventional Commits convention.

Format:

```text
<type>: <description>
```

Examples:

```text
feat: add kind cluster bootstrap
fix: correct ingress configuration
docs: document local-first architecture
ci: add infrastructure validation
test: add cluster smoke tests
refactor: reorganize tofu modules
chore: update dependencies
```

Keep commits focused on a single logical change whenever practical.

## Pull Requests

Pull requests should explain:

- What changed
- Why the change is necessary
- How the change was validated
- Whether architecture or operational behavior changed

Large changes should be divided into smaller reviewable units whenever possible.

## Validation

Before opening a pull request:

- Run relevant tests
- Run formatting and linting
- Verify documentation links
- Confirm no credentials or sensitive data were committed
- Confirm the change can be reproduced from documented instructions

Additional automated validation will be introduced as the platform evolves.

## Architecture Decisions

Changes affecting significant architectural decisions should be documented through an Architecture Decision Record (ADR).

Examples include:

- Runtime changes
- GitOps strategy
- Repository boundaries
- Security architecture
- Observability architecture
- Cloud provider adoption
- Major technology replacements

ADRs are stored under:

```text
docs/adr/
```

Existing accepted ADRs should normally remain immutable.

If a decision changes, create a new ADR that supersedes the previous decision.

## Documentation

Documentation should describe the