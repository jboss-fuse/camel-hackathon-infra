# Hackathon Helm Charts Repository

This repository contains Helm charts for the Hackathon infrastructure components on OpenShift.

## Available Charts

- **activemq-artemis** (v0.1.0) - ActiveMQ Artemis Broker
- **postgres-cluster** (v0.1.0) - PostgreSQL Cluster using Crunchy Data Operator
- **tempo-monolithic** (v0.1.0) - Grafana Tempo in monolithic mode for distributed tracing

## Prerequisites

### Operators Required

Before installing these charts, ensure the following operators are installed in your OpenShift cluster:

1. **AMQ Broker Operator** - for activemq-artemis chart
2. **Crunchy Data PostgreSQL Operator** - for postgres-cluster chart
3. **Tempo Operator** - for tempo-monolithic chart

You can install these operators from the OpenShift OperatorHub.

## Installation Methods

### Method 1: Using OpenShift HelmChartRepository (Recommended)

This method makes the charts available in the OpenShift Developer Console.

#### Step 1: Apply the HelmChartRepository Resource

```bash
oc apply -f https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/openshift-charts-repo.yaml
```

Or if you have the file locally:

```bash
oc apply -f openshift-charts-repo.yaml
```

#### Step 2: Verify the Repository is Added

```bash
oc get helmchartrepositories
```

You should see `hackathon-helm-charts` in the list.

#### Step 3: Install Charts via OpenShift Web Console

1. Navigate to the **Developer** perspective
2. Click **+Add** → **Helm Chart**
3. You'll see the Hackathon Helm Charts repository
4. Select the chart you want to install
5. Configure the release name and namespace
6. Click **Install**

### Method 2: Using Helm CLI

#### Step 1: Add the Helm Repository

```bash
helm repo add hackathon https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
```

#### Step 2: Update Repository Index

```bash
helm repo update
```

#### Step 3: Search Available Charts

```bash
helm search repo hackathon
```

#### Step 4: Install a Chart

Install ActiveMQ Artemis:
```bash
helm install artemis-broker hackathon/activemq-artemis -n my-namespace --create-namespace
```

Install PostgreSQL Cluster:
```bash
helm install postgres hackathon/postgres-cluster -n my-namespace --create-namespace
```

Install Tempo Monolithic:
```bash
helm install tempo hackathon/tempo-monolithic -n my-namespace --create-namespace
```

## Customizing Chart Values

Each chart has minimal configuration options. You can override the instance name:

### ActiveMQ Artemis

```bash
helm install my-broker hackathon/activemq-artemis \
  --set broker.name=my-artemis-broker \
  -n my-namespace
```

### PostgreSQL Cluster

```bash
helm install my-db hackathon/postgres-cluster \
  --set cluster.name=my-postgres \
  -n my-namespace
```

### Tempo Monolithic

```bash
helm install my-tempo hackathon/tempo-monolithic \
  --set tempo.name=my-tempo-instance \
  -n my-namespace
```

## Viewing Chart Details

To see the default values:

```bash
helm show values hackathon/activemq-artemis
helm show values hackathon/postgres-cluster
helm show values hackathon/tempo-monolithic
```

To see the full chart information:

```bash
helm show chart hackathon/activemq-artemis
helm show readme hackathon/postgres-cluster
```

## Uninstalling Charts

Using Helm CLI:

```bash
helm uninstall <release-name> -n <namespace>
```

Example:
```bash
helm uninstall artemis-broker -n my-namespace
```

Using OpenShift Console:
1. Navigate to **Helm** in the Developer perspective
2. Click on the release you want to uninstall
3. Click **Actions** → **Uninstall Helm Release**

## Chart Repository Structure

This repository follows the OpenShift Helm Charts directory structure:

```
hackathon/
├── openshift-charts-repo.yaml          # HelmChartRepository resource
├── index.yaml                          # Helm repository index
├── activemq-artemis-0.1.0.tgz         # Packaged charts
├── postgres-cluster-0.1.0.tgz
├── tempo-monolithic-0.1.0.tgz
└── charts/
    └── partners/
        └── hackathon/
            ├── activemq-artemis/0.1.0/src/
            ├── postgres-cluster/0.1.0/src/
            └── tempo-monolithic/0.1.0/src/
```

## Troubleshooting

### Chart Repository Not Showing in Console

1. Verify the HelmChartRepository is created:
   ```bash
   oc get helmchartrepositories
   ```

2. Check the repository status:
   ```bash
   oc describe helmchartrepository hackathon-helm-charts
   ```

### Chart Installation Fails

1. Ensure the required operator is installed:
   ```bash
   oc get csv -A | grep -E 'amq-broker|postgres-operator|tempo'
   ```

2. Check operator availability in the target namespace

3. Review the Helm release status:
   ```bash
   helm status <release-name> -n <namespace>
   ```

### Access Issues

Ensure your cluster can reach the GitLab repository URL. If behind a proxy or firewall, you may need to configure network policies or use an internal mirror of the repository.

## Support

For issues or questions about these charts, please contact the Hackathon Team.

## License

These charts are provided as-is for the Hackathon event.
