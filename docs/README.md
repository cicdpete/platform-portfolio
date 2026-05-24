# docs

Architecture decision records (ADRs), diagrams, and runbooks for reviewers and future-you.

## Contents

| Path | Description |
|------|-------------|
| [adr/0001-local-dev-stack.md](adr/0001-local-dev-stack.md) | **Accepted** — Minikube, Argo, GHCR, Terraform bootstrap, optional LocalStack |
| [runbooks/local-golden-path.md](runbooks/local-golden-path.md) | Step-by-step local deploy (commands filled in per slice) |
| `adr/` | Future ADRs (ingress/TLS, observability, etc.) |
| `runbooks/` | Destroy checklist, LocalStack setup (when added) |

## Conventions

- ADRs: `NNNN-short-title.md` with status, context, decision, consequences
- No secrets in this tree
- Mermaid diagrams may live in ADRs or the root README
