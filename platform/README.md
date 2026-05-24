# platform

Shared **cluster platform** managed by **Argo CD** after bootstrap: **separate app-of-apps roots** for platform vs workloads, with **AppProjects** that restrict resource kinds and reduce cascade-delete blast radius (see [ADR 0002](../docs/adr/0002-argocd-roots-and-guardrails.md)).

Ingress, certificates, observability, and other operators every workload assumes live here—not product Deployments.

See also [ADR 0001](../docs/adr/0001-local-dev-stack.md).

## Why not one `gitops-root`?

A single root Application is easy to operate but dangerous: **Cascade** delete in the Argo UI can remove every child app and the resources they own (including storage/PVCs if they were ever hung under that tree). This repo uses **sibling roots** (`platform-root`, `workloads-root`) plus project blacklists so platform and app teams do not share one deletable tree.

## Bootstrap vs GitOps

| Phase | Owner | Location |
|-------|--------|----------|
| **Install Argo CD once** | Terraform | [`iac/bootstrap/`](../iac/) |
| **Projects, roots, platform apps** | GitOps (Argo) | [`platform/argocd/`](argocd/) |
| **Workload apps** | GitOps (Argo) | [`workloads/argocd/`](../workloads/argocd/) (synced via `workloads-root`) |

Do not duplicate the Argo Helm release in both Terraform and a manifest Argo reconciles.

## Planned layout

```
platform/
├── README.md
├── argocd/
│   ├── projects/       # platform, workloads AppProjects
│   ├── roots/          # platform-root, workloads-root (sibling Applications)
│   └── apps/           # ingress, cert-manager, observability, …
├── ingress/
├── cert-manager/
└── observability/      # later
```

Details: [argocd/README.md](argocd/README.md).

## Flow

1. Minikube is running; `iac/bootstrap` installed Argo CD.
2. Apply **AppProjects**, then sync **`platform-root`** and **`workloads-root`** (siblings).
3. `platform-root` syncs ingress, cert-manager, and other platform child apps.
4. `workloads-root` syncs `workloads/argocd/` (e.g. hello-next).

## Guardrails (short)

| Mechanism | Purpose |
|-----------|---------|
| **Two roots** | Cascade on `platform-root` does not delete `workloads-root` (and vice versa). |
| **AppProject denylists** | Block PV/PVC, storage operators, and wrong namespaces per project. |
| **Prune policy** | Conservative defaults on platform apps until explicitly enabled. |
| **No storage under these roots** | Longhorn/CSI-style operators belong on a dedicated `storage-root` in real clusters—not in this MVP. |

## Runtime

All components run as pods **on Minikube**. Nothing in this folder runs on GitHub Actions except YAML consumed by Argo.
