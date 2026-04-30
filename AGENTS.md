# simplest-k8s — Agent Context

Minimal Kubernetes tutorial project deploying a Kaboom.js space shooter game to a local Rancher Desktop cluster. Educational reference for https://itnext.io/simplest-minimal-k8s-app-tutorial-with-rancher-desktop-in-5-min-5481edb9a4a5.

## Prerequisites

- [Rancher Desktop](https://rancherdesktop.io/) with:
  - `kubectl` enabled (Kubernetes distribution: k3s)
  - Traefik ingress controller enabled (default in Rancher Desktop)
- No build step — the container image is pre-built and hosted on ghcr.io

## Deploy & Teardown

```bash
# Deploy all manifests
kubectl apply -f .

# Remove all resources
kubectl delete -f .
```

## Manifest Overview

| File | Kind | Purpose |
|---|---|---|
| `jade-shooter-deployment.yaml` | Deployment | Runs the game container (1 replica, port 8080) |
| `jade-shooter-service.yaml` | Service | ClusterIP exposing port 8080 inside the cluster |
| `jade-shooter-ingress.yaml` | Ingress | Routes `jade-shooter.rancher.localhost` → service:8080 |

## Key Values

| Setting | Value |
|---|---|
| Container image | `ghcr.io/jwsy/jade-shooter-22:v2.0.3` |
| Container port | `8080` |
| Service port | `8080` → targetPort `8080` |
| Ingress hostname | `jade-shooter.rancher.localhost` |
| Memory limit/request | `128Mi` |
| CPU limit/request | `200m` |

## Editing Conventions

- **Image version bump**: update `image:` in `jade-shooter-deployment.yaml`. The tag format is `vX.Y.Z`.
- **Port changes**: keep the port consistent across all three manifests — `containerPort` in the Deployment, `port`/`targetPort` in the Service, and the backend `port.number` in the Ingress.
- **Replicas**: default is `1`; scale up in the Deployment `spec.replicas` field.
- **Labels**: the selector label is `app: jade-shooter` — it must match across the Deployment `spec.selector.matchLabels`, the pod template `metadata.labels`, and the Service `spec.selector`.

## Branches

| Branch | Description |
|---|---|
| `main` | Canonical minimal example (3 plain YAML manifests) |
| `helm` / `helm4` | Helm chart packaging of the same app |
| `kyaml` | KYAML-based variant |
| `tailscale-k8s-operator` | Tailscale ingress operator instead of Traefik |
| `traefik-ingress-route` | Traefik IngressRoute CRD instead of standard Ingress |
| `mount-local` | Local filesystem mount variant |

Do not merge feature/tutorial branches into `main` — each branch is a standalone tutorial variant.

## Verification

After applying manifests, confirm the app is running:

```bash
# Check pod is Running
kubectl get pods -l app=jade-shooter

# Check service exists
kubectl get svc jade-shooter-service

# Check ingress has an address
kubectl get ingress jade-shooter-ingress

# Open in browser (Rancher Desktop manages TLS automatically)
open https://jade-shooter.rancher.localhost
```

Expected: pod in `Running` state, ingress shows an address, game loads in browser.
