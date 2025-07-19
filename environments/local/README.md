# Local Development Environment Setup

This guide sets up a complete local development environment with Vault, HTTPS certificates, and external secrets management.

## 🎯 What You'll Get

- 🔐 **Local Vault** with UI at https://vault.aircast.localhost
- 🌐 **HTTPS ingress** with self-signed certificates  
- 🔑 **External Secrets** pulling from local Vault
- 📋 **cert-manager** for automatic certificate management
- 🚀 **Production-like environment** for better testing

## 📋 Prerequisites

1. **Minikube** and **Helm** installed
2. **Docker** running
3. **Vault CLI** installed (for adding secrets)

## 🚀 Step-by-Step Setup

### Step 1: Start Minikube
```bash
# Start minikube with enough resources
minikube start --driver=docker --memory=4096 --cpus=2

# Enable required addons
minikube addons enable ingress
minikube addons enable ingress-dns
```

### Step 2: Install Infrastructure Components

#### Install cert-manager
```bash
# Add Helm repos
helm repo add jetstack https://charts.jetstack.io
helm repo update

# Install cert-manager
helm install cert-manager ../infrastructure/cert-manager \
  -f ../infrastructure/cert-manager/values-local.yaml \
  -n cert-manager-system --create-namespace

# Wait for cert-manager to be ready
kubectl wait --for=condition=ready pod -l app=cert-manager \
  -n cert-manager-system --timeout=300s
```

#### Install Vault
```bash
# Add Vault Helm repo
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update

# Install Vault
helm install vault ../infrastructure/vault \
  -f ../infrastructure/vault/values-local.yaml

# Wait for Vault to be ready
kubectl wait --for=condition=ready pod vault-0 --timeout=300s
```

#### Install External Secrets Operator
```bash
# Add External Secrets Helm repo
helm repo add external-secrets https://charts.external-secrets.io
helm repo update

# Install External Secrets
helm install external-secrets ../infrastructure/external-secrets \
  -f ../infrastructure/external-secrets/values-local.yaml \
  -n external-secrets-system --create-namespace

# Wait for External Secrets to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=external-secrets \
  -n external-secrets-system --timeout=300s
```

### Step 3: Configure DNS
```bash
# Add DNS entries to /etc/hosts
MINIKUBE_IP=$(minikube ip)
echo "$MINIKUBE_IP api.aircast.localhost" | sudo tee -a /etc/hosts
echo "$MINIKUBE_IP vault.aircast.localhost" | sudo tee -a /etc/hosts
```

### Step 4: Add Secrets to Vault
```bash
# Port forward to access Vault
kubectl port-forward vault-0 8200:8200 &

# Set Vault environment (wait a few seconds for port-forward to establish)
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='myroot'

# Add your secrets to Vault
vault kv put secret/local/aircast-api \
  POSTGRES_HOST="postgres" \
  POSTGRES_PORT="5432" \
  POSTGRES_NAME="aircast_local" \
  POSTGRES_USER="postgres" \
  POSTGRES_PASSWORD="password" \
  REDIS_HOST="redis" \
  REDIS_PORT="6379" \
  API_HOST="0.0.0.0" \
  API_PORT="3333" \
  API_PROTOCOL="https" \
  LOG_LEVEL="debug" \
  INGRESS_HOST="api.aircast.localhost" \
  INGRESS_PORT="443" \
  FRONTEND_URL="https://app.aircast.localhost" \
  COOKIE_DOMAIN=".aircast.localhost" \
  ADMIN_PASSWORD="admin123" \
  JWT_SECRET="local-jwt-secret" \
  AWS_ACCESS_KEY_ID="local-aws-key" \
  AWS_SECRET_ACCESS_KEY="local-aws-secret" \
  GOOGLE_OAUTH_CLIENT_ID="local-google-id" \
  GOOGLE_OAUTH_CLIENT_SECRET="local-google-secret"

# Stop port forward
pkill -f "kubectl port-forward vault-0"
```

### Step 5: Deploy Your Applications
```bash
# Navigate to your application repo and deploy
cd /path/to/aircast-api/charts
helm install aircast-api ./aircast-api -f ./aircast-api/values-local.yaml
```

## 🌍 Access Your Services

- **Vault UI**: https://vault.aircast.localhost (token: `myroot`)
- **API**: https://api.aircast.localhost/health

**Note**: Accept self-signed certificates in your browser when prompted.

## 🔍 Troubleshooting

### Check Infrastructure Status
```bash
# Check all pods
kubectl get pods --all-namespaces

# Check ingress
kubectl get ingress

# Check external secrets
kubectl get externalsecrets
kubectl get clustersecretstores

# Check certificates
kubectl get certificates
```

### Common Issues

#### Vault Not Accessible
```bash
# Check Vault pod
kubectl describe pod vault-0

# Check Vault ingress
kubectl describe ingress vault-ui-ingress
```

#### External Secrets Not Working
```bash
# Check external secrets operator
kubectl get pods -n external-secrets-system

# Check cluster secret store
kubectl describe clustersecretstore vault-kv-secret

# Check if secrets are being created
kubectl get secrets
```

#### Certificate Issues
```bash
# Check cert-manager
kubectl get pods -n cert-manager-system

# Check cluster issuer
kubectl get clusterissuer

# Check certificate requests
kubectl get certificaterequests
```

## 🧹 Cleanup

```bash
# Remove applications
helm uninstall aircast-api

# Remove infrastructure
helm uninstall external-secrets -n external-secrets-system
helm uninstall vault
helm uninstall cert-manager -n cert-manager-system

# Remove DNS entries
sudo sed -i '' '/aircast\.localhost/d' /etc/hosts

# Stop minikube
minikube stop
```

## 📚 Next Steps

1. **Deploy your application** using the GitOps repo
2. **Test external secrets** by changing values in Vault
3. **Explore Vault UI** to understand secret management
4. **Practice with certificates** by accessing HTTPS endpoints

This setup gives you a production-like local environment for development and testing!