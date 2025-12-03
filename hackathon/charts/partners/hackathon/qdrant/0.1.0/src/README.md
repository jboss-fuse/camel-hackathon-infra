# Qdrant Helm Chart

This Helm chart deploys Qdrant vector database on OpenShift.

## Overview

Qdrant is a high-performance vector database designed for similarity search and AI applications. This chart deploys Qdrant using the unprivileged container image suitable for OpenShift environments.

## Prerequisites

- OpenShift cluster 4.x or later
- Helm 3.x or later
- Persistent storage available in the cluster (if storage is enabled)

## Installation

### Using Helm Repository

```bash
helm repo add hackathon https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
helm repo update
helm install qdrant hackathon/qdrant -n <namespace>
```

### Customize Installation

```bash
# Custom instance name
helm install my-vector-db hackathon/qdrant --set qdrant.name=my-qdrant -n <namespace>

# Disable persistent storage (ephemeral)
helm install qdrant hackathon/qdrant --set qdrant.storage.enabled=false -n <namespace>

# Custom storage size
helm install qdrant hackathon/qdrant --set qdrant.storage.size=10Gi -n <namespace>

# Custom resource limits
helm install qdrant hackathon/qdrant \
  --set qdrant.resources.limits.memory=4Gi \
  --set qdrant.resources.limits.cpu=2000m \
  -n <namespace>
```

## Configuration

The following table lists the configurable parameters:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `qdrant.name` | Name of the Qdrant instance | `qdrant` |
| `qdrant.image` | Docker image to use | `qdrant/qdrant:v1.15.1-unprivileged` |
| `qdrant.replicas` | Number of replicas | `1` |
| `qdrant.port` | HTTP API port | `6333` |
| `qdrant.grpcPort` | gRPC API port | `6334` |
| `qdrant.resources.limits.memory` | Memory limit | `2Gi` |
| `qdrant.resources.limits.cpu` | CPU limit | `1000m` |
| `qdrant.resources.requests.memory` | Memory request | `512Mi` |
| `qdrant.resources.requests.cpu` | CPU request | `250m` |
| `qdrant.storage.enabled` | Enable persistent storage | `true` |
| `qdrant.storage.size` | Storage size | `5Gi` |

## Accessing Qdrant

After deployment, you can access Qdrant via:

### Get Route URL

```bash
ROUTE_URL=$(oc get route qdrant -o jsonpath='{.spec.host}')
echo "Qdrant UI: https://$ROUTE_URL/dashboard"
echo "Qdrant API: https://$ROUTE_URL"
```

### Web UI

Access the Qdrant dashboard at: `https://<route-url>/dashboard`

### REST API

```bash
# Check cluster status
curl https://$ROUTE_URL/

# List collections
curl https://$ROUTE_URL/collections

# Create a collection
curl -X PUT https://$ROUTE_URL/collections/test_collection \
  -H "Content-Type: application/json" \
  -d '{
    "vectors": {
      "size": 4,
      "distance": "Dot"
    }
  }'
```

### gRPC API

For gRPC access from within the cluster:
```
qdrant:6334
```

## Health Checks

The deployment includes liveness and readiness probes:

- Liveness probe: HTTP GET on port 6333, path `/`
- Readiness probe: HTTP GET on port 6333, path `/`

## Storage

By default, the chart creates a PersistentVolumeClaim of 5Gi. To disable persistent storage:

```bash
helm install qdrant hackathon/qdrant --set qdrant.storage.enabled=false -n <namespace>
```

**Warning**: Disabling persistent storage means all data will be lost when the pod restarts.

## Uninstallation

```bash
helm uninstall qdrant -n <namespace>
```

Note: The PersistentVolumeClaim is not automatically deleted. To remove it:

```bash
oc delete pvc qdrant-pvc -n <namespace>
```

## Resources

- [Qdrant Documentation](https://qdrant.tech/documentation/)
- [Qdrant API Reference](https://qdrant.tech/documentation/interfaces/)
- [Qdrant GitHub](https://github.com/qdrant/qdrant)
