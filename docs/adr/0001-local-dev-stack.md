# ADR 0001: Local development stack (zero-cost portfolio)

## Status

Accepted

## Context

This repository is a **portfolio** project: reviewers should clone it, follow one golden path, and reproduce the story without cloud spend or paid SaaS dependencies. We need:

- A real Kubernetes control plane and GitOps flow
- A custom application (not only a public `nginx` pull)
- CI that demonstrates build → registry → deploy-by-tag
- Optional AWS-shaped IaC for platform narrative (IAM, S3) without real AWS bills
- No committed secrets; minimal or zero GitHub Actions secrets for a **public** repo

## Decision

### Runtime: Minikube

- **Minikube** is the Kubernetes cluster for local development and demos.
- Cluster creation is **not** managed by Terraform in v1; use a runbook (`minikube start`, addons as needed).
- Production-shaped managed Kubernetes (e.g. EKS) is **out of scope** until a future ADR.

### GitOps: Argo CD

- **Argo CD** runs on Minikube and owns ongoing config for `platform/` and `workloads/`.
- **Terraform** in `iac/bootstrap/` performs a **one-time** install of Argo CD (`kubernetes` / `helm` providers).
- **Argo Application manifests** live under `platform/` and `workloads/`; Terraform must not fight GitOps for the same resources.
- **Two app-of-apps roots** (`platform-root`, `workloads-root`) and **AppProject** guardrails — see [ADR 0002](0002-argocd-roots-and-guardrails.md).

### Application: Next.js hello world

- MVP app: **Next.js** hello-world site under `workloads/apps/`.
- Deployed via Helm or Kustomize + Argo CD `Application`.

### Images: GHCR (CI) vs local load (laptop)

| Path | Mechanism | When |
|------|-----------|------|
| **CI** | GitHub Actions builds and pushes to **GHCR** (`ghcr.io/<owner>/...`) using **`GITHUB_TOKEN`** | Every push/merge to main (workflow TBD) |
| **Local fast path** | `docker build` + `minikube image load` (or `eval $(minikube docker-env)`) | Developer iteration without registry |
| **Cluster pull** | Minikube kubelet pulls **public** GHCR tag referenced in manifests | After CI publishes an image |

- **Production analogy:** ECR (or any registry) in real AWS; this repo uses **GHCR** for a free, reproducible CI story (LocalStack does not emulate ECR here).

### Optional AWS emulation: LocalStack

- **LocalStack** (optional) for Terraform against AWS APIs: **IAM** policies, **S3** buckets, etc.
- **Not** on the critical path for serving the Next.js app in v1.
- **Out of scope for LocalStack in this repo:** ECR/image registry emulation—images use GHCR per above.

### CI/CD: GitHub Actions

- All workflow files live at the **repository root** in `.github/workflows/` (GitHub does not run workflows from `workloads/apps/*/.github/`).
- Per-app pipelines are separate YAML files at the root (e.g. `hello-next.yml`), scoped with `on.push.paths` to `workloads/apps/hello-next/**` and related chart/Argo paths.
- App code and `Dockerfile` stay under `workloads/apps/`; deploy manifests under `workloads/argocd/` (and charts if used).
- lint/test, `docker build` + push to **public GHCR**, update image tag in git (GitOps).
- **No cluster deploy from Actions to a developer laptop** in MVP; Argo on **local Minikube** syncs from git.
- **No Actions secrets required** for MVP if the repo is public and GHCR packages are public (`GITHUB_TOKEN` suffices).

### IaC layout

- `iac/bootstrap/` — Terraform installs Argo CD on existing Minikube context.
- `iac/localstack/` (optional, later) — Terraform + LocalStack for IAM/S3 demos.
- **Not in v1:** VPC, EKS, real AWS credentials in CI.

### Cost

- **$0** target: Minikube local, public repo Actions minutes, public GHCR, optional LocalStack free/non-commercial tier if used later.

## Consequences

### Positive

- Reviewers need no AWS account for the core app path.
- CI mirrors real-world “build once, push to registry, deploy by tag.”
- Clear separation: `iac/` bootstrap vs `platform/` / `workloads/` GitOps.

### Negative / tradeoffs

- LocalStack IAM is **not** demonstrated by `minikube docker-env` or image pull alone; add an explicit `iac/localstack` + small access demo later.
- CI does not prove deploy to a remote cluster until a follow-up workflow or environment is added.
- Two image paths (local load vs GHCR) must be documented to avoid confusion.

## References

- Root [README.md](../../README.md)
- [docs/runbooks/local-golden-path.md](../runbooks/local-golden-path.md)
