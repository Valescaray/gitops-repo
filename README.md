# Invoice Analyzer GitOps Repository

This repository contains the Kubernetes manifests for the Invoice Analyzer application, managed using GitOps principles with ArgoCD.

## Architecture

The Invoice Analyzer application consists of:
- **Frontend**: React-based web application
- **Backend**: Python FastAPI application with LangChain integration
- **Database**: PostgreSQL database
- **Monitoring**: Prometheus Operator + Grafana
- **Ingress**: AWS ALB Ingress Controller
- **Secrets**: External Secrets Operator with AWS Secrets Manager

## Repository Structure

```
gitops-repo/
├── argocd/
│   └── applications/          # ArgoCD Application manifests
├── base/                      # Base Kubernetes manifests
│   ├── frontend/
│   ├── backend/
│   ├── database/
│   ├── ingress/
│   ├── monitoring/
│   │   ├── prometheus-operator/
│   │   └── grafana/
│   ├── rbac/
│   ├── network-policies/
│   └── external-secrets/
├── environments/              # Environment-specific overlays
│   ├── dev/
│   └── prod/
└── namespaces/               # Namespace definitions
```

## Prerequisites

1. **Kubernetes Cluster** (EKS recommended)
2. **ArgoCD** installed in the cluster
3. **AWS ALB Ingress Controller** installed
4. **Prometheus Operator** installed
5. **External Secrets Operator** installed
6. **AWS Resources**:
   - ACM Certificate for SSL/TLS
   - Route53 hosted zone
   - AWS Secrets Manager secrets
   - IAM roles for IRSA (IAM Roles for Service Accounts)

## Setup Instructions

### 1. Install Required Operators

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Install AWS Load Balancer Controller
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<YOUR_CLUSTER_NAME>

# Install Prometheus Operator
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus-operator prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace

# Install External Secrets Operator
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets \
  -n external-secrets-system --create-namespace
```

### 2. Configure AWS Secrets Manager

Create the following secrets in AWS Secrets Manager:

```bash
# Backend secrets
aws secretsmanager create-secret \
  --name invoice-analyzer/backend \
  --secret-string '{
    "database-url": "postgresql://user:password@postgres-lb:5432/invoice_analyzer",
    "aws-access-key-id": "YOUR_ACCESS_KEY",
    "aws-secret-access-key": "YOUR_SECRET_KEY",
    "openai-api-key": "YOUR_OPENAI_KEY"
  }'

# Database secrets
aws secretsmanager create-secret \
  --name invoice-analyzer/database \
  --secret-string '{
    "username": "postgres",
    "password": "YOUR_DB_PASSWORD"
  }'

# Grafana secrets
aws secretsmanager create-secret \
  --name invoice-analyzer/grafana \
  --secret-string '{
    "admin-user": "admin",
    "admin-password": "YOUR_GRAFANA_PASSWORD"
  }'
```

### 3. Configure IRSA (IAM Roles for Service Accounts)

Create IAM roles and policies for:
- Backend service account (S3 access)
- External Secrets Operator (Secrets Manager access)

```bash
# Example for External Secrets Operator
eksctl create iamserviceaccount \
  --name external-secrets-sa \
  --namespace invoice-analyzer-dev \
  --cluster <YOUR_CLUSTER_NAME> \
  --attach-policy-arn arn:aws:iam::aws:policy/SecretsManagerReadWrite \
  --approve
```

### 4. Update Configuration

Update the following placeholders in the manifests:

- `<REGISTRY>`: Your container registry (e.g., `123456789.dkr.ecr.us-east-1.amazonaws.com`)
- `<AWS_ACCOUNT_ID>`: Your AWS account ID
- `<CERTIFICATE_ARN>`: ACM certificate ARN
- `<SECURITY_GROUP_ID>`: Security group ID for ALB
- `<DOMAIN_NAME>`: Your domain name
- `<YOUR_ORG>`: Your GitHub organization/username

### 5. Deploy with ArgoCD

```bash
# Apply ArgoCD applications
kubectl apply -f argocd/applications/

# Check application status
argocd app list
argocd app get invoice-analyzer-dev
```

## Environments

### Development (dev)
- **Namespace**: `invoice-analyzer-dev`
- **Domain**: `dev.invoice-analyzer.example.com`
- **Auto-sync**: Enabled with auto-prune
- **Resources**: Minimal for cost optimization

### Production (prod)
- **Namespace**: `invoice-analyzer-prod`
- **Domain**: `invoice-analyzer.example.com`
- **Auto-sync**: Enabled (manual prune for safety)
- **Resources**: Production-grade with high availability
- **WAF**: Enabled for security

## Monitoring

### Prometheus
- **URL**: Internal cluster service
- **Retention**: 30 days
- **Storage**: 50Gi

### Grafana
- **URL**: `grafana.<DOMAIN_NAME>`
- **Default Dashboard**: Invoice Analyzer Overview
- **Datasource**: Prometheus (auto-configured)

## Security Features

1. **Network Policies**: Restrict traffic between components
2. **RBAC**: Least-privilege service accounts
3. **External Secrets**: Secrets stored in AWS Secrets Manager
4. **IRSA**: No static credentials in pods
5. **WAF**: Web Application Firewall (prod only)

## Customization

### Adding a New Environment

1. Create a new directory under `environments/`
2. Create component-specific kustomization files
3. Create an ArgoCD Application manifest
4. Apply the ArgoCD application

### Modifying Resources

1. Update base manifests in `base/` for global changes
2. Update environment overlays for environment-specific changes
3. Commit and push changes
4. ArgoCD will automatically sync (if auto-sync is enabled)

## Troubleshooting

### Check ArgoCD Application Status
```bash
argocd app get invoice-analyzer-dev
argocd app sync invoice-analyzer-dev --force
```

### Check Pod Status
```bash
kubectl get pods -n invoice-analyzer-dev
kubectl logs -n invoice-analyzer-dev <pod-name>
```

### Check External Secrets
```bash
kubectl get externalsecrets -n invoice-analyzer-dev
kubectl get secretstores -n invoice-analyzer-dev
kubectl describe externalsecret backend-secrets -n invoice-analyzer-dev
```

### Check Ingress
```bash
kubectl get ingress -n invoice-analyzer-dev
kubectl describe ingress invoice-analyzer -n invoice-analyzer-dev
```

## Route53 Configuration

After deploying the ingress, configure Route53:

1. Get the ALB DNS name:
```bash
kubectl get ingress invoice-analyzer -n invoice-analyzer-dev -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

2. Create an A record (Alias) in Route53 pointing to the ALB

## Maintenance

### Updating Application Version

1. Update the image tag in environment-specific deployment patches
2. Commit and push changes
3. ArgoCD will sync automatically

### Backup Database

The database backup PVC is created automatically. Set up a CronJob for regular backups:

```bash
# Example backup script (to be implemented)
kubectl exec -n invoice-analyzer-prod postgres-0 -- pg_dump -U postgres invoice_analyzer > backup.sql
```

## Contributing

1. Create a feature branch
2. Make changes
3. Test in dev environment
4. Create a pull request
5. After approval, merge to main
6. Changes will be automatically deployed to dev
7. Manually promote to prod if needed

## License

[Your License]

## Support

For issues and questions, please contact [Your Team]
