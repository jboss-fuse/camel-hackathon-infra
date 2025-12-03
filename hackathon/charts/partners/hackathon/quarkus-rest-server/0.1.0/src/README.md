# Quarkus REST Server Helm Chart

This Helm chart deploys the Quarkus REST Server on OpenShift.

## Overview

The Quarkus REST Server provides REST API endpoints with OpenAPI documentation for:
- Fruit CRUD operations
- Slow service (delayed responses)
- Faulty service (error handling)
- Multipart file upload/download
- OpenAPI/Swagger UI

## Configuration

The following table lists the configurable parameters of the chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `restServer.name` | Instance name for resources | `quarkus-rest-server` |
| `restServer.replicas` | Number of replicas | `1` |
| `restServer.image.repository` | Container image repository | `quay.io/fuse_qe/quarkus-rest-server` |
| `restServer.image.tag` | Container image tag | `1.0.0` |
| `restServer.image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `restServer.service.type` | Service type | `ClusterIP` |
| `restServer.service.port` | Service port | `8080` |
| `restServer.route.enabled` | Enable OpenShift Route | `true` |
| `restServer.route.tls.enabled` | Enable TLS for Route | `true` |
| `restServer.route.tls.termination` | TLS termination type | `edge` |
| `restServer.resources.requests.memory` | Memory request | `512Mi` |
| `restServer.resources.requests.cpu` | CPU request | `250m` |
| `restServer.resources.limits.memory` | Memory limit | `2Gi` |
| `restServer.resources.limits.cpu` | CPU limit | `1000m` |

## Installation

### Install with default values

```bash
helm install my-rest-server hackathon/quarkus-rest-server -n my-namespace
```

### Install with custom name

```bash
helm install my-rest-server hackathon/quarkus-rest-server \
  --set restServer.name=my-quarkus-rest \
  -n my-namespace
```

### Install with custom resource limits

```bash
helm install my-rest-server hackathon/quarkus-rest-server \
  --set restServer.resources.limits.memory=4Gi \
  --set restServer.resources.limits.cpu=2000m \
  -n my-namespace
```

## Accessing the REST Server

After installation, get the route URL:

```bash
oc get route <restServer.name> -o jsonpath='{.spec.host}'
```

Access the following endpoints:
- OpenAPI: `https://<route-url>/q/openapi`
- Swagger UI: `https://<route-url>/q/swagger-ui`
- Health: `https://<route-url>/q/health`
- Fruits API: `https://<route-url>/fruits`

## Uninstallation

```bash
helm uninstall my-rest-server -n my-namespace
```
