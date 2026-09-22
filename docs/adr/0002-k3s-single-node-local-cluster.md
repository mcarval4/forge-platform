# ADR-0002: Adopt k3s for the Initial Single-Node Local Kubernetes Cluster

- **Status:** Accepted
- **Date:** 2026-09-22
- **Decision Owners:** Forge Platform Maintainers

## Context

ADR-0001 establishes a local-first, cloud-agnostic foundation for Forge Platform. The next learning milestone requires a Kubernetes environment in which the maintainer can manually install, inspect, operate, troubleshoot, and recover a cluster.

The current host is an Apple Silicon Mac running an Ubuntu 24.04 Server ARM64 virtual machine through Lima. The server VM is intentionally constrained to 2 vCPU, 2 GiB of memory, and a 20 GiB disk. The initial cluster must remain within the project's 6 GiB RAM and 60 GiB provisioned-storage budget.

ADR-0001 named Docker and kind as initial runtime technologies. That choice does not fit the current learning goal or the chosen VM-based environment as well as a lightweight Kubernetes distribution installed directly in the VM. This ADR refines the Kubernetes runtime decision without changing ADR-0001's local-first principle.

## Decision

Forge Platform will use **k3s** as the Kubernetes distribution for the initial local cluster.

The initial topology will be one k3s server running in one Lima-managed Ubuntu 24.04 ARM64 VM. This node provides both the Kubernetes control-plane and workload capacity.

The k3s installation will be performed manually for this milestone. No cloud resources, additional cluster nodes, Argo CD, Ingress customization, Prometheus, or infrastructure-provisioning automation will be introduced as part of this decision.

The initial installation must be investigated through its service status, kubeconfig, node status, system workloads, logs, and basic recovery procedures before any installation automation is added.

## Decision Drivers

### Resource Efficiency

k3s is suitable for the available 2 GiB VM while leaving capacity for a small number of learning workloads.

### Operational Learning

Installing k3s directly in Ubuntu exposes the Kubernetes service, bundled container runtime, kubeconfig, logs, and recovery behavior that this milestone is intended to teach.

### Reproducibility

The VM definition remains version-controlled and disposable. The k3s installation procedure and validation steps will be documented so the cluster can be recreated predictably.

### Local-First and Cloud-Agnostic Design

The cluster runs entirely on the local workstation at zero infrastructure cost and uses standard Kubernetes APIs for future workloads where practical.

## Considered Options

### Option 1 — k3s in a Single Lima VM

**Advantages**

- Lightweight distribution appropriate for the VM resource budget
- Direct exposure to Kubernetes host and service operations
- Supports ARM64 on Ubuntu
- Simple topology for focused troubleshooting and recovery practice
- Avoids a dependency on a hosted cloud service

**Disadvantages**

- One node cannot provide high availability or node-failure tolerance
- Available capacity is limited by the local workstation and VM allocation
- Bundled components differ from a manually assembled upstream Kubernetes installation

### Option 2 — kind on Docker

**Advantages**

- Fast cluster creation and destruction
- Useful for CI and disposable test clusters
- Familiar Docker-based workflow

**Disadvantages**

- Adds an additional containerized layer instead of exercising a Kubernetes server installed in the Ubuntu VM
- Is less aligned with the current objective of inspecting host services and recovery behavior
- Does not match the selected k3s runtime baseline

kind remains a possible future tool for targeted tests, but it is not the initial Forge cluster runtime.

### Option 3 — Multi-Node k3s Cluster

**Advantages**

- Enables node scheduling and failure experiments
- Provides a path toward multi-node operational learning

**Disadvantages**

- Consumes more of the finite memory and storage budget
- Introduces networking, topology, and operational complexity before the single-node fundamentals are understood

This option is deferred until the single-node cluster is documented and operated successfully.

### Option 4 — Managed Kubernetes in a Public Cloud

**Advantages**

- Provides managed control-plane and cloud-integration experience

**Disadvantages**

- Introduces cost, provider coupling, and external dependencies
- Conflicts with the initial local-first, zero-cost learning constraint

## Consequences

### Positive

- The platform has a small, local Kubernetes runtime that can be rebuilt from the Lima VM definition and documented manual steps.
- The maintainer can learn the operational surface of a Kubernetes distribution: `systemd`, kubeconfig access, node readiness, workloads, logs, and recovery.
- The initial footprint leaves room for later, deliberately introduced platform components.

### Negative

- The cluster has no high availability, and loss of the VM makes the cluster unavailable.
- It is suitable only for local learning and portfolio demonstrations; it must not be represented as production-scale infrastructure.
- Some k3s defaults and bundled components must be understood before adding overlapping tools.
- Existing references to Docker and kind as the initial Kubernetes runtime are superseded by this ADR and should be updated when those documents are next revised.

## Implementation Boundaries

- `forge-infrastructure` owns the reproducible Lima VM definition.
- `forge-platform` owns this ADR and cross-repository architectural documentation.
- The initial k3s installation is a manual learning exercise; no provisioning script is authorized by this ADR.
- Kubernetes desired state belongs in `forge-gitops` when GitOps is introduced later. CI must not deploy directly to the cluster.

## Validation

This decision is considered implemented when the following can be demonstrated on the Lima server VM:

1. k3s is installed manually and its service is running.
2. `kubectl` can access the cluster through the k3s kubeconfig.
3. The single node reports `Ready`.
4. System workloads are visible and their logs can be inspected.
5. A documented basic recovery procedure can restore service or explain when rebuilding the disposable VM is the appropriate recovery path.

## Recovery and Reversal

The Lima VM is disposable. If the manual k3s installation becomes unrecoverable, the preferred recovery for this learning stage is to remove and recreate the VM from its version-controlled Lima definition, then repeat the documented installation and validation steps.

Reversing this architectural decision requires a new ADR that supersedes ADR-0002; ADR-0001 remains accepted and continues to govern the local-first foundation.
