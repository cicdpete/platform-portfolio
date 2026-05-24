# workloads/argocd

Argo CD **Application** manifests for product apps (e.g. hello-next).

Synced by the top-level **`workloads-root`** Application in [`platform/argocd/roots/`](../platform/argocd/roots/) (see [ADR 0002](../../docs/adr/0002-argocd-roots-and-guardrails.md)).

- **AppProject:** `workloads`
- **Not** managed under `platform-root`—smaller blast radius if a root is deleted with cascade.

Implementation: **TBD**.
