# Forge Platform

> A cloud-native reference platform for building, delivering, securing and operating containerized applications through declarative infrastructure, GitOps, observability, and developer self-service.

## Overview

**Forge Platform** is a production-like engineering environment built around modern **DevOps**, **SRE** and **Platform Engineering** practices.

The platform proveides a reproducible path from source code to a running and observable workload while keeping infrastructure, application delivery, security policies, and operational configuration declarative and version-controlled.

Forge is a intentionally **local-first and cloud-agnostic**. Its core environment can run without paid cloud infrastructure, allowing the platform to be created, destroyed, and rebuild consistently.

Rather than focusing on individual tools, Forge explores the engineering challenges involved in designing and operating a modern software delivery platform.

---

## Goals

Forge is designed around the following capabilities:

- Reproducible infrastructure provisioning
- Automated sofrware delivery
- Declarative Kubernetes deployments
- GitOps-based reconciliation
- Software supply chain security
- Policy enforcement
- Full-stack observability
- Reliability engineering
- Developer self-service
- Automated platform bootstrap and recovery

A core design principle is:

> **The platform should be reproducible from version-controlled configuration with minimal manual intervention.**

---

## Architecture

The high-level delivery flow is:

```text
Developer
    │
    ▼
Git Repository
    │
    ├──── Pull Request
    │
    ▼
Continuous Integration
    │
    ├── Lint
    ├── Tests
    ├── Security Scanning
    ├── Container Build
    └── Artifact Signing
    │
    ▼
Container Registry
    │
    ▼
GitOps Repository
    │
    ▼
Argo CD
    │
    ▼
Kubernetes
    │
    ├── Application Workloads
    ├── Platform Services
    ├── Security Policies
    └── Observability
            │
            ├── Metrics
            ├── Logs
            └── Traces
```

### Platform Layers

```text
┌──────────────────────────────────────────────┐
│             Developer Experience             │
│                                              │
│         Backstage · Service Templates        │
├──────────────────────────────────────────────┤
│              Software Delivery               │
│                                              │
│ GitHub Actions · OCI Registry · Trivy        │
│ Cosign · Argo CD                             │
├──────────────────────────────────────────────┤
│             Application Platform             │
│                                              │
│ Kubernetes · Helm · Ingress · Policies       │
├──────────────────────────────────────────────┤
│                Observability                 │
│                                              │
│ Prometheus · Grafana · Loki · Tempo          │
│ OpenTelemetry                                │
├──────────────────────────────────────────────┤
│                Infrastructure                │
│                                              │
│        OpenTofu · Docker · kind              │
└──────────────────────────────────────────────┘
```

Detailed architecture documentation and engineering decisions are maintained under [`docs/`](./docs).

---

## Technology Stck

| Area | Technologies |
|---|---|
| Infrastructure as Code | OpenTofu |
| Containers | Docker |
| Kubernetes | kind, Kubernetes |
| Packaging | Helm |
| Continuous Integration | GitHub Actions |
| Container Registry | GitHub Container Registry |
| GitOps | Argo CD |
| Security | Trivy, Cosign, Kyverno |
| Metrics | Prometheus |
| Visualization | Grafana |
| Logs | Loki |
| Tracing | Tempo |
| Telemetry | OpenTelemetry |
| Developer Portal | Backstage |

---

## Platform Components

### Infrastructure

Forge infrastructure is designed to be **disposable and reproducible**.

The initial implementation uses a local Kubernetes environment, avoiding dependency on a specific cloud provider while preserving architectural concepts that can later be mapped to AWS, Azure or GCP.

Infrastructure responsibilities include:

- Cluster provisioning
- Network configuration
- Platform bootstrap
- Environment configuration
- Infrastructure validation
- Lifecycle automation

**Core technologies:**

- OpenTofu
- Docker
- kind
- Kubernetes

---

### Continuous Integration

Every change must pass automated validation before becoming a deployable artifact.

