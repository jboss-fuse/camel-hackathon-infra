# Quarkus REST Server

This directory provides deployment resources for the Quarkus REST Server, a demonstration application exposing REST API endpoints with OpenAPI documentation.

## Overview

The Quarkus REST Server provides the following services:

- **Fruit CRUD Service**: In-memory create, read, and delete operations for fruit entities
- **Slow Service**: Demonstrates delayed responses with a 1-second artificial delay
- **Faulty Service**: Demonstrates error handling and exception mapping
- **Multipart Service**: File upload/download with binary data handling
- **OpenAPI/Swagger UI**: Interactive API documentation

## Deployment

### Prerequisites

- Access to an OpenShift cluster
- `oc` CLI installed and logged in
- Target namespace/project created

### Deploy via Direct YAML

Deploy the Quarkus REST server:

```bash
# Set your namespace
NAMESPACE=<your-namespace>

# Deploy the REST server
oc create -f quarkus-rest/quarkus-rest-server.yaml -n $NAMESPACE
```

### Verify Deployment

Check deployment status:

```bash
# Check pod status
oc get pods -l app=quarkus-rest-server -n $NAMESPACE

# Get the route URL
oc get route quarkus-rest-server -n $NAMESPACE -o jsonpath='{.spec.host}'
```

### Access the Services

Once deployed, retrieve the route URL:

```bash
ROUTE_URL=$(oc get route quarkus-rest-server -n $NAMESPACE -o jsonpath='{.spec.host}')
echo "REST Server URL: https://$ROUTE_URL"
```

Access the following endpoints:

- **OpenAPI Specification**: `https://$ROUTE_URL/q/openapi`
- **Swagger UI**: `https://$ROUTE_URL/q/swagger-ui`
- **Health Check**: `https://$ROUTE_URL/q/health`
- **Fruits API**: `https://$ROUTE_URL/fruits`

### Example API Calls

**List all fruits**:
```bash
curl https://$ROUTE_URL/fruits
```

**Add a new fruit**:
```bash
curl -X POST https://$ROUTE_URL/fruits \
  -H "Content-Type: application/json" \
  -d '{"name": "Banana", "description": "Tropical fruit"}'
```

**Delete a fruit**:
```bash
curl -X DELETE https://$ROUTE_URL/fruits \
  -H "Content-Type: application/json" \
  -d '{"name": "Banana", "description": "Tropical fruit"}'
```

**Test slow service**:
```bash
curl https://$ROUTE_URL/slow/World
# Returns: "Hello Slow World!" after 1-second delay
```

**Test faulty service**:
```bash
curl https://$ROUTE_URL/faulty/World
# Returns: 500 error with exception details
```

**Upload a file**:
```bash
curl -X POST https://$ROUTE_URL/multipart/upload \
  -F "file=@/path/to/file.txt"
```

**Download a file**:
```bash
curl https://$ROUTE_URL/multipart/download/{fileId} -o downloaded-file
```

## Configuration

The deployment uses the following defaults:

| Parameter | Value |
|-----------|-------|
| Container Image | `quay.io/fuse_qe/quarkus-rest-server:1.0.0` |
| HTTP Port | 8080 |
| Replicas | 1 |
| Memory Request | 512Mi |
| Memory Limit | 2Gi |
| CPU Request | 250m |
| CPU Limit | 1000m |
| Health Endpoints | `/q/health/live`, `/q/health/ready` |

## Uninstall

Remove the deployment:

```bash
oc delete -f quarkus-rest/quarkus-rest-server.yaml -n $NAMESPACE
```

## Building Custom Images

If you want to build a custom version of the REST server:

```bash
# Clone or navigate to the source repository
cd /path/to/quarkus-rest-server

# Build the container image
mvn clean package \
  -Dquarkus.container-image.build=true \
  -Dquarkus.container-image.group=<your-registry> \
  -Dquarkus.container-image.name=quarkus-rest-server \
  -Dquarkus.container-image.tag=<your-tag>

# Push to your registry
docker push <your-registry>/quarkus-rest-server:<your-tag>

# Update the image reference in quarkus-rest-server.yaml
```

## Technical Details

- **Framework**: Quarkus 3.30.0
- **Runtime**: JVM mode (Java 17)
- **Build Tool**: Maven
- **Dependencies**:
  - quarkus-rest
  - quarkus-rest-jackson
  - quarkus-smallrye-openapi
- **Health Checks**: Quarkus SmallRye Health (`/q/health/live`, `/q/health/ready`)
- **OpenAPI Version**: OpenAPI 3.0
