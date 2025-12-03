# FTP Helm Chart

A Helm chart for deploying Apache FTP Server on OpenShift.

## Overview

This chart deploys an Apache FTP server with support for passive mode connections. The server uses a configurable port range (default: 2121-2130) to support FTP passive mode transfers.

## Prerequisites

- OpenShift cluster
- Helm 3.x

## Installation

### Add the Helm repository

```bash
helm repo add hackathon https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
helm repo update
```

### Install the chart

```bash
# Install with default values
helm install ftp-server hackathon/ftp -n <namespace>

# Install with custom instance name
helm install my-ftp hackathon/ftp --set ftp.name=my-ftp-server -n <namespace>

# Install with custom credentials
helm install ftp-server hackathon/ftp \
  --set ftp.username=admin \
  --set ftp.password=secret123 \
  -n <namespace>
```

## Configuration

The following table lists the configurable parameters of the FTP chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ftp.name` | Instance name for the FTP server | `ftp-server` |
| `ftp.username` | FTP username | `test` |
| `ftp.password` | FTP password | `test` |
| `ftp.replicas` | Number of replicas | `1` |
| `ftp.image.repository` | Image repository | `quay.io/fuse_qe/apache-ftp` |
| `ftp.image.tag` | Image tag | `latest` |
| `ftp.image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `ftp.service.type` | Service type | `ClusterIP` |
| `ftp.ports.control` | FTP control port | `2121` |
| `ftp.ports.passiveStart` | Start of passive port range | `2121` |
| `ftp.ports.passiveEnd` | End of passive port range | `2130` |
| `ftp.route.enabled` | Enable OpenShift Route | `true` |
| `ftp.route.tls.enabled` | Enable TLS on route | `true` |
| `ftp.route.tls.termination` | TLS termination type | `passthrough` |
| `ftp.resources.requests.memory` | Memory request | `256Mi` |
| `ftp.resources.requests.cpu` | CPU request | `100m` |
| `ftp.resources.limits.memory` | Memory limit | `512Mi` |
| `ftp.resources.limits.cpu` | CPU limit | `500m` |

## Usage

### Connecting to the FTP Server

After deployment, get the service hostname:

```bash
# Get the in-cluster hostname
echo "ftp://<ftp-server-name>.<namespace>.svc.cluster.local:2121"

# Get the route URL (if enabled)
ROUTE_URL=$(oc get route ftp-server -o jsonpath='{.spec.host}')
echo "Route: $ROUTE_URL"
```

### Testing the FTP Server

You can test the FTP server using standard FTP clients from within the cluster:

```bash
# Using lftp from within cluster
lftp -u test,test ftp://ftp-server.<namespace>.svc.cluster.local:2121

# Using curl from within cluster (via debug pod)
oc debug deployment/ftp-server -n <namespace> --image=curlimages/curl:latest -- curl -u test:test ftp://ftp-server:2121/
```

### Accessing FTP Server from External Applications

Since FTP uses a control connection and data connections on separate ports, OpenShift Routes do not work for FTP traffic. To access the FTP server from external applications, use `oc port-forward`:

```bash
# Forward the FTP control port (2121) and passive mode ports (2122-2130)
# Note: This requires multiple terminal sessions or run in background

# Control port
oc port-forward -n <namespace> svc/ftp-server 2121:2121 &

# Passive mode ports (run each in separate terminal or background)
for port in {2122..2130}; do
  oc port-forward -n <namespace> svc/ftp-server $port:$port &
done

# Now you can connect from your local machine
curl -u test:test ftp://localhost:2121/

# To stop all port-forwards
pkill -f "port-forward.*ftp-server"
```

**Alternative**: Use a single command to forward all ports in background:

```bash
# Forward all FTP ports in background
for port in {2121..2130}; do
  oc port-forward -n <namespace> svc/ftp-server $port:$port > /dev/null 2>&1 &
done

# Verify port-forwards are running
ps aux | grep port-forward
```

### Upload/Download Files

```bash
# Upload a file
curl -u test:test -T /path/to/local/file.txt ftp://ftp-server.<namespace>.svc.cluster.local:2121/

# Download a file
curl -u test:test ftp://ftp-server.<namespace>.svc.cluster.local:2121/file.txt -o local-file.txt

# List files
curl -u test:test ftp://ftp-server.<namespace>.svc.cluster.local:2121/
```

## FTP Passive Mode

This FTP server is configured for passive mode by default, using ports 2121-2130. All these ports are exposed through the Kubernetes Service.

## Uninstallation

```bash
helm uninstall ftp-server -n <namespace>
```

## Direct YAML Deployment

If you prefer to deploy without Helm, you can use the direct YAML manifest:

```bash
oc create -f ftp/ftp-server.yaml
```

## Customization Examples

### High-Resource Configuration

```bash
helm install ftp-server hackathon/ftp \
  --set ftp.resources.limits.memory=1Gi \
  --set ftp.resources.limits.cpu=1000m \
  --set ftp.resources.requests.memory=512Mi \
  --set ftp.resources.requests.cpu=250m \
  -n <namespace>
```

### Custom Port Range

```bash
helm install ftp-server hackathon/ftp \
  --set ftp.ports.passiveStart=3000 \
  --set ftp.ports.passiveEnd=3009 \
  -n <namespace>
```

## Support

For issues and questions, please refer to the main repository documentation.
