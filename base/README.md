# Base Manifests

This directory contains the base Kubernetes manifests for all components of the Invoice Analyzer application. These manifests are environment-agnostic and are overlaid by environment-specific configurations.

## Components

### Frontend
- **Deployment**: React application with health checks
- **Service**: ClusterIP service
- **ConfigMap**: Application configuration
- **HPA**: Horizontal Pod Autoscaler

### Backend
- **Deployment**: FastAPI application with Prometheus metrics
- **Service**: ClusterIP service
- **ConfigMap**: Application configuration
- **ExternalSecret**: AWS Secrets Manager integration
- **ServiceAccount**: IRSA-enabled for AWS access
- **ServiceMonitor**: Prometheus metrics collection
- **HPA**: Advanced horizontal scaling

### Database
- **StatefulSet**: PostgreSQL with persistent storage
- **Service**: Headless and load-balanced services
- **ConfigMap**: Database configuration and init scripts
- **ExternalSecret**: Database credentials
- **PVC**: Backup storage
- **PodMonitor**: Database metrics

### Ingress
- **Ingress**: AWS ALB configuration with SSL

### Monitoring

#### Prometheus Operator
- **Prometheus CR**: Prometheus instance with service discovery
- **ServiceAccount**: RBAC for metrics collection
- **ConfigMap**: Additional scrape configurations
- **PrometheusRule**: Alerting rules

#### Grafana
- **Deployment**: Grafana with dashboard provisioning
- **Service**: ClusterIP service
- **PVC**: Persistent storage
- **ExternalSecret**: Admin credentials
- **ConfigMap**: Datasources and dashboards
- **Ingress**: Public access

### RBAC
- **ServiceAccounts**: For application components
- **Roles**: Least-privilege access
- **RoleBindings**: Service account bindings

### Network Policies
- **Frontend**: Allow ingress from ALB, egress to backend
- **Backend**: Allow ingress from frontend, egress to database
- **Database**: Allow ingress from backend only

### External Secrets
- **SecretStore**: AWS Secrets Manager configuration
- **ServiceAccount**: IRSA for Secrets Manager access

## Usage

These base manifests are not meant to be applied directly. They should be referenced by environment-specific kustomizations in the `environments/` directory.

Example kustomization.yaml:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
  - ../../base/frontend

patchesStrategicMerge:
  - deployment.yaml
```

## Customization

To modify base configurations:
1. Edit the appropriate manifest in this directory
2. Changes will propagate to all environments
3. Environment-specific overrides can be added in environment overlays
