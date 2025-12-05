# MinIO Helm Chart

This Helm chart deploys MinIO object storage on OpenShift.

## Description

MinIO is a high-performance, S3-compatible object storage system. This chart deploys a single-node MinIO instance with persistent storage and exposes both the API and WebUI through OpenShift routes.

## Prerequisites

- OpenShift cluster
- Helm 3.x

## Installation

### Basic Installation

```bash
helm install minio hackathon/minio -n <namespace>
```

### Custom Installation

```bash
# Custom instance name and credentials
helm install my-storage hackathon/minio \
  --set minio.name=my-minio \
  --set minio.credentials.accessKey=admin \
  --set minio.credentials.secretKey=admin123 \
  -n <namespace>

# Custom storage size
helm install minio hackathon/minio \
  --set minio.persistence.size=10Gi \
  -n <namespace>

# With TLS on route
helm install minio hackathon/minio \
  --set minio.route.tls.enabled=true \
  -n <namespace>
```

## Configuration

### Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `minio.name` | Instance name | `minio` |
| `minio.replicas` | Number of replicas | `1` |
| `minio.image.repository` | MinIO image repository | `quay.io/minio/minio` |
| `minio.image.tag` | MinIO image tag | `latest` |
| `minio.credentials.accessKey` | MinIO access key | `minio` |
| `minio.credentials.secretKey` | MinIO secret key | `minio123` |
| `minio.persistence.enabled` | Enable persistent storage | `true` |
| `minio.persistence.size` | Storage size | `5Gi` |
| `minio.service.apiPort` | MinIO API port | `9000` |
| `minio.service.webuiPort` | MinIO WebUI port | `36491` |
| `minio.route.enabled` | Create OpenShift route for WebUI | `true` |
| `minio.route.tls.enabled` | Enable TLS on route | `false` |
| `minio.resources.requests.memory` | Memory request | `512Mi` |
| `minio.resources.requests.cpu` | CPU request | `250m` |
| `minio.resources.limits.memory` | Memory limit | `2Gi` |
| `minio.resources.limits.cpu` | CPU limit | `1000m` |

## Accessing MinIO

### Get WebUI Route URL

```bash
# Get the WebUI route URL
MINIO_URL=$(oc get route <minio-name>-webui -o jsonpath='{.spec.host}')
echo "MinIO WebUI: http://$MINIO_URL"
```

### Access from Command Line

```bash
# Install MinIO client
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc

# Configure MinIO client
MINIO_URL=$(oc get route <minio-name>-webui -o jsonpath='{.spec.host}')
./mc alias set myminio http://$MINIO_URL minio minio123

# List buckets
./mc ls myminio

# Create a bucket
./mc mb myminio/mybucket

# Upload a file
./mc cp myfile.txt myminio/mybucket/
```

## Resources Created

This chart creates the following Kubernetes/OpenShift resources:

- **Secret**: Stores MinIO credentials
- **PersistentVolumeClaim**: Storage for MinIO data
- **Service**: Exposes MinIO API and WebUI ports
- **Deployment**: Runs the MinIO server
- **Route**: OpenShift route for WebUI access (HTTP)

## Notes

- The WebUI route is configured for HTTP by default. Set `minio.route.tls.enabled=true` for HTTPS.
- Default credentials are `minio`/`minio123` - change these for production use.
- The chart creates a single-node MinIO deployment. For production, consider using MinIO in distributed mode.
- Storage class can be specified using `minio.persistence.storageClass` parameter.

## Uninstallation

```bash
helm uninstall minio -n <namespace>
```

Note: PersistentVolumeClaim may need to be deleted manually if you want to remove all data.
