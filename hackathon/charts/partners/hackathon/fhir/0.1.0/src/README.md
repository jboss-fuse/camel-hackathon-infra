# HAPI FHIR Server Helm Chart

This Helm chart deploys a HAPI FHIR server on OpenShift.

## Overview

HAPI FHIR is an open-source implementation of the HL7 FHIR (Fast Healthcare Interoperability Resources) standard for healthcare data exchange.

## Configuration

The following table lists the configurable parameters of the FHIR chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `fhir.version` | FHIR version (DSTU3, R4, R5) | `R4` |
| `fhir.name` | Instance name for resources | `fhir-server` |
| `fhir.auth.enabled` | Enable basic authentication | `false` |
| `fhir.auth.username` | Username for authentication | `admin` |
| `fhir.auth.password` | Password for authentication | `admin` |
| `fhir.resources.requests.memory` | Memory request | `512Mi` |
| `fhir.resources.requests.cpu` | CPU request | `250m` |
| `fhir.resources.limits.memory` | Memory limit | `2Gi` |
| `fhir.resources.limits.cpu` | CPU limit | `1000m` |
| `fhir.replicas` | Number of replicas | `1` |
| `fhir.image.repository` | Container image repository | `quay.io/fuse_qe/hapi-fhir` |
| `fhir.image.tag` | Container image tag | `v8.2.0` |
| `fhir.service.type` | Service type | `ClusterIP` |
| `fhir.service.port` | Service port | `8080` |
| `fhir.route.enabled` | Enable OpenShift Route | `true` |
| `fhir.route.tls.enabled` | Enable TLS for Route | `true` |

## Installation

### Install with default values (R4, no auth)

```bash
helm install my-fhir hackathon/fhir -n my-namespace
```

### Install FHIR DSTU3 with authentication

```bash
helm install my-fhir hackathon/fhir \
  --set fhir.version=DSTU3 \
  --set fhir.auth.enabled=true \
  --set fhir.auth.username=admin \
  --set fhir.auth.password=admin \
  -n my-namespace
```

### Install FHIR R4 with custom name

```bash
helm install my-fhir hackathon/fhir \
  --set fhir.name=my-fhir-r4 \
  --set fhir.version=R4 \
  -n my-namespace
```

### Install with custom resource limits

```bash
helm install my-fhir hackathon/fhir \
  --set fhir.resources.limits.memory=4Gi \
  --set fhir.resources.limits.cpu=2000m \
  -n my-namespace
```

## Accessing the FHIR Server

After installation, get the route URL:

```bash
oc get route <release-name>-fhir-server -o jsonpath='{.spec.host}'
```

Or use the configured name:

```bash
oc get route <fhir.name> -o jsonpath='{.spec.host}'
```

Test the FHIR metadata endpoint:

```bash
# Without authentication
curl https://<route-url>/fhir/metadata

# With authentication
curl -u admin:admin https://<route-url>/fhir/metadata
```

## Uninstallation

```bash
helm uninstall my-fhir -n my-namespace
```

## Notes

- The FHIR server may take 30-60 seconds to become ready due to bulk job initialization
- For production use, store credentials in Kubernetes Secrets instead of using values.yaml
- The server exposes the FHIR API at the `/fhir` base path