```text
Source
  │
  ├── Static Analysis
  ├── Automated Tests
  ├── Dependency Scan
  ├── Container Build
  ├── Vulnerability Scan
  └── Artifact Signing
        │
        ▼
   OCI Registry
```

The objective is to produce artifacts associated with a specific source revision and CI execution.

**Core technologies:**

- GitHub Actions
- Docker
- Trivy
- Cosign
- GitHub Container Registry

---

### Kubernetes

Kubernetes provides the runtime environment for platform and application workloads.

The platform progressively implements:

- Deployments and Services
- Ingress
- ConfigMaps and Secrets
- Resource requests and limits
- Liveness and readiness probes
- Horizontal scaling
- RBAC
- Network Policies
- Pod security controls

Runtime configuration should remain **declarative and version-controlled**.

---

### GitOps

Forge follows a **GitOps delivery model**.

Git represents the desired state of each environment, while Argo CD continuously reconciles that state with Kubernetes.

```text
Git
 │
 │ Desired State
 ▼
Argo CD
 │
 │ Reconciliation
 ▼
Kubernetes
```

This provides:

- Declarative deployments
- Version-controlled configuration
- Auditable changes
- Drift detection
- Reproducible rollbacks

---

### Supply Chain Security

Security controls are integrated into the software delivery lifecycle.

Forge progressively introduces:

- Dependency scanning
- Container vulnerability scanning
- Artifact signing
- Artifact verification
- Kubernetes policy enforcement
- Least-privilege access
- Secret management practices

**Core technologies:**

- Trivy
- Cosign
- Kyverno
- Kubernetes RBAC

Security controls should fail safely and be validated through automated tests whenever possible.

---

### Observability

Applications and platform components expose telemetry through standard observability signals.

```text
Application
     │
     ▼
OpenTelemetry
     │
     ├──── Metrics ───► Prometheus ──┐
     │                               │
     ├──── Logs ──────► Loki ────────┼──► Grafana
     │                               │
     └──── Traces ────► Tempo ───────┘
```

The observability stack supports investigation of:

- Availability
- Request latency
- Error rates
- Resource utilization
- Application behavior
- Deployment impact
- Service dependencies

Dashboards, alerts, and telemetry configuration should be maintained as code whenever possible.

---

### Reliability Engineering

Forge includes controlled failure scenarios to validate platform behavior rather than assuming reliability from successful deployments.

Experiments include:

- Pod termination
- Failed deployments
- Health-check failures
- Resource exhaustion
- Configuration drift
- Dependency failures

The operational lifecycle follows:

```text
Failure
   │
   ▼
Detection
   │
   ▼
Alert
   │
   ▼
Investigation
   │
   ▼
Mitigation
   │
   ▼
Recovery
   │
   ▼
Postmortem
```

**SLIs**, **SLOs**, runbooks, and postmortems are maintained alongside the platform documentation.

---

### Developer Experience

The final platform layer introduces developer self-service through **Backstage**.

A developer should eventually be able to request a standardized service through a predefined template:

```text
Developer
    │
    ▼
Backstage
    │
    ▼
Software Template
    │
    ├── Repository
    ├── Application Skeleton
    ├── CI Pipeline
    ├── Container Configuration
    ├── Kubernetes Configuration
    ├── GitOps Registration
    └── Observability Defaults
            │
            ▼
       Running Service
```

The objective is to provide a standardized **golden path** while keeping the underlying platform extensible.

---

## Repository Model

Forge is split into repositories according to engineering responsibility:

```text
forge-platform
│
├── forge-infrastructure
├── forge-services
├── forge-gitops
├── forge-observability
└── forge-developer-platform
```

### `forge-platform`

Architecture, engineering decisions, documentation, runbooks, and project-level integration.

### `forge-infrastructure`

Infrastructure provisioning, local Kubernetes bootstrap, OpenTofu modules, and infrastructure validation.

### `forge-services`

Reference workloads used to exercise the delivery platform.

### `forge-gitops`

