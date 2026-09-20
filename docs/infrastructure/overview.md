# Forge Platform Architecture

## 1. Purpose

Forge Platform is a local-first, cloud-agnostic reference platform designed to explore the engineering practices required to build, deliver, secure, observe, and operate containerized applications.

The platform combines infrastructure automation, Kubernetes, GitOps, software supply chain security, observability, reliability engineering, and developer self-service into a single engineering environment.

The architecture prioritizes:

- Reproducibility
- Automation
- Declarative configuration
- Security by default
- Observability by default
- Failure recovery
- Developer experience
- Portability

---

## 2. Architecture Principles

### Local First

The core platform must be executable without requiring paid cloud infrastructure.

Local execution provides a deterministic environment for development, testing, failure simulation, and platform experimentation.

Cloud-specific implementations may be introduced later without changing the fundamental delivery model.

### Everything as Code

Infrastructure, deployments, policies, observability configuration, and operational procedures should be version-controlled whenever practical.

### Declarative Configuration

Desired state should be declared rather than maintained through manual operations.

### Git as Source of Truth

Runtime configuration should originate from version-controlled repositories.

Direct modifications to managed environments should be treated as configuration drift.

### Immutable Artifacts

Applications should produce versioned container artifacts that remain unchanged between environments.

### Security by Default

Security validation should be integrated throughout the delivery lifecycle.

### Observability by Default

Applications deployed through Forge should expose sufficient telemetry for their behavior to be understood during normal operation and failure scenarios.

### Design for Failure

Platform behavior must be validated under controlled failure conditions.

---

## 3. System Context

Forge sits between application developers and the runtime environment.

```text
┌─────────────────┐
│    Developer    │
└────────┬────────┘
         │
         │ Git
         ▼
┌─────────────────┐
│     GitHub      │
└────────┬────────┘
         │
         │ CI
         ▼
┌─────────────────┐
│ Artifact Build  │
│ Scan & Signing  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  OCI Registry   │
└────────┬────────┘
         │
         │ Artifact Reference
         ▼
┌─────────────────┐
│ GitOps Desired  │
│      State      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│     Argo CD     │
└────────┬────────┘
         │
         │ Reconciliation
         ▼
┌─────────────────┐
│   Kubernetes    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Observability   │
└─────────────────┘
```

---

## 4. Platform Layers

### 4.1 Infrastructure Layer

Responsible for providing the runtime foundation.

Initial implementation:

- Docker
- kind
- OpenTofu
- Kubernetes

Responsibilities:

- Cluster provisioning
- Network configuration
- Environment lifecycle
- Platform bootstrap
- Infrastructure validation

The local environment must be disposable.

Deleting the cluster should not destroy the platform definition.

---

## 5. Software Delivery Layer

Continuous Integration validates source changes and creates immutable application artifacts.

```text
Pull Request
     │
     ▼
Static Analysis
     │
     ▼
Tests
     │
     ▼
Dependency Analysis
     │
     ▼
Container Build
     │
     ▼
Vulnerability Scan
     │
     ▼
Artifact Signing
     │
     ▼
OCI Registry
```

Application artifacts must be traceable to their source revision.

CI is responsible for **building and validating artifacts**.

CI is not responsible for directly modifying the Kubernetes runtime.

---

## 6. GitOps Layer

Continuous Delivery is handled through GitOps.

```text
Application Repository
        │
        │ produces artifact
        ▼
   OCI Registry

GitOps Repository
        │
        │ desired state
        ▼
      Argo CD
        │
        │ reconciliation
        ▼
    Kubernetes
```

Argo CD reconciles Kubernetes with the desired state stored in Git.

This separation prevents CI pipelines from requiring broad write access to the Kubernetes API.

---

## 7. Runtime Layer

Kubernetes provides the application runtime.

Workloads should define:

- Resource requests
- Resource limits
- Liveness probes
- Readiness probes
- Security context
- Configuration
- Network policies where applicable

Application packaging will use Helm.

---

## 8. Security Architecture

Security controls exist across multiple stages.

### Source

- Dependency analysis
- Secret detection
- Static analysis

### Build

