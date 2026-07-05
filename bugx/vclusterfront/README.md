# vclusterfront Helm Chart

This Helm chart deploys the vclusterfront application to a Kubernetes cluster.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- Docker image for vclusterfront

## Installation

### Build and push Docker image

```bash
# Build the image
docker build -t vclusterfront:latest .

# Tag for your registry (replace with your registry)
docker tag vclusterfront:latest your-registry/vclusterfront:latest

# Push to registry
docker push your-registry/vclusterfront:latest
```

### Install the chart

```bash
# Install with default values
helm install vclusterfront ./helm/vclusterfront

# Install with custom API URLs
helm install vclusterfront ./helm/vclusterfront \
  --set configMap.data.VITE_API_BASE_URL=http://api-service:8082 \
  --set configMap.data.VITE_VAULT_API_URL=http://vault-service:8080/api/v1

# Install with custom image
helm install vclusterfront ./helm/vclusterfront \
  --set image.repository=your-registry/vclusterfront \
  --set image.tag=v1.0.0
```

## Configuration

The following table lists the configurable parameters and their default values:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `2` |
| `image.repository` | Image repository | `vclusterfront` |
| `image.tag` | Image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `ingress.enabled` | Enable ingress | `false` |
| `configMap.data.VITE_API_BASE_URL` | Main API base URL | `http://api-service:8082` |
| `configMap.data.VITE_VAULT_API_URL` | Vault API base URL | `http://vault-service:8080/api/v1` |
| `resources.limits.cpu` | CPU limit | `500m` |
| `resources.limits.memory` | Memory limit | `512Mi` |
| `resources.requests.cpu` | CPU request | `100m` |
| `resources.requests.memory` | Memory request | `128Mi` |
| `autoscaling.enabled` | Enable HPA | `false` |
| `autoscaling.minReplicas` | Minimum replicas | `2` |
| `autoscaling.maxReplicas` | Maximum replicas | `10` |

## Upgrading

```bash
helm upgrade vclusterfront ./helm/vclusterfront \
  --set configMap.data.VITE_API_BASE_URL=http://new-api-service:8082
```

## Uninstalling

```bash
helm uninstall vclusterfront
```

## Notes

- The ConfigMap is automatically created and mounted as environment variables
- Environment variables are injected at build time for Vite applications
- Make sure your API services are accessible from the pods
- For production, use proper image tags instead of `latest`

