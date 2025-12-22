# Configuration Checklist

Before deploying the Invoice Analyzer application, ensure you have configured the following:

## AWS Resources

### ACM Certificates
- [ ] Dev environment certificate created
- [ ] Prod environment certificate created
- [ ] Certificate ARNs updated in ingress manifests

### Route53
- [ ] Hosted zone created
- [ ] Domain name configured
- [ ] NS records updated with domain registrar

### IAM Roles (IRSA)
- [ ] Backend service account role created
  - [ ] S3 access policy attached
  - [ ] Role ARN updated in backend serviceaccount.yaml
- [ ] External Secrets Operator role created
  - [ ] Secrets Manager access policy attached
  - [ ] Role ARN updated in external-secrets serviceaccount.yaml

### Security Groups
- [ ] ALB security group created
- [ ] Ingress rules configured (80, 443)
- [ ] Security group IDs updated in ingress manifests

### AWS Secrets Manager
- [ ] `invoice-analyzer/backend` secret created
  - [ ] database-url
  - [ ] aws-access-key-id
  - [ ] aws-secret-access-key
  - [ ] openai-api-key
- [ ] `invoice-analyzer/database` secret created
  - [ ] username
  - [ ] password
- [ ] `invoice-analyzer/grafana` secret created
  - [ ] admin-user
  - [ ] admin-password

### S3 Buckets
- [ ] Dev uploads bucket created
- [ ] Prod uploads bucket created
- [ ] Bucket policies configured

### WAF (Production only)
- [ ] WAF ACL created
- [ ] WAF ARN updated in prod ingress manifest

## Container Registry

- [ ] ECR repositories created
  - [ ] invoice-analyzer-frontend
  - [ ] invoice-analyzer-backend
- [ ] Registry URL updated in deployment manifests
- [ ] Image pull secrets configured (if using private registry)

## Kubernetes Cluster

### Operators Installed
- [ ] ArgoCD
- [ ] AWS Load Balancer Controller
- [ ] Prometheus Operator
- [ ] External Secrets Operator

### Storage Classes
- [ ] gp3 storage class available
- [ ] Default storage class configured

## Git Repository

- [ ] Repository created
- [ ] Repository URL updated in ArgoCD applications
- [ ] Deploy keys or credentials configured

## Manifest Updates

### Global Replacements
- [ ] `<REGISTRY>` → Your container registry URL
- [ ] `<AWS_ACCOUNT_ID>` → Your AWS account ID
- [ ] `<YOUR_ORG>` → Your GitHub org/username

### Dev Environment
- [ ] `<DEV_CERTIFICATE_ARN>` → Dev ACM certificate ARN
- [ ] `<DEV_SECURITY_GROUP_ID>` → Dev security group ID
- [ ] Dev domain name updated

### Prod Environment
- [ ] `<PROD_CERTIFICATE_ARN>` → Prod ACM certificate ARN
- [ ] `<PROD_SECURITY_GROUP_ID>` → Prod security group ID
- [ ] `<WAF_ACL_ARN>` → WAF ACL ARN
- [ ] Prod domain name updated

## Verification

After deployment:
- [ ] All pods are running
- [ ] External secrets are synced
- [ ] Ingress has been assigned an ALB
- [ ] Route53 records point to ALB
- [ ] Application is accessible via domain
- [ ] Grafana is accessible
- [ ] Prometheus is collecting metrics
- [ ] Network policies are enforced

## Commands for Verification

```bash
# Check pods
kubectl get pods -n invoice-analyzer-dev
kubectl get pods -n invoice-analyzer-prod
kubectl get pods -n monitoring

# Check external secrets
kubectl get externalsecrets -A
kubectl get secretstores -A

# Check ingress
kubectl get ingress -A

# Check ArgoCD applications
argocd app list
argocd app get invoice-analyzer-dev

# Check Prometheus targets
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090
# Visit http://localhost:9090/targets

# Check Grafana
kubectl port-forward -n monitoring svc/grafana 3000:3000
# Visit http://localhost:3000
```
