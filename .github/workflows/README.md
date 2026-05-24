# GitHub Actions

CI for a **public** portfolio repo: validate code, build the Next.js image, push to **GHCR**, update GitOps manifests (image tag).

See [ADR 0001](../../docs/adr/0001-local-dev-stack.md).

## Monorepo: where workflow files live

GitHub Actions **only** runs workflows from the **repository root**:

```
.github/workflows/*.yml    ← all pipeline YAML here
```

Do **not** put workflows under `workloads/apps/hello-next/.github/`—GitHub will not execute them.

| Concern | Location |
|---------|----------|
| Workflow definitions | `.github/workflows/` (repo root) |
| App source + `Dockerfile` | `workloads/apps/hello-next/` |
| Helm/Kustomize + Argo `Application` | `workloads/charts/`, `workloads/argocd/` |

Scope a workflow to one app with `on.push.paths` (and matching `pull_request.paths` if needed), for example:

```yaml
on:
  push:
    paths:
      - 'workloads/apps/hello-next/**'
      - 'workloads/argocd/**'
      - '.github/workflows/hello-next.yml'
```

Optional: reusable workflows or composite actions under `.github/` at the root—not under the app tree.

## Planned workflows

| Workflow | Purpose |
|----------|---------|
| `ci.yml` (TBD) | Lint/test Next.js |
| `image.yml` (TBD) | `docker build` + push to `ghcr.io/<owner>/hello-next` |
| `terraform.yml` (TBD, optional) | `terraform fmt` / `validate` on `iac/bootstrap` |

## Secrets (MVP)

For a **public** repo and **public** GHCR package:

- Use built-in **`GITHUB_TOKEN`** with `packages: write` — **no custom repository secrets** required.
- Do not deploy to a developer’s Minikube from Actions in v1; Argo on local Minikube syncs after git updates.

Add secrets later only if needed: private GHCR, `ARGOCD_AUTH_TOKEN` for remote sync, LocalStack auth, real AWS.

## What Actions does not do (v1)

- Start or reach Minikube on your laptop
- Replace Argo CD for deployment to the cluster you run locally

## Cost

Public repository standard runners and public GHCR packages are **free** under normal portfolio usage. See [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions) and [Packages billing](https://docs.github.com/en/billing/concepts/product-billing/github-packages).
