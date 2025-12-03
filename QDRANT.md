# Qdrant Vector Database

Qdrant is a high-performance vector database designed for similarity search and AI applications. This deployment uses ephemeral storage and is optimized for OpenShift environments.

## Quick Start

### Deploy Using Direct YAML

```bash
oc create -f qdrant/qdrant.yaml -n <namespace>
```

### Deploy Using Helm

```bash
# Install from local chart
helm install qdrant hackathon/qdrant-0.1.0.tgz -n <namespace>

# Or from repository (after adding the repo)
helm repo add hackathon https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
helm install qdrant hackathon/qdrant -n <namespace>
```

## Accessing Qdrant

Get the route URL:

```bash
ROUTE_URL=$(oc get route qdrant -o jsonpath='{.spec.host}')
echo "Qdrant API: https://$ROUTE_URL"
echo "Qdrant Dashboard: https://$ROUTE_URL/dashboard"
```

### Web Dashboard

Qdrant includes a built-in web dashboard for visual management of collections, points, and search operations. Access it at:

```
https://<route-url>/dashboard
```

The dashboard provides:
- Interactive collection browser
- Visual point exploration and filtering
- Real-time search interface
- Collection schema visualization
- Cluster status monitoring
