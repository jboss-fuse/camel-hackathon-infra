# OpenTelemetry Infrastructure Helm Chart

A comprehensive Helm chart for deploying the complete OpenTelemetry observability stack on OpenShift, including Grafana Tempo for distributed tracing.

## Description

This chart deploys a complete observability infrastructure stack with the following components:

1. **Grafana Tempo Monolithic** - Distributed tracing backend
2. **OpenTelemetry Collector** - Collects, processes, and exports telemetry data
3. **Auto-Instrumentation** - Automatic Java/Camel application instrumentation
4. **RBAC** - Complete role-based access control setup
5. **ServiceMonitor** - Prometheus metrics integration

## Prerequisites

### Required Operators

Before installing this chart, ensure the following operators are installed in your OpenShift cluster:

1. **Tempo Operator** - for Tempo Monolithic deployment
2. **OpenTelemetry Operator** - for OpenTelemetry Collector and Instrumentation
3. **Prometheus Operator** (optional) - for ServiceMonitor integration

You can install these operators from the OpenShift OperatorHub (Administrator → Operators → OperatorHub).

## Installation

### Using Helm CLI

```bash
helm install my-otel-stack hackathon/opentelemetry-infra -n observability --create-namespace
```

### Using OpenShift Console

1. Navigate to **Developer** perspective → **+Add** → **Helm Chart**
2. Search for "opentelemetry-infra"
3. Configure the release and click **Install**

## Components Deployed

### 1. Tempo Monolithic
Distributed tracing backend with OpenTelemetry support and multi-tenancy.

**Default Configuration:**
- Name: `monolitic`
- Multi-tenancy: Enabled (OpenShift mode)
- Tenant ID: `application`
- Storage: Memory (for demo/testing)
- OTLP ingestion: gRPC and HTTP enabled

### 2. ServiceAccount
ServiceAccount for OpenTelemetry Collector pods.

**Default:** `otel-collector-sa`

### 3. ClusterRoles
Two ClusterRoles for trace access control:
- **tempo-traces-reader** - Read access to traces
- **tempo-traces-write** - Write access to traces

**Note:** These resources are preserved on uninstall (kept for reuse).

### 4. ClusterRoleBindings
- **tempo-traces-reader** - Binds reader role to `system:authenticated` group (preserved on uninstall)
- **tempo-traces-from-otel** - Binds writer role to OTel Collector ServiceAccount

### 5. OpenTelemetry Collector
Collects telemetry data from applications and exports to Tempo.

**Default Configuration:**
- Name: `otel-collector`
- Replicas: 1
- Receivers: OTLP (gRPC + HTTP)
- Exporters: Tempo (OTLP) + Prometheus
- Batch processing: 500 records, 10s timeout
- Prometheus endpoint: `:8889`

### 6. Instrumentation
Auto-instrumentation for Java/Camel applications.

**Default Configuration:**
- Name: `camel-instrumentation`
- Java image: `ghcr.io/open-telemetry/opentelemetry-operator/autoinstrumentation-java:2.22.0`
- Sampling: `always_on`
- JMX target: Camel
- Disabled instrumentations: Apache HTTP Client, Undertow, Servlet

### 7. ServiceMonitor
Prometheus ServiceMonitor for scraping metrics from OTel Collector.

**Default Configuration:**
- Name: `otel-collector-sm`
- Scrape interval: 30s

## Configuration

### Configurable Values

| Parameter | Description | Default |
|-----------|-------------|---------|
| `tempo.name` | Name of Tempo instance | `monolitic` |
| `serviceAccount.name` | ServiceAccount name | `otel-collector-sa` |
| `clusterRole.reader.name` | Reader ClusterRole name | `tempo-traces-reader` |
| `clusterRole.writer.name` | Writer ClusterRole name | `tempo-traces-write` |
| `clusterRoleBinding.reader.name` | Reader binding name | `tempo-traces-reader` |
| `clusterRoleBinding.writer.name` | Writer binding name | `tempo-traces-from-otel` |
| `otelCollector.name` | OTel Collector name | `otel-collector` |
| `otelCollector.tenantId` | Tempo tenant ID | `application` |
| `otelCollector.metricExpiration` | Metric expiration time | `180m` |
| `instrumentation.name` | Instrumentation resource name | `camel-instrumentation` |
| `instrumentation.javaImage` | Java auto-instrumentation image | `ghcr.io/open-telemetry/...` |
| `instrumentation.sampler` | Trace sampling strategy | `always_on` |
| `instrumentation.logsExporter` | Logs exporter type | `none` |
| `instrumentation.jmxTargetSystem` | JMX target system | `camel` |
| `serviceMonitor.name` | ServiceMonitor name | `otel-collector-sm` |
| `serviceMonitor.interval` | Scrape interval | `30s` |

