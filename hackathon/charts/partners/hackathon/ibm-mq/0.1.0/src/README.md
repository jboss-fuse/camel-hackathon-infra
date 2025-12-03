# IBM MQ Queue Manager Helm Chart

This Helm chart deploys an IBM MQ Queue Manager using the IBM MQ Operator.

## Prerequisites

- IBM MQ Operator must be installed in the cluster
- OpenShift Container Platform 4.12 or later

## Installation

### Using Helm CLI

```bash
# Basic installation
helm install quickstart hackathon/ibm-mq -n <namespace>

# Custom instance name
helm install my-mq hackathon/ibm-mq --set queueManager.name=my-queue-manager --set queueManager.mqName=MYQM -n <namespace>

# Production license with persistent storage
helm install prod-mq hackathon/ibm-mq \
  --set queueManager.license.license=L-NUUP-23NH8Y \
  --set queueManager.license.use=Production \
  --set queueManager.storage.type=persistent-claim \
  -n <namespace>
```

## Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `queueManager.name` | Name of the QueueManager resource | `quickstart` |
| `queueManager.mqName` | IBM MQ Queue Manager name (uppercase) | `QUICKSTART` |
| `queueManager.license.accept` | Accept the license | `true` |
| `queueManager.license.license` | License identifier | `L-CYPF-CRPF3H` |
| `queueManager.license.use` | License use (NonProduction/Production) | `NonProduction` |
| `queueManager.resources.limits.cpu` | CPU limit | `500m` |
| `queueManager.resources.requests.cpu` | CPU request | `500m` |
| `queueManager.storage.type` | Storage type (ephemeral/persistent-claim) | `ephemeral` |
| `queueManager.version` | MQ version | `9.4.4.0-r3` |
| `queueManager.web.enabled` | Enable web console | `true` |

## License Options

### NonProduction (Default)
```yaml
queueManager:
  license:
    license: L-CYPF-CRPF3H
    use: NonProduction
```

### Production
```yaml
queueManager:
  license:
    license: L-NUUP-23NH8Y
    use: Production
```

## Accessing the Queue Manager

After installation, you can access the web console through the OpenShift route:

```bash
# Get the route URL
oc get route {{ .Values.queueManager.name }}-ibm-mq-web -o jsonpath='{.spec.host}'
```

## Connecting Applications

The Queue Manager is accessible at:

```bash
# Get the service name
oc get service {{ .Values.queueManager.name }}-ibm-mq
```

Connection details:
- Hostname: `{{ .Values.queueManager.name }}-ibm-mq.<namespace>.svc.cluster.local`
- Port: 1414
- Channel: SYSTEM.DEF.SVRCONN
- Queue Manager: {{ .Values.queueManager.mqName }}

## Examples

### Creating a Queue

```bash
# Access the pod
oc exec -it {{ .Values.queueManager.name }}-ibm-mq-0 -- bash

# Run MQSC commands
echo "DEFINE QLOCAL('MY.QUEUE')" | runmqsc {{ .Values.queueManager.mqName }}
```

### Multi-Instance Configuration

For production deployments with high availability:

```yaml
queueManager:
  license:
    license: L-NUUP-23NH8Y
    use: Production
  storage:
    type: persistent-claim
  availability:
    type: MultiInstance
```

### Native HA Configuration

For native high availability:

```yaml
queueManager:
  license:
    license: L-NUUP-23NH8Y
    use: Production
  availability:
    type: NativeHA
```

## Uninstallation

```bash
helm uninstall <release-name> -n <namespace>
```

## More Information

- [IBM MQ Documentation](https://www.ibm.com/docs/en/ibm-mq/9.4)
- [IBM MQ Operator Documentation](https://www.ibm.com/docs/en/ibm-mq/9.4?topic=mq-in-containers-cloud-pak-integration)
