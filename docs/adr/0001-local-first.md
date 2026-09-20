# ADR-0001: Adopt a Local-First Platform Architecture

- **Status:** Accepted
- **Date:** 2026-09-20
- **Decision Owners:** Forge Platform Maintainers

## Context

Forge Platform requires an execution environment for developing and validating infrastructure automation, Kubernetes workloads, GitOps workflows, security controls, observability, and reliability practices.

A public cloud provider could provide managed infrastructure for these capabilities. However, making a cloud provider mandatory at the beginning of the project would introduce several constraints:

- Infrastructure cost
- Dependence on free-tier limits or temporary credits
- Provider-specific configuration
- Longer infrastructure feedback loops
- External dependencies during development
- Increased cleanup requirements
- Risk of unexpected resource consumption

The initial platform should be reproducible without requiring paid infrastructure while preserving architectural concepts that can later be implemented in a public cloud.

The platform therefore needs an execution model that supports advanced engineering experiments without coupling the core architecture to AWS, Azure, or Google Cloud.

## Decision

Forge Platform will adopt a **local-first architecture** for its initial implementation.

The primary runtime will consist of:

- Docker as the local container runtime
- kind for Kubernetes clusters
- Kubernetes as the workload orchestration layer
- OpenTofu for infrastructure automation where appropriate

The local environment will be treated as a real implementation target rather than an attempt to emulate every service offered by a public cloud provider.

Cloud-specific implementations may be introduced later as separate infrastructure targets.

The core platform architecture should remain portable enough that major capabilities can be mapped to cloud services without redesigning the entire software delivery model.

## Decision Drivers

The primary drivers for this decision are:

### Cost

The complete core platform must be usable without paid cloud infrastructure.

### Reproducibility

A developer should be able to destroy and recreate the local platform from version-controlled configuration.

### Fast Feedback

Infrastructure and platform changes should be testable without waiting for remote infrastructure provisioning whenever local execution is sufficient.

### Portability

Core platform concepts should not depend on a single cloud provider.

### Experimentation

The environment must support controlled experiments involving:

- Kubernetes lifecycle
- GitOps reconciliation
- Policy enforcement
- Observability
- Failure scenarios
- Recovery procedures

### Automation

Manual environment configuration should be minimized.

## Considered Options

### Option 1 — Local-First Kubernetes

Run the initial platform locally using Docker and kind.

**Advantages**

- No infrastructure cost
- Fast provisioning and destruction
- Easy experimentation
- Suitable for automated tests
- Works without a cloud account
- Reduces provider coupling

**Disadvantages**

- Does not reproduce managed cloud services
- Limited by workstation resources
- Does not represent production networking exactly
- Cannot demonstrate every cloud-specific operational concern
- High availability testing is limited by the physical host

### Option 2 — AWS-First

Build the initial platform directly on AWS using services such as EKS and related infrastructure.

**Advantages**

- Experience with real cloud infrastructure
- Access to managed services
- More realistic networking and IAM scenarios
- Better representation of distributed infrastructure

**Disadvantages**

- Ongoing or potentially unexpected cost
- AWS-specific architecture decisions introduced early
- Slower infrastructure feedback cycles
- Additional cleanup and cost-control requirements
- Less accessible as a reproducible zero-cost environment

### Option 3 — Azure-First

Build the platform around Azure and AKS.

This provides many of the same benefits as the AWS-first approach but introduces similar cost and provider-coupling concerns.

### Option 4 — Google Cloud-First

Build the platform around Google Cloud and GKE.

Again, this provides access to managed cloud infrastructure but introduces cost, external dependencies, and provider-specific decisions before they are required.

### Option 5 — Local Cloud Emulation

Use local tools to imitate cloud APIs and managed services.

**Advantages**

- Can provide compatibility with selected cloud APIs
- Useful for testing some provider-specific integrations

**Disadvantages**

- Adds another abstraction layer
- Emulated behavior may differ from real provider behavior
- Increases platform complexity
- Encourages assumptions that local emulation is equivalent to a real cloud environment

This approach will not be the default architecture.

## Consequences

### Positive

The platform can be developed and operated without cloud infrastructure costs.

Cluster lifecycle becomes fast enough to be part of normal development workflows.

Failure scenarios can be executed repeatedly without affecting shared infrastructure.

The environment becomes easier to reproduce on another compatible workstation.

The architecture remains independent from a cloud provider during its initial development.

### Negative

Some infrastructure concerns cannot be realistically validated locally.

Examples include:

- Cloud IAM integration
- Managed load balancers
- Managed Kubernetes control planes
- Cloud networking
- Availability zones
- Managed databases
- Cloud-native secret management
- Cloud billing and quota behavior

These capabilities must be validated separately if Forge later introduces a public cloud implementation.

## Architectural Implications

The local-first decision affects several platform boundaries.

### Kubernetes

Applications should target standard Kubernetes APIs whenever practical.

Provider-specific Kubernetes resources should be isolated from portable application configuration.

### Infrastructure

Infrastructure code should distinguish between:

```text
platform concepts
```

and:

```text
provider implementations
```

A future structure may evolve toward:

```text
infrastructure/
├── local/
├── aws/
├── azure/
└── gcp/
```

The existence of these directories is not required until additional providers are actually implemented.

### GitOps

The GitOps model should remain largely independent of the underlying infrastructure provider.

Argo CD should reconcile desired Kubernetes state regardless of whether the cluster runs locally or in a public cloud.

### Observability

Application telemetry should use portable standards such as OpenTelemetry where appropriate.

### Application Delivery

CI pipelines should build artifacts without requiring direct access to the Kubernetes cluster.

Continuous delivery should remain separated from artifact creation.

## Validation

This decision will be considered successfully implemented when the following workflow can be completed from a supported clean development environment:

```text
Clone repositories
       │
       ▼
Install documented prerequisites
       │
       ▼
Run platform bootstrap
       │
       ▼
Create local Kubernetes cluster
       │
       ▼
Install core platform components
       │
       ▼
Deploy reference workload
       │
       ▼
Validate workload health
       │
       ▼
Destroy environment
       │
       ▼
Recreate environment
```

The recreated environment should not depend on undocumented manual configuration.

## Future Evolution

Local-first does not mean local-only.

A public cloud implementation may be introduced when there is a specific engineering capability that cannot be meaningfully validated in the local environment.

Examples include:

- Cloud IAM integration
- Managed Kubernetes
- Multi-zone architecture
- Cloud load balancing
- Managed databases
- Cloud-native secret management
- Autoscaling infrastructure
- Cloud networking

When a provider is introduced, the decision should be documented through a separate ADR describing:

1. Why the provider is required
2. Which capabilities require it
3. Which components remain provider-independent
4. Cost implications
5. Portability implications

## Review Conditions

This decision should be revisited if:

- Local resource limitations prevent meaningful platform testing
- A cloud-specific capability becomes a core platform requirement
- The platform requires distributed infrastructure that cannot be represented locally
- A reliable zero-cost or controlled-cost cloud environment becomes available and provides significant engineering value

## Related Decisions

Future related ADRs:

- ADR-0002 — Kubernetes as the Application Runtime
- ADR-0003 — GitOps for Continuous Delivery
- ADR-0004 — Separation of Application and Desired State
- ADR-0005 — OpenTelemetry for Application Telemetry