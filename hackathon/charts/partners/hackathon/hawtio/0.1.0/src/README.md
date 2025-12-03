# Hawtio Helm Chart

A Helm chart for deploying [Hawtio Online](https://hawt.io) on OpenShift via the Hawtio Operator.

## Overview

Hawtio is a lightweight and modular web console for managing Java applications. This chart deploys Hawtio Online, which provides a web-based management console for discovering and managing Java applications running in OpenShift/Kubernetes.

## Prerequisites

- OpenShift cluster with the **Hawtio Operator** installed
- Helm 3.x
- `oc` CLI tool configured with cluster access

## Installation

### Add the Helm Repository

```bash
helm repo add hackathon https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
helm repo update
```

### Install the Chart

```bash
# Install with default values (cluster mode)
helm install hawtio hackathon/hawtio -n <namespace>

# Install with custom instance name
helm install my-hawtio hackathon/hawtio --set hawtio.name=my-hawtio -n <namespace>

# Install in namespace mode (only manages apps in the same namespace)
helm install hawtio hackathon/hawtio --set hawtio.type=namespace -n <namespace>

# Install with custom route hostname
helm install hawtio hackathon/hawtio --set hawtio.routeHostName=hawtio.example.com -n <namespace>
```

## Configuration

The following table lists the configurable parameters of the Hawtio chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `hawtio.name` | Name of the Hawtio instance | `hawtio-online` |
| `hawtio.type` | Deployment type: `cluster` or `namespace` | `cluster` |
| `hawtio.replicas` | Number of replicas | `1` |
| `hawtio.routeHostName` | Custom route hostname (auto-generated if not set) | `""` |
| `hawtio.resources.limits.cpu` | CPU limit | `1` |
| `hawtio.resources.limits.memory` | Memory limit | `200Mi` |
| `hawtio.resources.requests.cpu` | CPU request | `200m` |
| `hawtio.resources.requests.memory` | Memory request | `32Mi` |
| `hawtio.route.certSecret.name` | Custom TLS certificate secret name | `""` |
| `hawtio.route.caCert.name` | Custom CA certificate secret name | `""` |
| `hawtio.route.caCert.key` | Custom CA certificate secret key | `tls.crt` |
| `hawtio.externalRoutes` | List of external routes to annotate | `[]` |

### Deployment Types

- **cluster**: Hawtio can discover and manage applications across all namespaces the authenticated user has access to
- **namespace**: Hawtio can only discover and manage applications within the deployment namespace

## Examples

### Namespace-Scoped Deployment

Deploy Hawtio that only manages applications in the current namespace:

```bash
helm install hawtio hackathon/hawtio \
  --set hawtio.type=namespace \
  --set hawtio.name=hawtio-dev \
  -n my-dev-namespace
```

### High-Availability Deployment

Deploy Hawtio with multiple replicas:

```bash
helm install hawtio hackathon/hawtio \
  --set hawtio.replicas=3 \
  --set hawtio.resources.limits.cpu=2 \
  --set hawtio.resources.limits.memory=512Mi \
  -n <namespace>
```

### Custom TLS Certificate

First, create the TLS secret:

```bash
oc create secret tls hawtio-tls-cert --cert=tls.crt --key=tls.key -n <namespace>
```

Then install with custom certificate:

```bash
helm install hawtio hackathon/hawtio \
  --set hawtio.route.certSecret.name=hawtio-tls-cert \
  -n <namespace>
```

### With Custom CA Certificate

```bash
# Create secrets
oc create secret tls hawtio-tls-cert --cert=tls.crt --key=tls.key -n <namespace>
oc create secret generic hawtio-ca-cert --from-file=ca.crt=ca-cert.pem -n <namespace>

# Install with custom certificates
helm install hawtio hackathon/hawtio \
  --set hawtio.route.certSecret.name=hawtio-tls-cert \
  --set hawtio.route.caCert.name=hawtio-ca-cert \
  --set hawtio.route.caCert.key=ca.crt \
  -n <namespace>
```

## Accessing Hawtio

After installation, get the route URL:

```bash
# Get the route
HAWTIO_URL=$(oc get hawtio hawtio-online -o jsonpath='{.status.URL}' -n <namespace>)
echo "Hawtio URL: $HAWTIO_URL"

# Or view the route directly
oc get route -l app=hawtio -n <namespace>
```

Open the URL in your browser. You'll be redirected to OpenShift's OAuth login page. After authentication, Hawtio will be accessible.

## Managing the Deployment

### Scaling

```bash
# Scale up
helm upgrade hawtio hackathon/hawtio --set hawtio.replicas=3 -n <namespace>

# Or use kubectl
kubectl scale hawtio hawtio-online --replicas=3 -n <namespace>
```

### Updating Configuration

```bash
# Change to namespace mode
helm upgrade hawtio hackathon/hawtio --set hawtio.type=namespace -n <namespace>

# Update resource limits
helm upgrade hawtio hackathon/hawtio \
  --set hawtio.resources.limits.cpu=2 \
  --set hawtio.resources.limits.memory=512Mi \
  -n <namespace>
```

### Uninstalling

```bash
helm uninstall hawtio -n <namespace>
```

## Troubleshooting

### Check Hawtio Status

```bash
# Check the Hawtio custom resource
oc get hawtio -n <namespace>

# View detailed status
oc describe hawtio hawtio-online -n <namespace>

# Check pods
oc get pods -l app=hawtio -n <namespace>

# View logs
oc logs -l app=hawtio -n <namespace>
```

### Common Issues

1. **OAuth Login Issues**: Ensure your user has the appropriate RBAC permissions
2. **No Applications Visible**:
   - In cluster mode: Check if your user has view access to other namespaces
   - In namespace mode: Ensure applications are in the same namespace
3. **Route Not Accessible**: Verify the route was created: `oc get route -l app=hawtio`

## Features

- **Application Discovery**: Automatically discovers Java applications with JMX/Jolokia
- **JMX Management**: Full JMX management capabilities via Jolokia
- **Camel Integration**: Rich support for Apache Camel applications
- **OAuth Integration**: Seamless OpenShift OAuth authentication
- **Multi-Namespace Support**: Cluster mode allows managing apps across namespaces
- **Plugin Architecture**: Extensible with custom plugins

## Resources

- [Hawtio Website](https://hawt.io)
- [Hawtio Operator GitHub](https://github.com/hawtio/hawtio-operator)
- [Hawtio Online GitHub](https://github.com/hawtio/hawtio-online)
- [Hawtio Documentation](https://hawt.io/docs/)