- Container vulnerability scanning
- Minimal base images
- Reproducible builds where practical

### Artifact

- Immutable tags or digests
- Artifact signing
- Provenance

### Runtime

- RBAC
- Network Policies
- Pod security controls
- Admission policies

Kyverno will provide policy enforcement within Kubernetes.

---

## 9. Observability Architecture

Forge uses the three primary telemetry signals:

- Metrics
- Logs
- Traces

Applications will be instrumented through OpenTelemetry where appropriate.

```text
                 ┌──► Prometheus ──┐
                 │                 │
Application ─► OTel ──► Loki ─────┼──► Grafana
                 │                 │
                 └──► Tempo ──────┘
```

Observability configuration should be version-controlled.

---

## 10. Reliability Engineering

The platform will define measurable service reliability through:

- Service Level Indicators
- Service Level Objectives
- Alerting
- Runbooks
- Failure experiments
- Postmortems

Failure scenarios will include:

- Pod termination
- Broken deployments
- Failed health checks
- Resource pressure
- Dependency failures
- Configuration drift

The objective is not only to detect failures but also to understand and recover from them.

---

## 11. Developer Experience

Backstage will provide the developer-facing entry point into Forge.

Developers should eventually be able to create standardized services through Software Templates.

A service template may generate:

```text
Service
├── Application skeleton
├── Dockerfile
├── Tests
├── CI workflow
├── Helm chart
├── GitOps configuration
├── Observability configuration
└── Documentation
```

This creates a standardized golden path while allowing advanced teams to customize lower-level configuration when necessary.

---

## 12. Repository Boundaries

Forge separates responsibilities between repositories.

### forge-platform

Owns:

- Architecture
- ADRs
- Engineering standards
- Runbooks
- SRE documentation
- Cross-platform documentation

### forge-infrastructure

Owns:

- OpenTofu
- kind
- Cluster lifecycle
- Infrastructure bootstrap

### forge-services

Owns:

- Reference applications
- Application tests
- Docker builds
- Application CI

### forge-gitops

Owns:

- Helm values
- Environment configuration
- Argo CD applications
- Desired runtime state

### forge-observability

Owns:

- Prometheus configuration
- Grafana dashboards
- Loki
- Tempo
- Alerting rules

### forge-developer-platform

Owns:

- Backstage
- Software Catalog
- Software Templates
- Developer workflows

---

## 13. Deployment Flow

A standard application change follows:

```text
Developer
    │
    ▼
Pull Request
    │
    ▼
CI Validation
    │
    ▼
Merge
    │
    ▼
Container Build
    │
    ▼
Security Scan
    │
    ▼
Artifact Signing
    │
    ▼
OCI Registry
    │
    ▼
GitOps Update
    │
    ▼
Argo CD Reconciliation
    │
    ▼
Kubernetes Deployment
    │
    ▼
Health Validation
    │
    ▼
Observability
```

---

## 14. Environment Strategy

The initial environment model is:

```text
local
```

Additional environments may later include:

```text
development
staging
production
```

The architecture should avoid assumptions that prevent future migration to a public cloud provider.

---

## 15. Constraints

The initial platform operates under the following constraints:

- No paid cloud infrastructure
- Single-machine execution
- Limited compute and memory
- No production-grade high availability
- No requirement for multi-region operation
- No production-grade multi-tenancy

These constraints are intentional.

They allow the project to focus on engineering principles without introducing unnecessary infrastructure cost.

---

## 16. Architecture Decisions

Significant technical decisions are documented separately using Architecture Decision Records.

Initial ADRs:

| ADR | Decision |
|---|---|
| ADR-0001 | Adopt a local-first architecture |
| ADR-0002 | Use Kubernetes as the application runtime |
| ADR-0003 | Use GitOps for continuous delivery |
| ADR-0004 | Separate application and desired-state repositories |
| ADR-0005 | Adopt OpenTelemetry for application telemetry |

Each ADR records the context, considered alternatives, decision, and consequences.

---

## 17. Evolution

The architecture is expected to evolve.

Changes that significantly affect platform boundaries, security, deployment strategy, runtime behavior, or operational characteristics should be captured through new ADRs rather than silently modifying previous decisions.