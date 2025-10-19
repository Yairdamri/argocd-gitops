# ArgoCD GitOps Repository

Declarative delivery for the Workout Generator & Tracker stack, using ArgoCD’s App-of-Apps pattern to sync infrastructure, applications, and logging into the EKS cluster.

## What Lives Here

| Component | Path | Purpose |
| --- | --- | --- |
| Parent applications | `*-parent.yaml` | Bootstrap entrypoints that register child apps and enforce sync waves |
| Infrastructure apps | `infra/` | Cert-Manager, Ingress, SealedSecrets, Prometheus, shared cluster services |
| Application tier | `applications/` | MongoDB operator + replica set, backend + frontend stack, supporting config |
| Logging & monitoring | `logging/` | Elasticsearch, Fluent Bit, Kibana, Grafana dashboards |
| Raw manifests | `manifests/` | SealedSecrets and ad-hoc K8s resources referenced by the apps above |

![ArgoCD app topology](../images/Argocd%20Ui.png)  
*Parent/child app graph once the repository is registered*

## Terraform Integration

The Terraform code in `infra/` (see `infra/modules/argocd` invoked from `infra/main`) provisions ArgoCD in the target cluster **and** applies the three parent manifests (`infra-parent.yaml`, `applications-parent.yaml`, `logging-parent.yaml`) automatically. Running `terraform apply` in that repo yields a fully bootstrapped GitOps control plane; manual steps below serve as a fallback or for non-Terraform environments.

## Prerequisites

- ArgoCD v2.8+ installed in the cluster (`kubectl create namespace argocd`)
- Cluster access with `kubectl` and ArgoCD CLI
- Git access token/SSH key configured in ArgoCD for this repo
- Secrets sealed with the cluster’s SealedSecrets public key

## Bootstrap Flow (Manual Option)

```bash
# Install ArgoCD (once per cluster)
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Point ArgoCD at this Git repo (replace URL and branch as needed)
argocd repo add git@gitlab.com:yair_portfolio/argocd-gitops.git --ssh-private-key-path ~/.ssh/id_rsa

# Register the three parent applications
kubectl apply -f infra-parent.yaml
kubectl apply -f applications-parent.yaml
kubectl apply -f logging-parent.yaml
