# Elasticsearch Cluster Helm Chart

A Helm chart for deploying Elasticsearch cluster using Elastic Cloud on Kubernetes (ECK) Operator on OpenShift.

## Description

This chart deploys an Elasticsearch cluster using the ECK Operator. The cluster is configured with a single node set that has both master and data roles.

## Prerequisites

### Required Operators

Before installing this chart, ensure the following operator is installed in your OpenShift cluster:

1. **Elastic Cloud on Kubernetes (ECK) Operator** - for Elasticsearch cluster deployment

You can install this operator from the OpenShift OperatorHub (Administrator → Operators → OperatorHub).

## Installation

### Using Helm CLI

```bash
helm install my-elasticsearch hackathon/elasticsearch-cluster -n elasticsearch --create-namespace
```

### Using OpenShift Console

1. Navigate to **Developer** perspective → **+Add** → **Helm Chart**
2. Search for "elasticsearch-cluster"
3. Configure the release and click **Install**

## Components Deployed

### 1. Elasticsearch Cluster
Elasticsearch cluster with configurable node count and resources.

**Default Configuration:**
- Name: `hackathon`
- Version: `9.2.0`
- Node count: 1
- Node roles: master, data
- TLS: Disabled (self-signed certificate disabled)
- Memory: 4Gi (request and limit)
- CPU: 1 (request), 2 (limit)
- Volume claim delete policy: DeleteOnScaledownAndClusterDeletion

### 2. OpenShift Route
Exposes the Elasticsearch cluster externally via OpenShift Route.

**Default Configuration:**
- Route name: `<cluster-name>-es`
- Service: `<cluster-name>-es-default`
- Port: http (9200)

## Configuration

### Configurable Values

| Parameter | Description | Default |
|-----------|-------------|---------|
| `elasticsearch.name` | Name of the Elasticsearch cluster | `hackathon` |
| `elasticsearch.version` | Elasticsearch version | `9.2.0` |
| `elasticsearch.nodeCount` | Number of nodes in the cluster | `1` |
| `elasticsearch.resources.requests.memory` | Memory request | `4Gi` |
| `elasticsearch.resources.requests.cpu` | CPU request | `1` |
| `elasticsearch.resources.limits.memory` | Memory limit | `4Gi` |
| `elasticsearch.resources.limits.cpu` | CPU limit | `2` |

### Example: Custom Configuration

```bash
helm install my-elasticsearch hackathon/elasticsearch-cluster \
  --set elasticsearch.name=my-es-cluster \
  --set elasticsearch.nodeCount=3 \
  --set elasticsearch.resources.limits.memory=8Gi \
  -n elasticsearch
```

## Accessing Elasticsearch

### From External (via OpenShift Route)

Get the route URL:

```bash
# Get the route URL
ROUTE_URL=$(oc get route hackathon-es -n elasticsearch -o jsonpath='{.spec.host}')
echo "Elasticsearch URL: http://$ROUTE_URL"

# Get the elastic user password
PASSWORD=$(oc get secret hackathon-es-elastic-user -n elasticsearch -o jsonpath='{.data.elastic}' | base64 -d)

# Test the connection
curl -u "elastic:$PASSWORD" http://$ROUTE_URL
```

### From Within the Cluster

Applications within the same namespace can connect to Elasticsearch using:

**HTTP endpoint (no TLS):**
```
http://hackathon-es-http:9200
```

### From Different Namespace

Use the fully qualified service name:

```
http://hackathon-es-http.elasticsearch.svc.cluster.local:9200
```

### Get Elastic User Password

The ECK operator creates a default `elastic` user with a generated password:

```bash
# Get the password
oc get secret hackathon-es-elastic-user -n elasticsearch -o jsonpath='{.data.elastic}' | base64 -d
```

### Test the Connection

```bash
# Get the password
PASSWORD=$(oc get secret hackathon-es-elastic-user -n elasticsearch -o jsonpath='{.data.elastic}' | base64 -d)

# Create a test pod to access Elasticsearch
oc run curl-test --image=curlimages/curl -i --rm --restart=Never -n elasticsearch -- \
  curl -u "elastic:$PASSWORD" http://hackathon-es-http:9200
```

## Verification

After installation, verify the Elasticsearch cluster is running:

```bash
# Check Elasticsearch cluster
oc get elasticsearch -n elasticsearch

# Check Elasticsearch pods
oc get pods -n elasticsearch -l elasticsearch.k8s.elastic.co/cluster-name=hackathon

# Check Elasticsearch services
oc get svc -n elasticsearch

# Check cluster health
PASSWORD=$(oc get secret hackathon-es-elastic-user -n elasticsearch -o jsonpath='{.data.elastic}' | base64 -d)
oc exec -n elasticsearch $(oc get pods -n elasticsearch -l elasticsearch.k8s.elastic.co/cluster-name=hackathon -o name | head -1) -- \
  curl -u "elastic:$PASSWORD" http://localhost:9200/_cluster/health?pretty
```

## Testing Elasticsearch

### Create an Index

```bash
# Get the password
PASSWORD=$(oc get secret hackathon-es-elastic-user -n elasticsearch -o jsonpath='{.data.elastic}' | base64 -d)

# Get Elasticsearch pod name
ES_POD=$(oc get pods -n elasticsearch -l elasticsearch.k8s.elastic.co/cluster-name=hackathon -o name | head -1)

# Create an index
oc exec -n elasticsearch $ES_POD -- \
  curl -u "elastic:$PASSWORD" -X PUT http://localhost:9200/test-index

# List indices
oc exec -n elasticsearch $ES_POD -- \
  curl -u "elastic:$PASSWORD" http://localhost:9200/_cat/indices?v
```

### Index and Search Documents

```bash
# Index a document
oc exec -n elasticsearch $ES_POD -- \
  curl -u "elastic:$PASSWORD" -X POST http://localhost:9200/test-index/_doc/1 \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello Elasticsearch", "timestamp": "2025-12-04T12:00:00"}'

# Search documents
oc exec -n elasticsearch $ES_POD -- \
  curl -u "elastic:$PASSWORD" http://localhost:9200/test-index/_search?pretty
```

## Uninstalling

```bash
helm uninstall my-elasticsearch -n elasticsearch
```

All resources (Elasticsearch cluster, pods, services) will be automatically deleted.

## Troubleshooting

### Elasticsearch Pods Not Starting

1. Check Elasticsearch status:
   ```bash
   oc get elasticsearch hackathon -n elasticsearch -o yaml
   ```

2. Check pod events:
   ```bash
   oc describe pods -n elasticsearch -l elasticsearch.k8s.elastic.co/cluster-name=hackathon
   ```

3. Check operator logs:
   ```bash
   oc logs -n openshift-operators deployment/elastic-operator
   ```

### Connection Issues

1. Verify Elasticsearch HTTP service:
   ```bash
   oc get svc hackathon-es-http -n elasticsearch
   ```

2. Check cluster health:
   ```bash
   oc get elasticsearch hackathon -n elasticsearch -o jsonpath='{.status.health}'
   ```

### Memory Issues

If pods are being OOMKilled, increase memory limits:

```bash
helm upgrade my-elasticsearch hackathon/elasticsearch-cluster \
  --set elasticsearch.resources.limits.memory=8Gi \
  --set elasticsearch.resources.requests.memory=8Gi \
  -n elasticsearch
```

## Support

For issues or questions about this chart, please contact the Hackathon Team.

## License

These charts are provided as-is for the Hackathon event.
