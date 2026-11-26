# iac

Infrastructure as code for **bootstrap** and **optional AWS-shaped demos**—not for provisioning Minikube or real EKS in v1.

See [ADR 0001: Local development stack](../docs/adr/0001-local-dev-stack.md) and [milestones](../docs/milestones.md).

## Boundaries

| In scope (`iac/`) | Out of scope (elsewhere) |
|-------------------|---------------------------|
| Terraform **installs Argo CD** on an existing cluster (`bootstrap/`) | Minikube creation → runbook / `minikube start` |
| Optional AWS Emulator: IAM, S3 (`aws-emulator/`, **M5**) | Ongoing Argo `Application` manifests → `platform/`, `workloads/` |
| Helm/kubernetes providers, local Terraform state for bootstrap | Next.js app source → `workloads/apps/` |
| | Application image build/push → GitHub Actions → GHCR |

## Planned layout

```
iac/
├── README.md
├── bootstrap/          # Terraform: helm/kubernetes → Argo CD on Minikube
│   └── (TBD)
└── aws-emulator/       # Optional (M5): AWS-shaped demos via AWS Emulator
    └── (TBD)
```

**v1 does not include:** VPC, EKS, real AWS credentials, or Terraform that creates the Kubernetes cluster.

## Bootstrap flow

1. Operator runs `minikube start` and sets `kubectl` context.
2. `cd iac/bootstrap && terraform init && terraform apply`
3. Argo CD runs in cluster; GitOps takes over from `platform/` and `workloads/`.

Terraform manages **only** the initial Argo CD install. Do not let Terraform and Argo both own the same resources after bootstrap.

## Optional AWS Emulator slice (M5)

For portfolio narrative (“IAM policy allows S3 read”, etc.):

- Run **AWS Emulator** locally (documented in runbooks when M5 lands).
- Point Terraform `aws` provider / AWS CLI/SDK at the emulator endpoint and credentials it generates.
- Treat demos as API-shape exercises—not proof of real IAM evaluation.
- **Images:** GHCR (CI) and `minikube image load` (local)—out of scope for this slice.

## State and secrets

- Use **local** Terraform state for bootstrap in v1 (`.gitignore` `*.tfstate*`).
- Do not commit secrets, `terraform.tfvars` with credentials, or kubeconfigs.
