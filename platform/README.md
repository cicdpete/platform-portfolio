# platform

Shared **cluster platform** managed by **Argo CD** after bootstrap: app-of-apps root, ingress, certificates, observability, and other operators every workload assumes.

See [ADR 0001](../docs/adr/0001-local-dev-stack.md).

## Bootstrap vs GitOps

| Phase | Owner | Location |
|-------|--------|----------|
| **Install Argo CD once** | Terraform | [`iac/bootstrap/`](../iac/) |
| **Configure Argo CD & platform apps** | GitOps (Argo) | `platform/argocd/`, etc. |

Do not duplicate the Argo Helm release in both Terraform and a raw manifest Argo would reconcile.

## Planned layout

```
platform/
├── README.md
├── argocd/           # AppProject, app-of-apps, root Applications
├── ingress/          # e.g. ingress-nginx (or rely on minikube addon + docs)
├── cert-manager/
└── observability/    # later
```

## Flow

1. Minikube is running; `iac/bootstrap` installed Argo CD.
2. Apply or sync root app-of-apps from `platform/argocd/`.
3. Argo syncs ingress, cert-manager, and other platform apps.
4. Argo syncs `workloads/` applications.

## Runtime

All components run as pods **on Minikube**. Nothing in this folder runs on GitHub Actions except as YAML consumed by Argo.
