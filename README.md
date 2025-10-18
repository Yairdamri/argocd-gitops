# GitOps Repository - Workout Application

Declarative Kubernetes deployments for workout application and platform infrastructure using ArgoCD App-of-Apps pattern.

---

## 🌐 Overview

**GitOps**: All Kubernetes manifests in Git. ArgoCD auto-syncs cluster state.

**Pattern**: App-of-Apps (3 parent apps manage child apps with sync waves)  
**Target**: AWS EKS cluster

---

## 📁 Repository Structure

```
argocd/
├── *-parent.yaml       # 3 parent apps (infra, applications, logging)
├── infra/              # SealedSecrets, Cert-Manager, Ingress, Prometheus
├── applications/       # MongoDB Operator, MongoDB ReplicaSet, Workout App
├── logging/            # Elasticsearch, Fluent-bit, Kibana
└── manifests/          # Raw K8s manifests (SealedSecrets, Ingress)
```

**Deployment order**: Infra (wave 0) → Apps (wave 1-2) → Logging (wave 3)

---

## 🚀 Deploy

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Bootstrap all apps
kubectl apply -f infra-parent.yaml
kubectl apply -f applications-parent.yaml
kubectl apply -f logging-parent.yaml
```

**Access UI**: `kubectl port-forward svc/argocd-server -n argocd 8080:443`

### ArgoCD Applications View

![ArgoCD Applications Dashboard](../images/Argocd%20Ui.png)

*ArgoCD UI showing all deployed applications: infrastructure, applications, and logging stacks*

### Monitoring Dashboard

![Monitoring Stack](../images/monitoring.png)

*Prometheus & Grafana monitoring stack deployed via ArgoCD*

---

## 🔁 Deployment Flow

```
Jenkins builds image → Updates Helm values → Commits to Git
    ↓
ArgoCD detects change (3min)
    ↓
Rolling update to EKS
```

---

## 🛡️ Secrets

**SealedSecrets**: Encrypt secrets with `kubeseal`, commit to Git safely  
**Example**: `kubectl create secret ... | kubeseal > sealed-secret.yaml`

---

## ⚙️ Operations

```bash
argocd app sync <app-name>         # Manual sync
argocd app rollback <app-name>     # Rollback
argocd app diff <app-name>         # View changes
```

---

## 📝 Notes

- **Git = source of truth** - Manual changes reverted by self-heal
- **Auto-sync + prune** - Deleted manifests = deleted resources
- **Sync waves** - Ensures dependency order (infra → db → app)

---

**Author**: Yair Damri | DevOps Portfolio (2025)

