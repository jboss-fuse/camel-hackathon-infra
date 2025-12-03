# REST API Server

This document provides instructions for deploying and using the Quarkus REST API Server for the Camel Hackathon.

## Overview

The REST API Server is a Quarkus-based application that acts as an HTTP proxy for legacy applications. The primary use case is to provide TLS/HTTPS support for legacy in-house applications that broke when third-party services started requiring secure connections.

**Use Case**: Act as an HTTP proxy for a very legacy in-house application that broke when a third-party app started requiring TLS.

**Source Code**: https://github.com/Croway/quarkus-rest-server

The server provides multiple REST endpoints demonstrating various integration patterns:
- **Fruit API**: CRUD operations for managing fruit entities
- **Slow Service**: Simulates delayed responses (1-second delay)
- **Faulty Service**: Demonstrates error handling and exception scenarios
- **Multipart Service**: File upload and download capabilities
- **OpenAPI/Swagger UI**: Interactive API documentation

## Installation

### Prerequisites

Ensure you have:
- Access to an OpenShift cluster
- `oc` CLI installed and configured
- `helm` CLI installed (for Helm installation method)

### Option 1: Install via Helm Chart (Recommended)

#### Step 1: Add the Helm Repository

```bash
helm repo add hackathon https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
helm repo update
```

#### Step 2: Install REST API Server

Choose your namespace and install:

```bash
# Set your namespace
NAMESPACE=<your-namespace>

# Install with default configuration
helm install quarkus-rest hackathon/quarkus-rest-server -n $NAMESPACE

# Or install with custom name
helm install quarkus-rest hackathon/quarkus-rest-server \
  --set restServer.name=my-rest-api \
  -n $NAMESPACE

# Or install with custom resource limits
helm install quarkus-rest hackathon/quarkus-rest-server \
  --set restServer.resources.limits.memory=4Gi \
  --set restServer.resources.limits.cpu=2000m \
  -n $NAMESPACE
```

#### Step 3: Verify Installation

```bash
# Check if pods are running
oc get pods -l app=quarkus-rest-server -n $NAMESPACE

# Check if route is created
oc get routes -n $NAMESPACE | grep quarkus-rest
```

### Option 2: Install via Direct YAML Manifests

```bash
# Set your namespace
NAMESPACE=<your-namespace>

# Deploy the REST server
oc create -f quarkus-rest/quarkus-rest-server.yaml -n $NAMESPACE
```

## Retrieving Server URL

After installation, retrieve the route URL:

```bash
# Set your namespace
NAMESPACE=<your-namespace>

# Get the REST API Server route
REST_URL=$(oc get route quarkus-rest-server -n $NAMESPACE -o jsonpath='{.spec.host}')
echo "REST API Server URL: https://$REST_URL"
```

## Available Endpoints

Once deployed, you can access the following endpoints:

| Endpoint | Description | Method |
|----------|-------------|--------|
| `/openapi` | OpenAPI specification | GET |
| `/q/swagger-ui` | Interactive Swagger UI | GET |
| `/q/health` | Health check endpoint | GET |
| `/fruits` | List all fruits | GET |
| `/fruits` | Add a new fruit | POST |
| `/fruits` | Delete a fruit | DELETE |
| `/slow/{text}` | Slow service (1s delay) | GET |
| `/faulty/{text}` | Faulty service (throws exception) | GET |
| `/multipart/upload` | Upload a file | POST |
| `/multipart/download/{fileId}` | Download a file | GET |

## Usage Examples

### Access OpenAPI Documentation

```bash
# Get route URL
REST_URL=$(oc get route quarkus-rest-server -n $NAMESPACE -o jsonpath='{.spec.host}')

# Access OpenAPI specification
curl https://$REST_URL/openapi

# Access Swagger UI in browser
echo "Swagger UI: https://$REST_URL/q/swagger-ui"
```

**Example OpenAPI URL**:
```
https://quarkus-rest-server-camel-hackathon.camel-dev-eu-de-1-bx2-4x1-b0521fcfdb6f3f868b0758fbda095bbe-0000.eu-de.containers.appdomain.cloud/openapi
```