Desired Kubernetes state, Helm configuration, environment definitions, and Argo CD resources.

### `forge-observability`

Metrics, logs, traces, dashboards, alerts, and SRE configuration.

### `forge-developer-platform`

Backstage configuration, software catalog, templates, and developer self-service workflows.

---

## Engineering Principles

### Everything as Code

Infrastructure, deployments, policies, dashboards, and operational configuration should be version-controlled whenever practical.

### Declarative over Imperative

Desired state should be described rather than maintained through manual procedures.

### Git as Source of Truth

Production-like runtime changes should originate from version-controlled configuration.

### Security by Default

Security controls belong inside the delivery process rather than being added after deployment.

### Observability by Default

Services should expose useful telemetry from the moment they are deployed.

### Automate Repetitive Operations

Common provisioning, deployment, validation, and recovery operations should be automated.

### Design for Failure

Failure scenarios should be deliberately tested and documented.

### Reproducibility

A clean environment should be able to reconstruct the platform from its repositories.

---

## Documentation

Engineering documentation is maintained under:

```text
docs/
├── architecture/
│   ├── overview.md
│   └── diagrams/
│
├── adr/
│   ├── 0001-local-first.md
│   ├── 0002-kubernetes-runtime.md
│   └── 0003-gitops-delivery.md
│
├── runbooks/
│   ├── cluster-recovery.md
│   └── failed-deployment.md
│
├── postmortems/
│
└── sre/
    ├── sli.md
    └── slo.md
```

Architectural decisions are documented using **Architecture Decision Records (ADRs)** so that significant technical choices include their context, alternatives, consequences, and trade-offs.

---

## Project Status

> 🚧 **Forge Platform is under active development.**

| Capability | Status |
|---|---|
| Architecture | 🚧 In progress |
| Local Infrastructure | ⏳ Planned |
| CI Pipeline | ⏳ Planned |
| Kubernetes Runtime | ⏳ Planned |
| GitOps | ⏳ Planned |
| Supply Chain Security | ⏳ Planned |
| Observability | ⏳ Planned |
| SRE Practices | ⏳ Planned |
| Developer Platform | ⏳ Planned |

Status indicators represent implementation progress, not production-readiness guarantees.

---

## Roadmap

Development is organized incrementally:

```text
Foundation
    │
    ▼
Infrastructure
    │
    ▼
CI & Supply Chain
    │
    ▼
Kubernetes
    │
    ▼
GitOps
    │
    ▼
Observability
    │
    ▼
Reliability Engineering
    │
    ▼
Developer Platform
```

Each stage should leave the platform in a **demonstrable and reproducible state**.

---

## Current Focus

### Foundation

The current milestone establishes the platform foundation.

- [ ] Define repository conventions
- [ ] Document the initial architecture
- [ ] Establish Architecture Decision Records
- [ ] Bootstrap the local development environment
- [ ] Provision the first Kubernetes cluster
- [ ] Add automated infrastructure validation

---

## Non-Goals

At this stage, Forge does **not** attempt to:

- Replace a production cloud platform
- Emulate every managed cloud service locally
- Provide production-grade multi-tenancy
- Guarantee high availability on a single development machine
- Abstract Kubernetes entirely from platform engineers

These constraints are intentional and keep the project focused on reproducible engineering practices without requiring paid infrastructure.

---

## Contributing

Changes should follow the repository engineering workflow:

```text
Issue
  │
  ▼
Feature Branch
  │
  ▼
Commit
  │
  ▼
Pull Request
  │
  ▼
Automated Validation
  │
  ▼
Review
  │
  ▼
Merge
```

### Commit Convention

The project follows **Conventional Commits**.

Examples:

```text
feat: add local cluster bootstrap
fix: correct ingress configuration
docs: document gitops architecture
ci: add infrastructure validation
test: add cluster smoke tests
refactor: reorganize tofu modules
chore: update dependencies
```

---

## License

This project is licensed under the terms defined in [`LICENSE`](./LICENSE).