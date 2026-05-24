# Runbook: Local golden path

> Commands marked **(TBD)** will be filled in as each slice lands. See [ADR 0001](../adr/0001-local-dev-stack.md).

## Prerequisites

- Docker
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Terraform](https://developer.hashicorp.com/terraform/install) `>= 1.6`
- [Helm](https://helm.sh/docs/intro/install/) (for Terraform Argo bootstrap)
- Optional: [Argo CD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)

## 1. Start the cluster

```bash
minikube start
# Optional: minikube addons enable ingress
kubectl cluster-info
```

## 2. Bootstrap Argo CD (Terraform)

```bash
# (TBD) cd iac/bootstrap && terraform init && terraform apply
kubectl get pods -n argocd
```

## 3. Access Argo CD UI

```bash
# (TBD) kubectl port-forward svc/argocd-server -n argocd 8080:443
# (TBD) argocd admin initial-password -n argocd
```

## 4. Sync AppProjects and both roots

Two **sibling** app-of-apps roots (`platform-root`, `workloads-root`)—not one monolithic `gitops-root`. See [ADR 0002](../adr/0002-argocd-roots-and-guardrails.md).

```bash
# (TBD) Apply AppProjects from platform/argocd/projects/
# (TBD) Apply or sync platform-root → platform apps (ingress, cert-manager, …)
# (TBD) Apply or sync workloads-root → workloads/argocd/
# (TBD) argocd app sync platform-root
# (TBD) argocd app sync workloads-root
```

Avoid **Cascade** delete on root Applications in the Argo UI.

## 5. Application image — local fast path

For development without pushing to GHCR:

```bash
# (TBD) cd workloads/apps/hello-next
docker build -t hello-next:local .
minikube image load hello-next:local
# Manifest uses hello-next:local and imagePullPolicy: Never (or IfNotPresent)
```

Alternative: build inside Minikube’s Docker daemon:

```bash
eval $(minikube docker-env)
docker build -t hello-next:local .
```

## 6. Application image — CI / GHCR path

After GitHub Actions pushes a **public** image to `ghcr.io/<owner>/hello-next:<tag>`:

1. Ensure workload manifests reference that image and tag.
2. Argo CD syncs; Minikube pulls from GHCR (public = no `imagePullSecret`).

## 7. Tear down

```bash
# (TBD) terraform destroy in iac/bootstrap if applied
minikube delete
```

No cloud resources to destroy for the core MVP. Stop LocalStack containers if you started them for optional IaC demos.

## CI note

Actions builds and pushes images; it does **not** connect to your laptop’s Minikube. For “full stack on my machine,” run steps 1–5 locally after merging CI manifest updates (or use local image override in step 5).
