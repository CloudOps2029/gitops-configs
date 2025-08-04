CloudOps2029/gitops-configs
Overview
This repository contains Kubernetes manifests and configurations for deploying a 3-tier application on Azure Kubernetes Service (AKS) using GitOps with ArgoCD. It supports development, test, and production environments with Terraform-based infrastructure management.
Components

Infrastructure: Azure Kubernetes Service (AKS) with auto-scaling
GitOps Platform: ArgoCD for continuous deployment
State Management: Terraform with optional remote state in Azure Storage
Authentication: Azure AD integration with local admin accounts
Networking: Azure CNI with network policies

Environment Structure
.
├── dev                     # Development environment
│   ├── argocd-app-manifest.yaml
│   ├── backend.tf
│   ├── backend.tf.example
│   ├── deploy-argocd-app.sh
│   ├── kubernetes-resources.tf
│   ├── main.tf
│   ├── outputs.tf
│   ├── provider.tf
│   ├── terraform.tfvars
│   ├── validate-deployment.sh
│   └── variables.tf
├── prod                    # Production environment
│   ├── argocd-app-manifest.yaml
│   ├── backend.tf
│   ├── backend.tf.example
│   ├── deploy-argocd-app.sh
│   ├── kubernetes-resources.tf
│   ├── main.tf
│   ├── outputs.tf
│   ├── provider.tf
│   ├── terraform.tfvars
│   └── variables.tf
├── test                    # Test environment
│   ├── argocd-app-manifest.yaml
│   ├── backend.tf
│   ├── backend.tf.example
│   ├── deploy-argocd-app.sh
│   ├── kubernetes-resources.tf
│   ├── main.tf
│   ├── outputs.tf
│   ├── provider.tf
│   ├── terraform.tfvars
│   └── variables.tf
├── manifests               # Kubernetes manifests for 3-tier application
│   ├── argocd-application.yaml
│   ├── backend-config.yaml
│   ├── backend.yaml
│   ├── frontend-config.yaml
│   ├── frontend.yaml
│   ├── kustomization.yaml
│   ├── namespace.yaml
│   ├── postgres-config.yaml
│   ├── postgres-pvc.yaml
│   ├── postgres.yaml
│   └── key-vault-secrets.yaml
└── README.md

✅ Prerequisites
1. Essential Tools (Required)
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Install Terraform
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

2. Optional Tools (for manual management)
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

Note: Terraform handles ArgoCD installation and Kubernetes resources. Use kubectl/helm for manual troubleshooting.
3. Azure Authentication & Service Principal
Step 1: Login to Azure
az login
az account set --subscription "your-subscription-id"
az account show

Step 2: Create Service Principal for Terraform
SUBSCRIPTION_ID=$(az account show --query id --output tsv)
echo "Subscription ID: $SUBSCRIPTION_ID"

az ad sp create-for-rbac \
  --name "terraform-aks-gitops" \
  --role "Contributor" \
  --scopes "/subscriptions/$SUBSCRIPTION_ID" \
  --sdk-auth

Save the output (clientId, clientSecret, subscriptionId, tenantId) for Terraform authentication.
Step 3: Configure Terraform Authentication
Option A: Environment Variables (Recommended)
export ARM_CLIENT_ID="your-client-id"
export ARM_CLIENT_SECRET="your-client-secret"
export ARM_SUBSCRIPTION_ID="your-subscription-id"
export ARM_TENANT_ID="your-tenant-id"
terraform version

Option B: Azure CLI Authentication
az login
az account set --subscription "your-subscription-id"

Step 4: SSH Key (Optional)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_azure -N ""

🔄 Complete Recreation Guide
⚠️ CRITICAL: Follow this sequence to ensure successful deployment.
🎯 Step 1: Set Up Your GitOps Repository
1.1 Create GitHub Repository

Create a repository named gitops-configs on GitHub.
Description: "Kubernetes manifests for 3-tier application GitOps deployment"
Visibility: Public (or private with access configured)
Initialize with README

git clone https://github.com/CloudOps2029/gitops-configs.git
cd gitops-configs

