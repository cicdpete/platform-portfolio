# iac

Infrastructure as code for **bootstrap** and **optional AWS-shaped demos**—not for provisioning Minikube or real EKS in v1.

See [ADR 0001: Local development stack](../docs/adr/0001-local-dev-stack.md).

## Boundaries

| In scope (`iac/`) | Out of scope (elsewhere) |
|-------------------|---------------------------|
| Terraform **installs Argo CD** on an existing cluster (`bootstrap/`) | Minikube creation → runbook / `minikube start` |
| Optional LocalStack: IAM, S3, ECR **repository** resources (`localstack/`, later) | Ongoing Argo `Application` manifests → `platform/`, `workloads/` |
| Helm/kubernetes providers, local Terraform state for bootstrap | Next.js app source → `workloads/apps/` |
| | Application image build/push → GitHub Actions → GHCR |

## Planned layout

```
iac/
├── README.md
├── bootstrap/          # Terraform: helm/kubernetes → Argo CD on Minikube
│   └── (TBD)
└── localstack/         # Optional: AWS provider → LocalStack endpoint
    └── (TBD)
```

**v1 does not include:** VPC, EKS, real AWS credentials, or Terraform that creates the Kubernetes cluster.

## Bootstrap flow

1. Operator runs `minikube start` and sets `kubectl` context.
2. `cd iac/bootstrap && terraform init && terraform apply`
3. Argo CD runs in cluster; GitOps takes over from `platform/` and `workloads/`.

Terraform manages **only** the initial Argo CD install. Do not let Terraform and Argo both own the same resources after bootstrap.

## Optional LocalStack slice

For portfolio narrative (“IAM policy allows S3 read”, etc.):

- Run LocalStack via compose or CLI (documented in runbooks when added).
- Terraform `aws` provider with `endpoints` / `skip_*` aimed at `http://localhost:4566`.
- **ECR:** creating repos via API is fine; **do not** depend on host `docker push` to LocalStack ECR for the Next.js MVP—use GHCR in CI and `minikube image load` locally.

## State and secrets

- Use **local** Terraform state for bootstrap in v1 (`.gitignore` `*.tfstate*`).
- Do not commit secrets, `terraform.tfvars` with credentials, or kubeconfigs.
