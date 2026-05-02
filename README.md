# erthiscan-infra

GitOps repository for managing the workloads and infrastructure components of the Erthiscan Kubernetes cluster.

## Overview

This repository acts as the single source of truth for the cluster state. It utilizes ArgoCD following the "App of Apps" pattern to declaratively deploy and manage infrastructure services and the main application workloads.

## Repository Structure

- `apps/`: ArgoCD `Application` resources. Contains the `root.yaml` for the App of Apps pattern which recursively synchronizes the rest of the cluster applications (Traefik, CloudNativePG, Prometheus, Longhorn, and the Erthiscan backend).
- `bootstrap/`: Configuration and instructions for the initial ArgoCD installation on a fresh cluster.
- `k8s/`: The primary Kubernetes manifests for the Erthiscan application stack, managed via Kustomize. Includes the core application, background workers, scheduled cronjobs, Gateway API routing, and stateful dependencies (PostgreSQL via CloudNativePG).
- `longhorn/`: Storage configuration and Longhorn-specific manifests for persistent data management.

## Deployment Flow

1. **Bootstrap**: ArgoCD is manually installed using the manifests in `bootstrap/`.
2. **Root App**: The `apps/root.yaml` application is applied to the cluster.
3. **Synchronization**: ArgoCD detects the root application and begins synchronizing all other applications defined in the `apps/` directory, which in turn pull their configurations from `k8s/` and other relevant folders.

