# Quarkus CXF Server Helm Chart

This Helm chart deploys the Quarkus CXF SOAP Server on OpenShift.

## Overview

The Quarkus CXF Server provides SOAP web services using Apache CXF:
- HelloService - Simple greeting service
- FruitService - Fruit CRUD operations
- FaultyHelloService - SOAP fault handling
- SlowHelloService - Delayed responses (1s)
- MtomService - Binary data with MTOM

All services are available under the `/soap` base path.

## Configuration

The following table lists the configurable parameters of the chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `cxfServer.name` | Instance name for resources | `quarkus-cxf-server` |
| `cxfServer.replicas` | Number of replicas | `1` |
| `cxfServer.image.repository` | Container image repository | `quay.io/fuse_qe/quarkus-cxf-server` |
| `cxfServer.image.tag` | Container image tag | `1.0.0` |
| `cxfServer.image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `cxfServer.service.type` | Service type | `ClusterIP` |
| `cxfServer.service.port` | Service port | `8080` |
| `cxfServer.route.enabled` | Enable OpenShift Route | `true` |
| `cxfServer.route.tls.enabled` | Enable TLS for Route | `true` |
| `cxfServer.route.tls.termination` | TLS termination type | `edge` |
| `cxfServer.resources.requests.memory` | Memory request | `512Mi` |
| `cxfServer.resources.requests.cpu` | CPU request | `250m` |
| `cxfServer.resources.limits.memory` | Memory limit | `2Gi` |
| `cxfServer.resources.limits.cpu` | CPU limit | `1000m` |

## Installation

### Install with default values

```bash
helm install my-cxf-server hackathon/quarkus-cxf-server -n my-namespace
```

### Install with custom name

```bash
helm install my-cxf-server hackathon/quarkus-cxf-server \
  --set cxfServer.name=my-quarkus-cxf \
  -n my-namespace
```

### Install with custom resource limits

```bash
helm install my-cxf-server hackathon/quarkus-cxf-server \
  --set cxfServer.resources.limits.memory=4Gi \
  --set cxfServer.resources.limits.cpu=2000m \
  -n my-namespace
```

## Accessing the CXF Server

After installation, get the route URL:

```bash
oc get route <cxfServer.name> -o jsonpath='{.spec.host}'
```

Access the WSDL files:
- HelloService: `https://<route-url>/soap/hello?wsdl`
- FruitService: `https://<route-url>/soap/fruits?wsdl`
- FaultyHelloService: `https://<route-url>/soap/faulty-hello?wsdl`
- SlowHelloService: `https://<route-url>/soap/SlowHelloServiceImpl?wsdl`
- MtomService: `https://<route-url>/soap/mtom?wsdl`
- Health: `https://<route-url>/q/health`

## Uninstallation

```bash
helm uninstall my-cxf-server -n my-namespace
```
