# IBM MQ

IBM MQ Queue Manager deployment for the Camel Hackathon, configured without authentication for easy development use.

## Quick Start

### Deploy with Helm

```bash
# Add Helm repository
helm repo add hackathon https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
helm repo update

# Install IBM MQ (no authentication by default)
helm install ibmmq hackathon/ibm-mq -n <namespace>
```

### Deploy with YAML

```bash
# No authentication version (recommended for development)
oc create -f ibmmq/queuemanager-noauth.yaml -n <namespace>

# With manual authentication
oc create -f ibmmq/queuemanager.yaml -n <namespace>
```

## Access Information

### Web Console

URL: `https://quickstart-ibm-mq-web-<namespace>.apps.<cluster-domain>/ibmmq/console/`

No login required (when using no-auth configuration).

### JMS/Client Connection

**Connection Details:**
- Hostname: `quickstart-ibm-mq.<namespace>.svc.cluster.local`
- Port: `1414`
- Queue Manager: `QUICKSTART`
- Channel: `DEV.APP.SVRCONN` or `SYSTEM.DEF.SVRCONN`
- Username: Not required
- Password: Not required

**Example Connection String:**
```properties
mqConnectionFactory.hostName=quickstart-ibm-mq.<namespace>.svc.cluster.local
mqConnectionFactory.port=1414
mqConnectionFactory.queueManager=QUICKSTART
mqConnectionFactory.channel=DEV.APP.SVRCONN
mqConnectionFactory.transportType=1
```

**Example Java Code:**
```java
MQConnectionFactory cf = new MQConnectionFactory();
cf.setHostName("quickstart-ibm-mq.<namespace>.svc.cluster.local");
cf.setPort(1414);
cf.setQueueManager("QUICKSTART");
cf.setChannel("DEV.APP.SVRCONN");
cf.setTransportType(WMQConstants.WMQ_CM_CLIENT);
// No authentication needed
Connection connection = cf.createConnection();
```

## Helm Chart Configuration

### Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `queueManager.name` | QueueManager resource name | `quickstart` |
| `queueManager.mqName` | MQ Queue Manager name | `QUICKSTART` |
| `queueManager.web.authentication.enabled` | Enable web console auth | `false` |
| `queueManager.mqsc.enabled` | Enable MQSC config (disables JMS auth) | `true` |
| `queueManager.storage.type` | Storage type | `ephemeral` |

### Examples

**Custom name:**
```bash
helm install my-mq hackathon/ibm-mq \
  --set queueManager.name=my-queue-manager \
  --set queueManager.mqName=MYQM \
  -n <namespace>
```

**Enable authentication:**
```bash
helm install ibmmq hackathon/ibm-mq \
  --set queueManager.web.authentication.enabled=true \
  --set queueManager.mqsc.enabled=false \
  -n <namespace>
```

## Creating Queues

```bash
# Access the pod
oc exec -it quickstart-ibm-mq-0 -n <namespace> -- bash

# Create a queue
echo "DEFINE QLOCAL('MY.QUEUE')" | runmqsc QUICKSTART

# List queues
echo "DISPLAY QLOCAL('*')" | runmqsc QUICKSTART
```

## Common Operations

**Check Queue Manager status:**
```bash
oc get queuemanager -n <namespace>
```

**View logs:**
```bash
oc logs quickstart-ibm-mq-0 -n <namespace>
```

**Get connection details:**
```bash
oc get service quickstart-ibm-mq -n <namespace>
oc get route quickstart-ibm-mq-web -n <namespace>
```

## Prerequisites

- IBM MQ Operator must be installed in the cluster
- OpenShift Container Platform 4.12 or later

## License

Uses IBM MQ Advanced for Developers (Non-Warranted) license - **FREE** for development use.

## Uninstall

```bash
# Helm
helm uninstall ibmmq -n <namespace>

# YAML
oc delete -f ibmmq/queuemanager-noauth.yaml -n <namespace>
```

## More Information

- [IBM MQ Documentation](https://www.ibm.com/docs/en/ibm-mq/9.4)
- [IBM MQ Operator Documentation](https://www.ibm.com/docs/en/ibm-mq/9.4?topic=mq-in-containers-cloud-pak-integration)
