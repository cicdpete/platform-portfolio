# Milestones

Numbered implementation slices for this portfolio. Each milestone is meant to be a reviewable PR with a clear demo outcome. Core path (M0–M4) stays **$0** (Minikube, public GHCR). **M5** is optional AWS-shaped demos and is not required to serve the Next.js app.

| ID | Scope | Primary paths | Status |
|----|--------|---------------|--------|
| **M0** | Plan, ADRs, monorepo scaffold | `docs/adr/`, README layout | **Done** (`main`, PR #1) |
| **M1** | Terraform installs Argo CD on Minikube | `iac/bootstrap/` | **In progress** (`feat/iac-for-argo`) |
| **M2** | AppProjects + sibling app-of-apps roots | `platform/argocd/` | Not started |
| **M3** | Next.js hello-world + GitOps manifests | `workloads/apps/`, `workloads/argocd/` | Not started |
| **M4** | CI: build, push public GHCR, update image tag | `.github/workflows/` | Not started |
| **M5** | Optional **AWS Emulator** demos (IAM/S3-shaped) | `iac/aws-emulator/` | Not started |

## Milestone detail

### M0 — Plan and scaffold

Accepted [ADR 0001](adr/0001-local-dev-stack.md) (local stack) and [ADR 0002](adr/0002-argocd-roots-and-guardrails.md) (roots and AppProject guardrails). Folder boundaries and golden-path runbook stubs exist; nothing deployable yet on `main` alone.

### M1 — Argo CD bootstrap (Terraform)

One-time Helm install of Argo CD via Terraform `kubernetes` / `helm` providers against an existing Minikube context. Stop at healthy Argo pods; no app-of-apps yet. Work lives on `feat/iac-for-argo`.

### M2 — GitOps roots

AppProjects plus sibling roots **`platform-root`** and **`workloads-root`** per ADR 0002. Cascade-delete blast radius stays limited by ownership split.

### M3 — Sample workload

Next.js hello-world under `workloads/apps/`, plus Helm/Kustomize and Argo `Application` manifests. Local fast path: `minikube image load` (or `docker-env`).

### M4 — GHCR CI

GitHub Actions at repo root builds/tests, pushes **public** `ghcr.io` images with `GITHUB_TOKEN`, updates the image tag in git for Argo to sync on local Minikube. No Actions deploy to a laptop cluster in MVP.

### M5 — AWS Emulator (optional)

Optional portfolio narrative for AWS-shaped IaC (e.g. IAM + S3) using **[AWS Emulator](https://aws.amazon.com/blogs/aws/test-your-applications-against-aws-locally-with-aws-emulator/)**. Not on the critical path for serving the app. Scaffold path: `iac/aws-emulator/`. Document that the emulator is for API shape/local demos—not a substitute for real IAM policy evaluation on AWS.

## Sequence

```text
M0 plan/ADRs  →  M1 Argo bootstrap  →  M2 roots  →  M3 hello-next  →  M4 GHCR CI
                                                              ↘
                                                               M5 AWS Emulator (optional, anytime after M0)
```

## Related

- [ADR 0001](adr/0001-local-dev-stack.md) · [ADR 0002](adr/0002-argocd-roots-and-guardrails.md)
- [Local golden path](runbooks/local-golden-path.md)
- Root [README.md](../README.md)
