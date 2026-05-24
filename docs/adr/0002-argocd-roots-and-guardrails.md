# ADR 0002: Argo CD app-of-apps roots and guardrails

## Status

Accepted

## Context

A single top-level GitOps root (e.g. one `gitops-root` Application) is simple but creates a **large blast radius**: deleting that Application in the Argo UI with **Cascade** can remove every child Application and the Kubernetes resources they manage—including storage operators, PVCs, or PVs if they live under the same tree.

Prior experience with cascade delete on a monolithic root motivates **splitting roots by ownership** and enforcing **AppProject** limits, not relying on UI discipline alone.

This portfolio runs on **Minikube** and does not deploy Longhorn or production storage here; the **pattern** (separate roots + projects) is what we document and implement in manifest layout.

## Decision

### Multiple app-of-apps roots (siblings, not one mega-tree)

Use **separate top-level** Argo CD `Application` resources as roots:

| Root | Manages | AppProject |
|------|---------|------------|
| **`platform-root`** | Ingress, cert-manager, observability, shared namespaces | `platform` |
| **`workloads-root`** | Product apps (e.g. hello-next) | `workloads` |

Roots are **siblings**. Cascade delete on `platform-root` affects only its descendant apps—not `workloads-root` and vice versa.

**Out of scope for this repo (document only):** a third `storage-root` for operators like Longhorn/CSI, with manual sync and the strictest project—recommended for real clusters, not required on Minikube MVP.

### AppProject guardrails

Each project restricts **what** Argo may sync and **where**:

- **`platform` project**
  - Allow: platform namespaces, ingress, cert-manager, monitoring CRDs in approved API groups.
  - Deny / blacklist: workload Deployments in app tenant namespaces, cluster storage (e.g. `PersistentVolume`, `PersistentVolumeClaim`, volume snapshot CRDs) unless explicitly added later.
  - Prefer `syncPolicy.automated.prune: false` on critical platform apps until prune behavior is well understood.

- **`workloads` project**
  - Allow: application namespaces, standard workload kinds (Deployment, Service, Ingress, HPA, etc.).
  - Deny: cluster-scoped infra, storage operators, `kube-system`, Argo CD’s own namespace.

Optional later: [Argo CD RBAC](https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/) so only platform admins can delete root Applications.

### Bootstrap vs GitOps (unchanged from ADR 0001)

- **Terraform** (`iac/bootstrap/`) installs Argo CD once.
- **Roots, projects, and child Applications** live in git under `platform/argocd/` and `workloads/argocd/`—Terraform does not reconcile the same resources.

### Operational guardrails (human + config)

| Practice | Rationale |
|----------|-----------|
| Avoid **Cascade** delete on root Applications in the UI | Non-cascading delete removes the app object without mass-deleting cluster resources (still use care). |
| Do not hang storage operators under `platform-root` or `workloads-root` | Storage gets its own root + project in real environments. |
| Keep prune off or scoped until policies are tested | Prevents git-driven mass pruning surprises. |
| Prefer project blacklists over “one big allowed list” | Explicit deny for PV/PVC and sensitive CRDs. |

## Consequences

### Positive

- Smaller blast radius than a single `gitops-root` for platform vs apps.
- Interview-ready story: learned from cascade delete, encoded guardrails in git.
- Clear mapping: `platform/` vs `workloads/` folders match Argo roots.

### Negative

- Two roots to register/sync instead of one (acceptable for clarity).
- AppProjects must be maintained as the stack grows.
- Blacklists are not a substitute for cluster policy (Kyverno/OPA) in production—optional future ADR.

## References

- [ADR 0001: Local development stack](0001-local-dev-stack.md)
- [platform/README.md](../../platform/README.md)
- [platform/argocd/README.md](../../platform/argocd/README.md)
- [Argo CD AppProject](https://argo-cd.readthedocs.io/en/stable/operator-manual/project/)
