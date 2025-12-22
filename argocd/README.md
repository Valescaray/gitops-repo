# ArgoCD Applications

This directory contains ArgoCD Application manifests for deploying the Invoice Analyzer application to different environments.

## Applications

### invoice-analyzer-dev
- **Environment**: Development
- **Namespace**: invoice-analyzer-dev
- **Auto-sync**: Enabled with auto-prune
- **Path**: environments/dev

### invoice-analyzer-prod
- **Environment**: Production
- **Namespace**: invoice-analyzer-prod
- **Auto-sync**: Enabled (manual prune for safety)
- **Path**: environments/prod

### monitoring
- **Purpose**: Monitoring namespace creation
- **Namespace**: monitoring
- **Auto-sync**: Enabled
- **Path**: namespaces

## Deployment

To deploy all applications:
```bash
kubectl apply -f argocd/applications/
```

To deploy a specific application:
```bash
kubectl apply -f argocd/applications/invoice-analyzer-dev.yaml
```

## Monitoring Applications

Check application status:
```bash
argocd app list
argocd app get invoice-analyzer-dev
```

View application details in ArgoCD UI:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Access at https://localhost:8080
```

## Sync Policies

### Development
- **Auto-sync**: Enabled
- **Auto-prune**: Enabled (removes resources deleted from Git)
- **Self-heal**: Enabled (reverts manual changes)

### Production
- **Auto-sync**: Enabled
- **Auto-prune**: Disabled (manual approval required)
- **Self-heal**: Enabled

## Customization

To modify sync policies or add new applications:
1. Edit the appropriate YAML file
2. Update the `repoURL` to point to your Git repository
3. Apply the changes: `kubectl apply -f argocd/applications/`
