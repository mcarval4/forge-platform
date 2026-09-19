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