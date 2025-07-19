# Aircast GitOps Repository

Infrastructure as Code for the Aircast platform using Helm charts and GitOps practices.

## 🏗️ Repository Structure

```
aircast-gitops/
├── infrastructure/           # Core infrastructure components
│   ├── argocd/              # ArgoCD GitOps operator
│   ├── vault/               # Vault secret management
│   ├── cert-manager/        # Certificate management
│   └── external-secrets/    # External secrets operator
├── applications/            # ArgoCD Application manifests
│   ├── infrastructure/      # Infrastructure app definitions
│   └── app-of-apps.yaml     # Bootstrap app-of-apps
├── environments/            # Environment-specific configurations
│   ├── local/              # Local development setup
│   │   └── argocd/         # Local ArgoCD applications
│   ├── dev/                # Development environment
│   │   └── argocd/         # Dev ArgoCD applications
│   └── prod/               # Production environment
│       └── argocd/         # Prod ArgoCD applications
```

## 🚀 Quick Start

### Local Development
See the complete setup guide: [Local Environment Setup](./environments/local/README.md)

**Quick Commands:**
```bash
# 1. Setup infrastructure
cd environments/local
# Follow the step-by-step guide in README.md

# 2. Deploy applications
cd /path/to/aircast-api/charts
helm install aircast-api ./aircast-api -f ./aircast-api/values-local.yaml
```

### Development Environment
```bash
# 1. Install ArgoCD first
helm install argocd ./infrastructure/argocd \
  -f ./infrastructure/argocd/values-dev.yaml \
  -n argocd --create-namespace

# 2. Deploy the app-of-apps to manage all infrastructure
kubectl apply -f environments/dev/argocd/app-of-apps-dev.yaml

# ArgoCD will automatically deploy:
# - cert-manager
# - vault  
# - external-secrets
```

### Production Environment
```bash
# 1. Install ArgoCD first
helm install argocd ./infrastructure/argocd \
  -f ./infrastructure/argocd/values-prod.yaml \
  -n argocd --create-namespace

# 2. Deploy the app-of-apps to manage all infrastructure
kubectl apply -f environments/prod/argocd/app-of-apps-prod.yaml

# ArgoCD will automatically deploy:
# - cert-manager
# - vault
# - external-secrets
```

## 🔐 Secret Management

### Vault Paths by Environment

| Environment | Vault Path | Access |
|-------------|------------|--------|
| **Local** | `secret/local/aircast-api` | https://vault.aircast.localhost |
| **Dev** | `secret/development/aircast-api` | https://vault.dev.aircast.one |
| **Prod** | `secret/production/aircast-api` | https://vault.aircast.one |

### Adding Secrets to Vault

```bash
# Local development
export VAULT_ADDR='https://vault.aircast.localhost'
export VAULT_TOKEN='myroot'

vault kv put secret/local/aircast-api \
  POSTGRES_PASSWORD="your_password" \
  JWT_SECRET="your_jwt_secret"

# Development
export VAULT_ADDR='https://vault.dev.aircast.one'
export VAULT_TOKEN='your_dev_token'

vault kv put secret/development/aircast-api \
  POSTGRES_PASSWORD="dev_password" \
  JWT_SECRET="dev_jwt_secret"
```

## 🏗️ Infrastructure Components

### ArgoCD
- **Purpose**: GitOps deployment automation and management
- **Local**: Single instance, insecure mode for development
- **Dev**: HA setup with GitHub OAuth, staging certificates
- **Prod**: HA setup with strict RBAC, production certificates
- **Access**: 
  - Local: https://argocd.aircast.localhost (admin/admin)
  - Dev: https://argocd.dev.aircast.one (GitHub OAuth)
  - Prod: https://argocd.aircast.one (GitHub OAuth)

### cert-manager
- **Purpose**: Automatic TLS certificate management
- **Local**: Self-signed certificates
- **Dev**: Let's Encrypt staging certificates
- **Prod**: Let's Encrypt production certificates

### Vault
- **Purpose**: Centralized secret management
- **Local**: Single instance, dev mode
- **Dev**: HA setup with 3 replicas
- **Prod**: HA setup with 5 replicas, anti-affinity

### External Secrets Operator
- **Purpose**: Sync secrets from Vault to Kubernetes
- **All Envs**: Pulls secrets from Vault and creates K8s secrets

## 🌍 Environment Progression

```
Local Development → Development → Production
        ↓               ↓            ↓
vault.aircast.localhost → vault.dev.aircast.one → vault.aircast.one
argocd.aircast.localhost → argocd.dev.aircast.one → argocd.aircast.one
api.aircast.localhost   → dev.aircast.one       → api.aircast.one
```

## 📚 Learning Path

1. **Start Local** - Follow [Local Setup Guide](./environments/local/README.md)
2. **Understand Components** - Learn Vault, cert-manager, external-secrets
3. **Practice GitOps** - Make changes, see them deployed
4. **Scale Up** - Move to dev/prod environments

## 🔍 Monitoring and Troubleshooting

### Check Infrastructure Health
```bash
# All pods across namespaces
kubectl get pods --all-namespaces

# ArgoCD health
kubectl get pods -n argocd
kubectl get applications -n argocd

# Infrastructure-specific checks
kubectl get pods -n cert-manager-system
kubectl get pods -n external-secrets-system
kubectl get pod vault-0

# Check secrets and certificates
kubectl get externalsecrets
kubectl get certificates
kubectl get clustersecretstores
```

### Useful Commands
```bash
# ArgoCD application status
kubectl get applications -n argocd
kubectl describe application <app-name> -n argocd

# View ArgoCD logs
kubectl logs -n argocd deployment/argocd-server
kubectl logs -n argocd deployment/argocd-application-controller

# View Vault logs
kubectl logs vault-0

# Check external secret sync
kubectl describe externalsecret <name>

# View certificate status
kubectl describe certificate <name>

# Port forward to services (for debugging)
kubectl port-forward vault-0 8200:8200
kubectl port-forward -n argocd svc/argocd-server 8080:443
```

## 🛡️ Security Best Practices

1. **Never commit secrets** to this repository
2. **Use environment-specific tokens** for Vault access
3. **Rotate Vault tokens regularly** in dev/prod
4. **Use least-privilege access** for applications
5. **Monitor secret access** via Vault audit logs

## 🤝 Contributing

1. **Infrastructure changes** go in `infrastructure/`
2. **Environment configs** go in `environments/`
3. **Test locally first** using the local environment
4. **Document changes** in relevant README files

This repository follows GitOps principles - infrastructure changes are applied through version control and automated deployment.