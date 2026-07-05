# Quick Start Guide

## Prerequisites

1. Kubernetes cluster running
2. Helm 3.x installed
3. MongoDB accessible (or configure connection)
4. Auth Service running (or configure URL)

## Installation Steps

### 1. Build and Push Docker Image (if needed)

```bash
# Build the image
docker build -t vclusterapi:latest .

# Tag for your registry
docker tag vclusterapi:latest your-registry/vclusterapi:latest

# Push to registry
docker push your-registry/vclusterapi:latest
```

### 2. Update values.yaml

Edit `helm/vclusterapi/values.yaml`:

```yaml
image:
  repository: your-registry/vclusterapi
  tag: "latest"

mongodb:
  connectionString: "mongodb://user:password@mongodb-host:27017/"

authService:
  url: "http://auth-service:8083"
```

### 3. Install the Chart

```bash
# Install with default values
helm install vclusterapi ./helm/vclusterapi

# Or install with custom values file
helm install vclusterapi ./helm/vclusterapi -f my-custom-values.yaml

# Install in specific namespace
helm install vclusterapi ./helm/vclusterapi -n vclusterapi --create-namespace
```

### 4. Verify Installation

```bash
# Check pods
kubectl get pods -l app.kubernetes.io/name=vclusterapi

# Check service
kubectl get svc -l app.kubernetes.io/name=vclusterapi

# View logs
kubectl logs -l app.kubernetes.io/name=vclusterapi
```

### 5. Access the API

```bash
# Port forward
kubectl port-forward svc/vclusterapi 8082:8082

# Test
curl http://localhost:8082/health
```

## Common Customizations

### Enable Ingress

```yaml
ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix
```

### Enable Autoscaling

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

### Use Secrets for MongoDB

```yaml
mongodb:
  existingSecret: "mongodb-secret"
  secretKey: "password"

# Create secret first
kubectl create secret generic mongodb-secret \
  --from-literal=password='your-password'
```

## Troubleshooting

### Pods Not Starting

```bash
# Check pod status
kubectl describe pod <pod-name>

# Check events
kubectl get events --sort-by='.lastTimestamp'
```

### Connection Issues

```bash
# Test MongoDB connectivity from pod
kubectl exec -it <pod-name> -- sh
# Inside pod, test connection
```

### Auth Service Issues

```bash
# Verify auth service is accessible
kubectl exec -it <pod-name> -- curl http://auth-service:8083/health
```