### Example: Custom Configuration

```bash
helm install my-otel-stack hackathon/opentelemetry-infra \
  --set tempo.name=my-tempo \
  --set otelCollector.tenantId=production \
  --set instrumentation.sampler=parentbased_traceidratio \
  -n observability
```

## Using Auto-Instrumentation

To enable automatic instrumentation for your Java/Camel applications, add the following annotation to your Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-camel-app
spec:
  template:
    metadata:
      annotations:
        instrumentation.opentelemetry.io/inject-java: "true"
    spec:
      containers:
      - name: app
        image: my-camel-app:latest
```

The OpenTelemetry Operator will automatically inject the Java agent and configure it to send traces to the collector.

## Verification

After installation, verify all resources are created:

```bash
# Check Tempo instance
oc get tempomono -n observability

# Check OpenTelemetry Collector
oc get otelcol -n observability

# Check Instrumentation
oc get instrumentation -n observability

# Check ServiceAccount
oc get sa -n observability | grep otel-collector

# Check ClusterRoles (cluster-scoped)
oc get clusterrole | grep tempo-traces

# Check ClusterRoleBindings (cluster-scoped)
oc get clusterrolebinding | grep tempo-traces
```

## Accessing Traces and Metrics

### View Traces via OpenShift Console

1. Navigate to the OpenShift Console
2. Go to **Observe** → **Traces**
3. Select the Tempo instance from the dropdown (e.g., `monolitic`)
4. You can now search and view distributed traces from your applications

**Note:** Make sure your applications are instrumented and sending traces to the OpenTelemetry Collector.

### View Metrics via OpenShift Console

1. Navigate to the OpenShift Console
2. Go to **Observe** → **Metrics**
3. Use PromQL queries to view metrics from the OpenTelemetry Collector
4. Example queries:
   - `{job="otel-collector"}` - All metrics from OTel Collector
   - `otelcol_exporter_sent_spans` - Number of spans sent to exporters
   - `otelcol_receiver_accepted_spans` - Number of spans accepted by receivers

### Alternative: Direct API Access

For programmatic access or troubleshooting:

#### Query Traces via Tempo Gateway

```bash
# Port-forward to Tempo gateway
oc port-forward svc/tempo-monolitic-gateway 3200:3200 -n observability

# Query traces (requires authentication token)
curl -H "X-Scope-OrgID: application" \
     http://localhost:3200/api/search
```

#### View Collector Metrics

```bash
# Port-forward to OTel Collector Prometheus endpoint
oc port-forward svc/otel-collector 8889:8889 -n observability

# View metrics
curl http://localhost:8889/metrics
```

## Uninstalling

```bash
helm uninstall my-otel-stack -n observability
```

**Note:** The following cluster-scoped resources are intentionally preserved for reuse:
- ClusterRole: `tempo-traces-reader`
- ClusterRole: `tempo-traces-write`
- ClusterRoleBinding: `tempo-traces-reader`

To manually remove them:

```bash
oc delete clusterrole tempo-traces-reader tempo-traces-write
oc delete clusterrolebinding tempo-traces-reader
```

All other resources (Tempo, OTel Collector, Instrumentation, ServiceMonitor, ServiceAccount, and the writer ClusterRoleBinding) will be automatically deleted.

## Troubleshooting

### Collector Not Receiving Traces

1. Check collector logs:
   ```bash
   oc logs deployment/otel-collector -n observability
   ```

2. Verify instrumentation injection:
   ```bash
   oc describe pod <app-pod> -n <app-namespace> | grep -A 10 "OTEL"
   ```

### Tempo Not Accessible

1. Check Tempo status:
   ```bash
   oc get tempomono monolitic -n observability
   ```

2. Check gateway service:
   ```bash
   oc get svc tempo-monolitic-gateway -n observability
   ```

### RBAC Issues

1. Verify ClusterRoleBindings:
   ```bash
   oc get clusterrolebinding tempo-traces-from-otel
   ```

2. Check ServiceAccount:
   ```bash
   oc get sa otel-collector-sa -n observability
   ```

