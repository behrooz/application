# vclusterapi Helm Chart

This Helm chart deploys the vclusterapi application on a Kubernetes cluster.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- MongoDB (external or internal)
- Auth Service (running separately)

## Installation

### Quick Start

```bash
# Add your chart repository (if applicable)
helm repo add vclusterapi ./helm/vclusterapi

# Install with default values
helm install vclusterapi ./helm/vclusterapi

# Install with custom values
helm install vclusterapi ./helm/vclusterapi -f my-values.yaml

# Install in specific namespace
helm install vclusterapi ./helm/vclusterapi -n vclusterapi --create-namespace
```

### Upgrade

```bash
helm upgrade vclusterapi ./helm/vclusterapi

# With custom values
helm upgrade vclusterapi ./helm/vclusterapi -f my-values.yaml
```

### Uninstall

```bash
helm uninstall vclusterapi
```

## Configuration

The following table lists the configurable parameters and their default values:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `2` |
| `image.repository` | Image repository | `vclusterapi` |
| `image.tag` | Image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `8082` |
| `ingress.enabled` | Enable ingress | `false` |
| `mongodb.connectionString` | MongoDB connection string | `mongodb://root:secret123@212.64.215.155:32169/` |
| `mongodb.database` | MongoDB database name | `vcluster` |
| `authService.url` | Auth service URL | `http://localhost:8083` |
| `argocd.server` | ArgoCD server address | `argocd.bugx.ir:443` |
| `argocd.token` | ArgoCD token (use secret in production) | `""` |
| `argocd.existingSecret` | Existing secret for ArgoCD token | `""` |
| `git.defaultRepoUrl` | Default Git repository URL | `https://github.com/behrooz/application` |
| `git.defaultRepoPath` | Default Git repository path | `vcluster-0.28.0` |
| `domain.baseDomain` | Base domain for ingress | `bugx.ir` |
| `tunnel.baseDomain` | Base domain for tunnels | `bugx.ir` |
| `tunnel.portRangeMin` | Tunnel port range minimum | `30000` |
| `tunnel.portRangeMax` | Tunnel port range maximum | `31000` |
| `storage.defaultStorageClass` | Default storage class | `local-path` |
| `storage.defaultStorageSize` | Default storage size | `5Gi` |
| `hostCluster.name` | Host cluster name | `host-cluster-1` |
| `cors.allowedOrigins` | CORS allowed origins | `*` |
| `server.port` | Server port | `8082` |
| `server.host` | Server host | `0.0.0.0` |
| `resources.limits.cpu` | CPU limit | `1000m` |
| `resources.limits.memory` | Memory limit | `1Gi` |
| `resources.requests.cpu` | CPU request | `500m` |
| `resources.requests.memory` | Memory request | `512Mi` |
| `autoscaling.enabled` | Enable HPA | `false` |
| `autoscaling.minReplicas` | Minimum replicas | `2` |
| `autoscaling.maxReplicas` | Maximum replicas | `10` |

## Examples

### Basic Installation

```bash
helm install vclusterapi ./helm/vclusterapi
```

### With Custom MongoDB

```yaml
# custom-values.yaml
mongodb:
  connectionString: "mongodb://user:password@mongodb.example.com:27017/"
  database: "vcluster"
```

```bash
helm install vclusterapi ./helm/vclusterapi -f custom-values.yaml
```

### With Ingress

```yaml
# ingress-values.yaml
ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: vclusterapi-tls
      hosts:
        - api.example.com
```

```bash
helm install vclusterapi ./helm/vclusterapi -f ingress-values.yaml
```

### With Autoscaling

```yaml
# autoscaling-values.yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80
```

```bash
helm install vclusterapi ./helm/vclusterapi -f autoscaling-values.yaml
```

### With Secrets

```yaml
# secrets-values.yaml
secrets:
  enabled: true
  mongodbPassword: "secure-password"

mongodb:
  existingSecret: "mongodb-secret"
  secretKey: "password"

argocd:
  existingSecret: "argocd-secret"
  secretKey: "argocd-token"
```

```bash
# Create secrets first
kubectl create secret generic mongodb-secret \
  --from-literal=password='your-password'

kubectl create secret generic argocd-secret \
  --from-literal=argocd-token='your-argocd-token'

# Install with secrets
helm install vclusterapi ./helm/vclusterapi -f secrets-values.yaml
```

### With Custom Domain

```yaml
# domain-values.yaml
domain:
  baseDomain: "example.com"

tunnel:
  baseDomain: "example.com"

argocd:
  server: "argocd.example.com:443"
```

```bash
helm install vclusterapi ./helm/vclusterapi -f domain-values.yaml
```

## Environment Variables

You can add custom environment variables:

```yaml
env:
  - name: LOG_LEVEL
    value: "debug"
  - name: CUSTOM_CONFIG
    value: "custom-value"
```

## Health Checks

The chart includes liveness and readiness probes. Make sure your application has a `/health` endpoint, or update the probe paths in `values.yaml`:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 10
  periodSeconds: 5
```

## Notes

- The application requires MongoDB to be accessible
- The application requires Auth Service to be accessible
- Update MongoDB connection string and Auth Service URL in `values.yaml` before installation
- For production, use secrets for sensitive data instead of plain text values
- Consider enabling ingress for external access
- Enable autoscaling for production workloads

## Troubleshooting

### Check Pod Status

```bash
kubectl get pods -l app.kubernetes.io/name=vclusterapi
```

### View Logs

```bash
kubectl logs -l app.kubernetes.io/name=vclusterapi
```

### Check Service

```bash
kubectl get svc -l app.kubernetes.io/name=vclusterapi
```

### Describe Pod

```bash
kubectl describe pod <pod-name>
```

### Test Connectivity

```bash
# Port forward to test locally
kubectl port-forward svc/vclusterapi 8082:8082

# Test endpoint
curl http://localhost:8082/health
```

