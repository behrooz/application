# Redis Single Node - Important Values

This document lists the most important values needed to configure Redis as a **single node standalone** deployment.

## Critical Configuration

### 1. Architecture (MOST IMPORTANT)
```yaml
architecture: standalone
```
**Required:** Set to `standalone` for single node Redis. Default is `replication` which creates master-replica setup.

### 2. Authentication
```yaml
auth:
  enabled: true
  password: "your-secure-password"
  sentinel: false  # Not needed for standalone
```

### 3. Image Configuration
```yaml
image:
  registry: "172.16.16.1:30516"  # Your registry
  repository: "bitnami/redis"
  tag: "7.2.4-debian-11-r0"
```

### 4. Master Persistence (Data Storage)
```yaml
master:
  persistence:
    enabled: true
    storageClass: "longhorn"  # or your preferred storage class
    size: 8Gi                 # Adjust based on your needs
    path: /data
    accessModes:
      - ReadWriteOnce
```

### 5. Master Resources
```yaml
master:
  resources:
    requests:
      cpu: 200m
      memory: 256Mi
    limits:
      cpu: 500m
      memory: 512Mi
```

### 6. Service Type
```yaml
auth:
  service:
    type: ClusterIP  # Use ClusterIP for internal access
```

## Complete Minimal Configuration Example

```yaml
# Global settings
global:
  redis:
    password: "your-secure-password"

# Image settings
image:
  registry: "172.16.16.1:30516"
  repository: "bitnami/redis"
  tag: "7.2.4-debian-11-r0"

# Architecture - CRITICAL for single node
architecture: standalone

# Authentication
auth:
  enabled: true
  password: "your-secure-password"
  sentinel: false
  service:
    type: ClusterIP

# Master configuration
master:
  # Persistence
  persistence:
    enabled: true
    storageClass: "longhorn"  # Change to your storage class
    size: 8Gi
    path: /data
    accessModes:
      - ReadWriteOnce
  
  # Resources
  resources:
    requests:
      cpu: 200m
      memory: 256Mi
    limits:
      cpu: 500m
      memory: 512Mi
```

## Key Points

1. **`architecture: standalone`** - This is the most critical setting. Without it, Redis will deploy in replication mode with master and replicas.

2. **Persistence** - Enable persistence if you want data to survive pod restarts. Set `master.persistence.enabled: true` and specify a storage class.

3. **No Replica Configuration Needed** - When using `standalone` architecture, you don't need to configure the `replica:` section.

4. **Service Type** - Use `ClusterIP` for internal access within the cluster. NodePort and LoadBalancer are not needed for single node.

5. **Password** - Always set a password for production use.

## Optional but Recommended

```yaml
# Common configuration
commonConfiguration: |-
  appendonly yes
  save ""

# Node selector (if you want to pin to specific node)
nodeSelector:
  node.name: node-01
```

## What NOT to Configure

- **`replica:`** section - Not needed for standalone
- **`sentinel:`** section - Not needed for standalone
- **`architecture: replication`** - This creates master-replica setup

