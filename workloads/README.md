# workloads

What a **product team** ships: the **Next.js** hello-world MVP, packaging (Helm/Kustomize), and Argo CD `Application` manifests.

Depends on `platform/` for ingress, namespaces, and shared conventions.

See [ADR 0001](../docs/adr/0001-local-dev-stack.md).

## Planned layout

```
workloads/
├── README.md
├── apps/
│   └── hello-next/     # Next.js app + Dockerfile (TBD)
├── charts/             # optional shared Helm library
└── argocd/             # Application manifests per app
```

## Images

| Environment | Image source |
|-------------|----------------|
| **CI** | Build in Actions → push to **public GHCR** `ghcr.io/<owner>/hello-next:<tag>` |
| **Local dev** | `docker build` + `minikube image load` (or `minikube docker-env`) → `hello-next:local` |
| **Deploy** | Manifests reference GHCR tag after CI; local override documented in [local golden path](../docs/runbooks/local-golden-path.md) |

Production analogy: ECR in AWS; this repo uses GHCR locally/CI to avoid cost and LocalStack ECR friction.

## CI pipelines

Pipeline YAML lives at the **repo root** in [`.github/workflows/`](../.github/workflows/), not in `apps/hello-next/.github/`. Workflows use `paths` filters so only relevant changes trigger builds. See [workflows README](../.github/workflows/README.md).

## GitOps

Argo CD (on Minikube) syncs manifests from this repo. CI updates image tags in git; it does not kubectl-apply to your laptop unless a future workflow adds a remote cluster.