### Fruit CRUD Operations

**List all fruits**:
```bash
curl https://$REST_URL/fruits
```

**Add a new fruit**:
```bash
curl -X POST https://$REST_URL/fruits \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Banana",
    "description": "A yellow tropical fruit"
  }'
```

**Delete a fruit**:
```bash
curl -X DELETE https://$REST_URL/fruits \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Banana",
    "description": "A yellow tropical fruit"
  }'
```

### Test Slow Service

This endpoint simulates a delayed response (1-second delay):

```bash
curl https://$REST_URL/slow/World
# Returns: "Hello Slow World!" after 1 second
```

### Test Faulty Service

This endpoint demonstrates error handling by intentionally throwing an exception:

```bash
curl https://$REST_URL/faulty/World
# Returns: 500 Internal Server Error with exception details
```

### File Upload and Download

**Upload a file**:
```bash
curl -X POST https://$REST_URL/multipart/upload \
  -F "file=@/path/to/your/file.txt"
```

**Download a file**:
```bash
curl https://$REST_URL/multipart/download/{fileId} -o downloaded-file.txt
```

## Configuration

The deployment uses the following default configuration:

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

### Customizing the Deployment

You can customize the deployment using Helm values:

```bash
# Custom instance name
helm install my-rest hackathon/quarkus-rest-server \
  --set restServer.name=my-custom-rest-api \
  -n $NAMESPACE

# Custom resource limits
helm install my-rest hackathon/quarkus-rest-server \
  --set restServer.resources.limits.memory=4Gi \
  --set restServer.resources.limits.cpu=2000m \
  -n $NAMESPACE

# Custom replica count
helm install my-rest hackathon/quarkus-rest-server \
  --set restServer.replicas=3 \
  -n $NAMESPACE
```

## Integration with Apache Camel

The REST API Server can be easily integrated with Apache Camel routes:

### HTTP Proxy Example

```java
// Camel route acting as an HTTP proxy to the REST API
from("direct:callRestApi")
    .setHeader(Exchange.HTTP_METHOD, constant("GET"))
    .to("https://" + restServerUrl + "/fruits")
    .convertBodyTo(String.class);
```

### Fruit API Integration

```java
// Add a fruit via Camel
from("direct:addFruit")
    .setHeader(Exchange.HTTP_METHOD, constant("POST"))
    .setHeader(Exchange.CONTENT_TYPE, constant("application/json"))
    .setBody(constant("{\"name\":\"Orange\",\"description\":\"Citrus fruit\"}"))
    .to("https://" + restServerUrl + "/fruits");
```

## Uninstallation

### Uninstall Helm Release

```bash
helm uninstall quarkus-rest -n $NAMESPACE
```

### Uninstall Direct YAML

```bash
oc delete -f quarkus-rest/quarkus-rest-server.yaml -n $NAMESPACE
```

## Troubleshooting

### Check Pod Status

```bash
oc get pods -l app=quarkus-rest-server -n $NAMESPACE
oc describe pod <pod-name> -n $NAMESPACE
```

### Check Logs

```bash
oc logs -l app=quarkus-rest-server -n $NAMESPACE
```

### Verify Health Endpoints

```bash
# Liveness probe
curl https://$REST_URL/q/health/live

# Readiness probe
curl https://$REST_URL/q/health/ready

# Overall health
curl https://$REST_URL/q/health
```

## Technical Details

- **Framework**: Quarkus 3.30.0
- **Runtime**: JVM mode (Java 17)
- **Build Tool**: Maven
- **Dependencies**:
  - quarkus-rest
  - quarkus-rest-jackson
  - quarkus-smallrye-openapi
- **Health Checks**: Quarkus SmallRye Health
- **OpenAPI**: OpenAPI 3.0 specification
- **Route**: OpenShift Route with edge TLS termination

## Additional Resources

- Source Code: https://github.com/Croway/quarkus-rest-server
- Quarkus Documentation: https://quarkus.io/
- OpenAPI Specification: https://spec.openapis.org/oas/v3.0.0
