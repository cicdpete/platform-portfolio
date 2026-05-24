# platform/argocd

Argo CD **AppProjects**, **root Applications**, and platform child apps. See [ADR 0002](../../docs/adr/0002-argocd-roots-and-guardrails.md).

## Layout (planned)

```
platform/argocd/
├── README.md
├── projects/
│   ├── platform.yaml      # AppProject: platform namespaces, ingress, certs; deny PV/PVC/storage ops
│   └── workloads.yaml     # AppProject: app namespaces; deny cluster infra & argocd ns
├── roots/
│   ├── platform-root.yaml # Application → platform/argocd/apps/* (or charts)
│   └── workloads-root.yaml # Application → ../../workloads/argocd/*
└── apps/                  # Child Applications (ingress-nginx, cert-manager, …)
```

## Roots (siblings)

| Application | Project | Points at |
|-------------|---------|-----------|
| `platform-root` | `platform` | Platform operators and shared config |
| `workloads-root` | `workloads` | `workloads/argocd/` (product apps) |

Cascade delete on one root does **not** delete the other root or its tree.

## Guardrails (summary)

- **Separate AppProjects** per root with resource **denylists** (PV, PVC, storage operators, `kube-system`, etc.).
- **No Longhorn/storage operator** under these roots in this repo (pattern documented for real clusters).
- **Prune** disabled or limited on platform apps until explicitly enabled.
- Register both roots after `iac/bootstrap` (manual apply or a one-time “bootstrap apps” step—TBD).

Implementation: **TBD** (YAML manifests land in later slices).