1.2 Copy and Push Manifest Files
# Copy manifests to your repository
mkdir manifests
cp /path/to/source/manifests/* manifests/

# Verify files
ls -la manifests/
# Expected:
# ├── argocd-application.yaml
# ├── backend-config.yaml
# ├── backend.yaml
# ├── frontend-config.yaml
# ├── frontend.yaml
# ├── kustomization.yaml
# ├── namespace.yaml
# ├── postgres-config.yaml
# ├── postgres-pvc.yaml
# ├── postgres.yaml
# ├── key-vault-secrets.yaml

1.3 Update Repository URL in Manifest Files
Edit manifests/argocd-application.yaml:
# Change:
# repoURL: https://github.com/ORIGINAL_USERNAME/gitops-configs.git
# To:
repoURL: https://github.com/CloudOps2029/gitops-configs.git

1.4 Commit and Push
git add .
git commit -m "Add 3-tier application manifests"
git push origin main

1.5 Verify Repository
curl -s https://api.github.com/repos/CloudOps2029/gitops-configs

🔧 Step 2: Update Terraform Configuration Files
2.1 Update Repository URLs
For each environment (dev, test, prod):
cd dev/
vim terraform.tfvars
# Update: app_repo_url = "https://github.com/CloudOps2029/gitops-configs.git"

Repeat for test/terraform.tfvars and prod/terraform.tfvars.
2.2 Customize Resource Names (Optional)
Edit terraform.tfvars in each environment:
resource_group_name     = "my-aks-rg"
kubernetes_cluster_name = "my-aks-cluster"

2.3 Validate GitOps Repository Access
curl -s https://raw.githubusercontent.com/CloudOps2029/gitops-configs/main/namespace.yaml

🔧 Step 3: Configure Remote State Backend (Optional)
cd dev/
cp backend.tf.example backend.tf

Create Azure Storage for state:
az group create --name "rg-terraform-state" --location "East US"
STORAGE_ACCOUNT_NAME="tfstate$(date +%s)"
az storage account create \
  --resource-group "rg-terraform-state" \
  --name "$STORAGE_ACCOUNT_NAME" \
  --sku "Standard_LRS" \
  --encryption-services blob
az storage container create \
  --name "tfstate" \
  --account-name "$STORAGE_ACCOUNT_NAME"

Update backend.tf:
resource_group_name  = "rg-terraform-state"
storage_account_name = "<STORAGE_ACCOUNT_NAME>"
container_name       = "tfstate"
key                  = "dev/terraform.tfstate"

🔧 Step 4: Deploy Development Environment
cd dev/
terraform init
terraform plan
terraform apply -auto-approve

Deploys:

AKS cluster with auto-scaling
ArgoCD via Helm
Azure AD integration
Network policies
3-tier application via GitOps

🔧 Step 5: Configure kubectl Access
az aks get-credentials \
  --resource-group $(terraform output -raw resource_group_name) \
  --name $(terraform output -raw aks_cluster_name) \
  --admin \
  --overwrite-existing
kubectl get nodes
kubectl get namespaces

🔧 Step 6: Verify ArgoCD Installation
kubectl get pods -n argocd
kubectl get svc -n argocd

🔐 Step 7: Configure Key Vault Integration
7.1 Get Key Vault Information
cd dev/
KEY_VAULT_NAME=$(terraform output -raw key_vault_name)
TENANT_ID=$(az account show --query tenantId -o tsv)
KV_SECRETS_PROVIDER_CLIENT_ID=$(az aks show \
  --resource-group $(terraform output -raw resource_group_name) \
  --name $(terraform output -raw aks_cluster_name) \
  --query "addonProfiles.azureKeyvaultSecretsProvider.identity.clientId" -o tsv)

7.2 Update GitOps Repository
Edit manifests/key-vault-secrets.yaml:
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: postgres-secrets-provider
  namespace: 3tirewebapp-dev
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    useVMManagedIdentity: "true"
    userAssignedIdentityID: "<KV_SECRETS_PROVIDER_CLIENT_ID>"
    keyvaultName: "<KEY_VAULT_NAME>"
    tenantId: "<TENANT_ID>"
    objects: |
      array:
        - |
          objectName: postgres-username
          objectType: secret
        - |
          objectName: postgres-password
          objectType: secret
        - |
          objectName: postgres-database
          objectType: secret
        - |
          objectName: postgres-connection-string
          objectType: secret
  secretObjects:
  - secretName: postgres-credentials-from-kv
    type: Opaque
    data:
    - objectName: postgres-username
      key: POSTGRES_USER
    - objectName: postgres-password
      key: POSTGRES_PASSWORD
    - objectName: postgres-database
      key: POSTGRES_DB
    - objectName: postgres-connection-string
      key: DATABASE_URL
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: key-vault-config
  namespace: 3tirewebapp-dev
data:
  KEY_VAULT_NAME: "<KEY_VAULT_NAME>"
  KEY_VAULT_SECRET_POSTGRES_USERNAME: "postgres-username"
  KEY_VAULT_SECRET_POSTGRES_PASSWORD: "postgres-password"
  KEY_VAULT_SECRET_POSTGRES_DATABASE: "postgres-database"
  KEY_VAULT_SECRET_CONNECTION_STRING: "postgres-connection-string"

7.3 Commit and Push
cd /path/to/gitops-configs
git add manifests/key-vault-secrets.yaml
git commit -m "Configure Key Vault integration"
git push origin main

7.4 Verify ArgoCD Sync
kubectl get applications -n argocd
kubectl patch application 3tirewebapp-dev -n argocd --type merge \
  --patch '{"operation":{"sync":{"syncStrategy":{"force":true}}}}'
kubectl get secretproviderclass -n 3tirewebapp-dev

🏢 Multi-Environment Setup



Environment
Location
VM Size
Node Count
Auto-Scaling
OS Disk
Use Case



Dev
East US
Standard_D2s_v3
2
1-5 nodes
30GB
Development & Testing


Test
East US 2
Standard_D4s_v3
3
2-8 nodes
50GB
Integration Testing


Prod
West US 2
Standard_D8s_v3
5
3-10 nodes
100GB
Production Workloads


Deployment Instructions
For each environment:
cd <env>/
terraform init
terraform plan
terraform apply -auto-approve

🌐 Accessing ArgoCD WebUI
ARGOCD_IP=$(kubectl get svc argocd-server -n argocd -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
ARGOCD_PASSWORD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
echo "ArgoCD URL: http://$ARGOCD_IP"
echo "Username: admin"
echo "Password: $ARGOCD_PASSWORD"

Access: http://<EXTERNAL-IP>
🎯 Quick Application Access
kubectl port-forward svc/frontend -n 3tirewebapp-dev 3000:3000

Access: http://localhost:3000
🌐 Using Built-in Ingress
Step 1: Install NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=90s

Step 2: Configure Local Domain
INGRESS_IP=$(kubectl get svc ingress-nginx-controller -n ingress-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "$INGRESS_IP 3tirewebapp-dev.local" | sudo tee -a /etc/hosts

Access: http://3tirewebapp-dev.local
🔍 3-Tier Application Architecture

Frontend (React): Service on port 3000, Ingress: 3tirewebapp-dev.local
Backend (Node.js): Service on port 8080, connects to PostgreSQL
Database (PostgreSQL): Service on port 5432, uses persistent volume

✅ Verification
chmod +x validate-deployment.sh
./validate-deployment.sh

Health Indicators:

All nodes "Ready"
ArgoCD pods "Running"
Applications "Synced" and "Healthy"

🧹 Clean Up
kubectl delete applications --all -n argocd
cd dev/
terraform destroy -auto-approve
kubectl config delete-context <cluster-context>

🎯 Next Steps

Explore ArgoCD WebUI
Deploy additional applications
Configure monitoring with Azure Monitor
Enable TLS for production


⚠️ Important: Key Vault Identity
Use the correct identity for SecretProviderClass:
az aks show --resource-group $(terraform output -raw resource_group_name) \
  --name $(terraform output -raw aks_cluster_name) \
  --query "addonProfiles.azureKeyvaultSecretsProvider.identity.clientId" -o tsv

Footer
© 2025 GitHub, Inc.
