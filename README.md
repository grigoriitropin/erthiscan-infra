# erthiscan-infra

GitOps repository for managing the workloads and infrastructure components of the Erthiscan Kubernetes cluster.

## Overview

This repository acts as the single source of truth for the cluster state. It utilizes ArgoCD following the "App of Apps" pattern to declaratively deploy and manage infrastructure services and the main application workloads.

## Repository Structure

- `apps/`: ArgoCD `Application` resources. Contains the `root.yaml` for the App of Apps pattern which recursively synchronizes the rest of the cluster applications (Traefik, CloudNativePG, Prometheus, Longhorn, and the Erthiscan backend).
- `bootstrap/`: Configuration and [instructions](bootstrap/README.md) for the initial ArgoCD installation on a fresh cluster.
- `k8s/`: The primary Kubernetes manifests for the Erthiscan application stack, managed via Kustomize. Includes the core application, background workers, scheduled cronjobs, Gateway API routing, and stateful dependencies (PostgreSQL via CloudNativePG).
- `longhorn/`: Storage configuration and Longhorn-specific manifests for persistent data management.

## Deployment Flow

1. **Bootstrap**: ArgoCD is manually installed using the instructions in [bootstrap/README.md](bootstrap/README.md).
2. **Root App**: The `apps/root.yaml` application is applied to the cluster.
3. **Synchronization**: ArgoCD detects the root application and begins synchronizing all other applications defined in the `apps/` directory, which in turn pull their configurations from `k8s/` and other relevant folders.

## Operational Access

The infrastructure provides several web-based management panels. Use `kubectl port-forward` to access them locally.

### Management Panels

| Panel | Namespace | Command | URL |
| :--- | :--- | :--- | :--- |
| **ArgoCD** | `argocd` | `kubectl port-forward svc/argo-cd-argocd-server -n argocd 8080:443` | [https://localhost:8080](https://localhost:8080) |
| **Grafana** | `monitoring` | `kubectl port-forward svc/prometheus-stack-grafana -n monitoring 3000:3000` | [http://localhost:3000](http://localhost:3000) |
| **Longhorn** | `longhorn-system` | `kubectl port-forward svc/longhorn-frontend -n longhorn-system 8081:80` | [http://localhost:8081](http://localhost:8081) |
| **Prometheus** | `monitoring` | `kubectl port-forward svc/prometheus-stack-kube-prom-prometheus -n monitoring 9090:9090` | [http://localhost:9090](http://localhost:9090) |
| **Traefik** | `traefik` | `kubectl port-forward svc/traefik -n traefik 9000:80` | [http://localhost:9000/dashboard/](http://localhost:9000/dashboard/) |

### Retrieving Passwords

#### ArgoCD Admin Password
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

#### Grafana Admin Password
The default password is set to `admin` in `apps/prometheus-stack.yaml`. It is highly recommended to change it via the UI after the first login and store it in a secure password manager (e.g., `pass`).

#### Other Panels (Longhorn, Prometheus, Traefik)
By default, these panels do not have built-in authentication in this setup. They are secured by the fact that they are **not exposed to the internet** and are only accessible via `kubectl port-forward` (which requires valid cluster credentials).

