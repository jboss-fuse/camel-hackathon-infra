# Strimzi Kafka Cluster Helm Chart

A Helm chart for deploying Apache Kafka cluster using Strimzi Operator on OpenShift.

## Description

This chart deploys a Kafka cluster with KRaft mode (no ZooKeeper) using the Strimzi Kafka Operator.

## Prerequisites

### Required Operators

Before installing this chart, ensure the following operator is installed in your OpenShift cluster:

1. **Strimzi Kafka Operator** - for Kafka cluster deployment

You can install this operator from the OpenShift OperatorHub (Administrator → Operators → OperatorHub).

## Installation

### Using Helm CLI

```bash
helm install my-kafka hackathon/strimzi-cluster -n kafka --create-namespace
```

### Using OpenShift Console

1. Navigate to **Developer** perspective → **+Add** → **Helm Chart**
2. Search for "strimzi-cluster"
3. Configure the release and click **Install**

## Components Deployed

### 1. KafkaNodePool
Defines a pool of Kafka nodes with combined controller and broker roles.

**Default Configuration:**
- Name: `multirole`
- Roles: controller, broker
- Storage: Ephemeral
- Replicas: 1

### 2. Kafka Cluster
Apache Kafka cluster running in KRaft mode (no ZooKeeper).

**Default Configuration:**
- Name: `strimzi-cluster`
- Version: `3.9.0`
- Mode: KRaft (ZooKeeper-less)
- Replication factor: 1
- Listeners:
  - Plain (port 9092, no TLS)
  - TLS (port 9093, with TLS)

## Configuration

### Configurable Values

| Parameter | Description | Default |
|-----------|-------------|---------|
| `kafka.name` | Name of the Kafka cluster | `strimzi-cluster` |
| `kafka.version` | Kafka version | `3.9.0` |
| `nodePool.name` | Name of the node pool | `multirole` |
| `nodePool.replicas` | Number of Kafka nodes | `1` |

### Example: Custom Configuration

```bash
helm install my-kafka hackathon/strimzi-cluster \
  --set kafka.name=my-kafka-cluster \
  --set nodePool.replicas=3 \
  -n kafka
```

## Accessing Kafka

### From Within the Cluster

Applications within the same namespace can connect to Kafka using:

**Plain (no TLS):**
```
strimzi-cluster-kafka-bootstrap:9092
```

**TLS:**
```
strimzi-cluster-kafka-bootstrap:9093
```

### From Different Namespace

Use the fully qualified service name:

```
strimzi-cluster-kafka-bootstrap.kafka.svc.cluster.local:9092
```

## Verification

After installation, verify the Kafka cluster is running:

```bash
# Check Kafka cluster
oc get kafka -n kafka

# Check Kafka node pool
oc get kafkanodepool -n kafka

# Check Kafka pods
oc get pods -n kafka -l strimzi.io/cluster=strimzi-cluster

# Check Kafka services
oc get svc -n kafka
```

## Testing Kafka

### Create a Test Topic

```bash
# Get Kafka pod name
KAFKA_POD=$(oc get pods -n kafka -l strimzi.io/name=strimzi-cluster-kafka -o name | head -1)

# Create a topic
oc exec -n kafka $KAFKA_POD -- bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --create \
  --topic test-topic \
  --partitions 1 \
  --replication-factor 1

# List topics
oc exec -n kafka $KAFKA_POD -- bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --list
```

### Produce and Consume Messages

```bash
# Produce messages
oc exec -n kafka $KAFKA_POD -- bin/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic test-topic

# Consume messages (in another terminal)
oc exec -n kafka $KAFKA_POD -- bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic test-topic \
  --from-beginning
```

## Uninstalling

```bash
helm uninstall my-kafka -n kafka
```

All resources (Kafka cluster, node pool) will be automatically deleted.

## Troubleshooting

### Kafka Pods Not Starting

1. Check Kafka status:
   ```bash
   oc get kafka strimzi-cluster -n kafka -o yaml
   ```

2. Check node pool status:
   ```bash
   oc get kafkanodepool multirole -n kafka -o yaml
   ```

3. Check operator logs:
   ```bash
   oc logs -n openshift-operators deployment/strimzi-cluster-operator
   ```

### Connection Issues

1. Verify Kafka bootstrap service:
   ```bash
   oc get svc strimzi-cluster-kafka-bootstrap -n kafka
   ```

2. Check listener configuration:
   ```bash
   oc get kafka strimzi-cluster -n kafka -o jsonpath='{.spec.kafka.listeners}'
   ```

## Support

For issues or questions about this chart, please contact the Hackathon Team.

## License

These charts are provided as-is for the Hackathon event.
