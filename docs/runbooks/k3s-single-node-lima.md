# Runbook: Operate a Single-Node k3s Cluster on Lima

- **Status:** Validated locally
- **Scope:** Forge's local single-node learning cluster
- **Related decision:** [ADR-0002](../adr/0002-k3s-single-node-local-cluster.md)

## Purpose

Use this runbook to install, validate, access, diagnose, and perform basic recovery of the k3s cluster that runs inside the Lima Ubuntu ARM64 server VM.

This is a local learning environment. It is not a highly available or production cluster. The Lima VM is disposable; its version-controlled definition is the recovery source of truth.

## Boundaries

- Install k3s manually for this milestone; do not add provisioning scripts.
- Keep the k3s system kubeconfig protected. Do not commit a kubeconfig, token, certificate, or other cluster credential.
- Traefik is disabled because Ingress is not part of the current milestone.
- Do not install Argo CD, Prometheus, or other platform components as part of this runbook.

## Prerequisites

- A running Lima server VM based on `forge-infrastructure/lima/server.yaml`.
- An Ubuntu 24.04 ARM64 VM with 2 vCPU, 2 GiB memory, and a 20 GiB disk.
- `limactl` on the macOS host and `kubectl` on the host when host-side access is required.
- `sudo` access in the VM.

Find the Lima instance name on the macOS host before using commands that reference it:

```bash
limactl list
```

The examples below use `forge-server-01`. Replace it if the local instance has another name.

## Install k3s

Run these commands inside the Lima VM.

```bash
sudo -v
curl -sfL https://get.k3s.io | sh -s - server --disable traefik
```

The official installer installs k3s as a `systemd` service, enables it at boot, installs the `kubectl` command, and writes the system kubeconfig to `/etc/rancher/k3s/k3s.yaml`.

Record the installed version as part of the installation evidence:

```bash
sudo k3s --version
```

## Validate the Cluster

Run inside the VM:

```bash
sudo systemctl status k3s --no-pager
sudo kubectl get nodes -o wide
sudo kubectl get pods -A
sudo kubectl cluster-info
```

Expected state:

- The `k3s` service is `active (running)`.
- The server node is `Ready` with the `control-plane` role.
- CoreDNS, `local-path-provisioner`, and `metrics-server` become `Running`.
- No Traefik workload is present.

Immediately after installation, wait for the system pods if necessary:

```bash
sudo kubectl get pods -n kube-system -w
```

Stop the watch with `Ctrl+C` after every pod is ready.

## Configure Host Access

The system kubeconfig is readable only by root. Copy it through a short-lived file rather than changing its permissions.

### 1. Prepare the temporary copy in the VM

```bash
sudo install -o "$USER" -g "$(id -gn)" -m 600 \
  /etc/rancher/k3s/k3s.yaml /tmp/forge-k3s.yaml
```

### 2. Copy it to macOS

Run on the macOS host:

```bash
mkdir -p ~/.kube
limactl copy forge-server-01:/tmp/forge-k3s.yaml ~/.kube/forge-k3s.yaml
chmod 600 ~/.kube/forge-k3s.yaml
```

Lima's default localhost port forwarding makes the kubeconfig server endpoint (`127.0.0.1:6443`) reachable from the host. Validate it before relying on it:

```bash
KUBECONFIG=~/.kube/forge-k3s.yaml kubectl get nodes
KUBECONFIG=~/.kube/forge-k3s.yaml kubectl get pods -A
KUBECONFIG=~/.kube/forge-k3s.yaml kubectl top nodes
```

For the current terminal session, select the cluster configuration with:

```bash
export KUBECONFIG="$HOME/.kube/forge-k3s.yaml"
```

After successful validation, remove the temporary copy from the VM:

```bash
limactl shell forge-server-01 rm -f /tmp/forge-k3s.yaml
```

## Diagnose the Cluster

Run these commands inside the VM when diagnosing the k3s service:

```bash
sudo systemctl cat k3s
sudo journalctl -u k3s -n 80 --no-pager
sudo kubectl get events -A --sort-by=.lastTimestamp
sudo kubectl get pods -A
```

Useful symptoms and first checks:

| Symptom | First check |
| --- | --- |
| `kubectl` reports permission denied for `/etc/rancher/k3s/k3s.yaml` | Use `sudo kubectl` in the VM or configure the host kubeconfig as described above. |
| Node is not `Ready` | Check `sudo systemctl status k3s --no-pager` and the service journal. |
| A system pod is not ready | Inspect `sudo kubectl describe pod -n kube-system <pod-name>` and recent events. |
| Resource metrics are unavailable | Confirm `metrics-server` is `1/1 Running`, then run `kubectl top nodes`. |

## Basic Service Recovery

Restarting k3s is a controlled local recovery exercise. It briefly makes the single-node cluster unavailable.

```bash
sudo systemctl restart k3s
sudo systemctl is-active k3s
sudo kubectl get nodes -w
```

Wait until the node returns to `Ready`, stop the watch with `Ctrl+C`, and validate the workloads:

```bash
sudo kubectl get pods -A
```

## Workload Smoke Test

Use a temporary deployment to validate scheduling, logs, events, and cleanup. Run these commands from the macOS host after setting `KUBECONFIG`:

```bash
kubectl create deployment hello --image=nginx:alpine
kubectl rollout status deployment/hello --timeout=90s
kubectl get deployment,replicaset,pods
kubectl logs deployment/hello
kubectl describe pod -l app=hello
kubectl get events --sort-by=.lastTimestamp
```

Remove the test workload when finished:

```bash
kubectl delete deployment hello
kubectl get pods
```

## Rebuild Recovery

If the VM or k3s installation cannot be recovered through the service-level checks above, rebuild the disposable VM from the version-controlled Lima definition and repeat this runbook. Do not treat an unrecoverable local VM as a production incident; capture the observed failure and learning outcome before rebuilding.
