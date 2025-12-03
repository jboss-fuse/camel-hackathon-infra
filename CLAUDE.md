# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains infrastructure resources for the 2025 December Camel Hackathon, providing third-party services deployable on OpenShift via both direct YAML manifests and packaged Helm charts.

## Architecture

### Two Deployment Approaches

The repository supports two parallel deployment methods:

1. **Direct YAML Manifests** (in `activemq/`, `postgres/`, `observability/`, `fhir/`, `quarkus-rest/`, `quarkus-cxf/`): Custom Resource definitions or standard Kubernetes resources that can be applied directly with `oc create -f` or `oc apply -f`
2. **Helm Charts** (in `hackathon/`): Packaged Helm charts that wrap the same CRDs for deployment via Helm or OpenShift Developer Console

### Service Components

Seven core services are provided:

- **ActiveMQ Artemis Broker**: Message broker using AMQ Broker Operator
  - Direct manifest: `activemq/broker.yaml`
  - Helm chart: `hackathon/activemq-artemis-0.1.0.tgz`
  - Creates `ActiveMQArtemis` custom resource (API: `broker.amq.io/v1beta1`)

- **PostgreSQL Cluster**: Database using Crunchy Data PostgreSQL Operator
  - Direct manifest: `postgres/postgres.yaml`
  - Helm chart: `hackathon/postgres-cluster-0.1.0.tgz`
  - Creates `PostgresCluster` custom resource (API: `postgres-operator.crunchydata.com/v1beta1`)

- **Observability Stack**: Two deployment options available:
  - **Tempo Only** (minimal): Basic distributed tracing backend
    - Direct manifest: `observability/tempo.yaml`
    - Helm chart: `hackathon/tempo-monolithic-0.1.0.tgz`
    - Creates `TempoMonolithic` custom resource (API: `tempo.grafana.com/v1alpha1`)
  - **Full OpenTelemetry Infrastructure** (recommended): Complete observability stack including Tempo, OpenTelemetry Collector, auto-instrumentation, RBAC, and ServiceMonitor
    - Direct manifest: `observability/otel.yaml`
    - Helm chart: `hackathon/opentelemetry-infra-0.1.0.tgz`
    - Creates: `TempoMonolithic`, `OpenTelemetryCollector`, `Instrumentation`, ClusterRoles, ServiceMonitor
    - Enables automatic Java/Camel application instrumentation

- **HAPI FHIR Server**: Healthcare interoperability server
  - Direct manifests in `fhir/` directory (3 variants: DSTU3 with auth, DSTU3 without auth, R4 default)
  - Helm chart: `hackathon/fhir-0.1.0.tgz`
  - Uses standard Kubernetes Deployment, Service, and OpenShift Route resources
  - Image: `quay.io/fuse_qe/hapi-fhir:v8.2.0`
  - Supports FHIR versions: DSTU3, R4, R5

- **Quarkus REST Server**: REST API demo with OpenAPI documentation
  - Direct manifest: `quarkus-rest/quarkus-rest-server.yaml`
  - Helm chart: `hackathon/quarkus-rest-server-0.1.0.tgz`
  - Uses standard Kubernetes Deployment, Service, and OpenShift Route resources
  - Image: `quay.io/fuse_qe/quarkus-rest-server:1.0.0`
  - Provides: Fruit CRUD API, Slow/Faulty services, Multipart upload/download, OpenAPI/Swagger UI
  - Port: 8080, Health: `/q/health/live`, `/q/health/ready`

- **Quarkus CXF Server**: SOAP web services demo using Apache CXF
  - Direct manifest: `quarkus-cxf/quarkus-cxf-server.yaml`
  - Helm chart: `hackathon/quarkus-cxf-server-0.1.0.tgz`
  - Uses standard Kubernetes Deployment, Service, and OpenShift Route resources
  - Image: `quay.io/fuse_qe/quarkus-cxf-server:1.0.0`
  - Provides: Hello, Fruit, Faulty, Slow, MTOM SOAP services
  - Base path: `/soap`, Port: 8080, WSDLs: `?wsdl` on each endpoint

### Helm Repository Structure

The `hackathon/` directory serves as a Helm repository compatible with OpenShift's HelmChartRepository:

```
hackathon/
├── index.yaml                          # Helm repository index
├── openshift-charts-repo.yaml          # HelmChartRepository CRD for OpenShift
├── activemq-artemis-0.1.0.tgz         # Packaged charts
├── postgres-cluster-0.1.0.tgz
├── opentelemetry-infra-0.1.0.tgz
├── fhir-0.1.0.tgz
├── quarkus-rest-server-0.1.0.tgz
├── quarkus-cxf-server-0.1.0.tgz
└── charts/partners/hackathon/          # Chart sources
    ├── activemq-artemis/0.1.0/src/
    ├── postgres-cluster/0.1.0/src/
    ├── opentelemetry-infra/0.1.0/src/
    ├── fhir/0.1.0/src/
    ├── quarkus-rest-server/0.1.0/src/
    └── quarkus-cxf-server/0.1.0/src/
```

Each chart source contains:
- `Chart.yaml`: Chart metadata (version 0.1.0)
- `values.yaml`: Configurable instance name
- `templates/*.yaml`: CR definition template
- `README.md`: Chart-specific documentation

## Common Commands

### Direct Manifest Deployment

Deploy services using direct YAML manifests:

```bash
# ActiveMQ Artemis (requires AMQ Broker Operator)
oc create -f activemq/broker.yaml

# PostgreSQL (requires Crunchy Data PostgreSQL Operator)
oc create -f postgres/postgres.yaml

# Observability - Tempo only (requires Tempo Operator)
oc create -f observability/tempo.yaml

# Observability - Full OpenTelemetry stack (requires Tempo + OpenTelemetry Operators)
oc create -f observability/otel.yaml

# FHIR - Deploy all three variants
oc apply -f fhir/

# FHIR - Deploy individual instances
oc create -f fhir/fhir-dstu3-auth.yaml      # FHIR DSTU3 with authentication (admin/admin)
oc create -f fhir/fhir-dstu3-noauth.yaml    # FHIR DSTU3 without authentication
oc create -f fhir/fhir-r4-default.yaml      # FHIR R4 without authentication

# Quarkus REST Server
oc create -f quarkus-rest/quarkus-rest-server.yaml

# Quarkus CXF Server
oc create -f quarkus-cxf/quarkus-cxf-server.yaml
```

### Helm Chart Deployment

Add the Helm repository hosted on GitHub:

```bash
helm repo add hackathon https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
helm repo update
```

Install charts:

```bash
# ActiveMQ Artemis
helm install artemis-broker hackathon/activemq-artemis -n <namespace>

# PostgreSQL
helm install postgres hackathon/postgres-cluster -n <namespace>

# Observability - Full OpenTelemetry Infrastructure (recommended)
helm install otel-stack hackathon/opentelemetry-infra -n <namespace>

# FHIR (default: R4, no auth)
helm install fhir hackathon/fhir -n <namespace>

# Quarkus REST Server
helm install rest-server hackathon/quarkus-rest-server -n <namespace>

# Quarkus CXF Server
helm install cxf-server hackathon/quarkus-cxf-server -n <namespace>
```

Customize instance names:

```bash
helm install my-broker hackathon/activemq-artemis --set broker.name=my-artemis-broker -n <namespace>
helm install my-db hackathon/postgres-cluster --set cluster.name=my-postgres -n <namespace>
helm install my-otel hackathon/opentelemetry-infra --set tempo.name=my-tempo --set otelCollector.tenantId=production -n <namespace>
helm install my-fhir hackathon/fhir --set fhir.name=my-fhir-server -n <namespace>
helm install my-rest hackathon/quarkus-rest-server --set restServer.name=my-quarkus-rest -n <namespace>
helm install my-cxf hackathon/quarkus-cxf-server --set cxfServer.name=my-quarkus-cxf -n <namespace>
```

FHIR-specific customization:

```bash
# FHIR DSTU3 with authentication
helm install fhir hackathon/fhir \
  --set fhir.version=DSTU3 \
  --set fhir.auth.enabled=true \
  --set fhir.auth.username=admin \
  --set fhir.auth.password=admin \
  -n <namespace>

# FHIR R4 with custom resources
helm install fhir hackathon/fhir \
  --set fhir.version=R4 \
  --set fhir.resources.limits.memory=4Gi \
  --set fhir.resources.limits.cpu=2000m \
  -n <namespace>
```

### OpenShift HelmChartRepository

Register the repository in OpenShift for GUI access:

```bash
oc apply -f hackathon/openshift-charts-repo.yaml
oc get helmchartrepositories
```

After registration, charts appear in OpenShift Developer Console under **+Add** → **Helm Chart**.

### Service-Specific Operations

**ActiveMQ Artemis**: Get broker connection URL after deployment:

```bash
kubectl exec artemis-broker-ss-0 --container artemis-broker-container -- amq-broker/bin/artemis address show
# Output contains: tcp://artemis-broker-ss-0.artemis-broker-hdls-svc.<namespace>.svc.cluster.local:61616
```

**PostgreSQL**: Connect to database pod and configure:

```bash
# Connect to master pod
oc rsh $(oc get pods -l postgres-operator.crunchydata.com/role=master -o name)

# Grant privileges
psql -U postgres test -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO postgresadmin;"

# Change admin password
psql -U postgres -c "ALTER USER postgresadmin PASSWORD '<admin_pwd>';"

# Create test table and data
psql -U postgres test -c "CREATE TABLE test (data TEXT PRIMARY KEY); INSERT INTO test(data) VALUES ('hello'), ('world');"

# Verify
psql -U postgres test -c "SELECT * from test;"
```

**FHIR**: Get route URLs and test endpoints:

```bash
# Get route URLs
oc get route fhir-dstu3-auth -o jsonpath='{.spec.host}'
oc get route fhir-dstu3-noauth -o jsonpath='{.spec.host}'
oc get route fhir-r4-default -o jsonpath='{.spec.host}'

# Test FHIR metadata endpoint (no auth)
FHIR_URL=$(oc get route fhir-r4-default -o jsonpath='{.spec.host}')
curl https://$FHIR_URL/fhir/metadata

# Test with authentication
FHIR_AUTH_URL=$(oc get route fhir-dstu3-auth -o jsonpath='{.spec.host}')
curl -u admin:admin https://$FHIR_AUTH_URL/fhir/metadata
```

**OpenTelemetry Infrastructure**: Enable auto-instrumentation for Java/Camel applications:

```bash
# Add annotation to your Deployment to enable automatic instrumentation
oc patch deployment <your-app-deployment> -n <namespace> -p '
spec:
  template:
    metadata:
      annotations:
        instrumentation.opentelemetry.io/inject-java: "true"'

# Or add directly in your deployment YAML:
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
        image: my-app:latest

# View traces in OpenShift Console: Observe → Traces
# Select the Tempo instance (e.g., "monolitic")

# Check collector logs
oc logs deployment/otel-collector -n <namespace>

# Verify post-install job
oc logs job/otel-post-install -n <namespace>
```

**Quarkus REST Server**: Access OpenAPI and APIs

```bash
# Get route URL
ROUTE_URL=$(oc get route quarkus-rest-server -o jsonpath='{.spec.host}')

# Access OpenAPI spec
curl https://$ROUTE_URL/q/openapi

# Access Swagger UI in browser
echo "Swagger UI: https://$ROUTE_URL/q/swagger-ui"

# List fruits
curl https://$ROUTE_URL/fruits

# Add a fruit
curl -X POST https://$ROUTE_URL/fruits \
  -H "Content-Type: application/json" \
  -d '{"name": "Banana", "description": "Tropical fruit"}'

# Test slow service (1-second delay)
curl https://$ROUTE_URL/slow/World

# Test faulty service (throws exception)
curl https://$ROUTE_URL/faulty/World

# Upload a file
curl -X POST https://$ROUTE_URL/multipart/upload \
  -F "file=@/path/to/file.txt"
```

**Quarkus CXF Server**: Access SOAP services and WSDLs

```bash
# Get route URL
ROUTE_URL=$(oc get route quarkus-cxf-server -o jsonpath='{.spec.host}')

# Access WSDL files
echo "HelloService WSDL: https://$ROUTE_URL/soap/hello?wsdl"
echo "FruitService WSDL: https://$ROUTE_URL/soap/fruits?wsdl"
echo "FaultyHelloService WSDL: https://$ROUTE_URL/soap/faulty-hello?wsdl"
echo "SlowHelloService WSDL: https://$ROUTE_URL/soap/SlowHelloServiceImpl?wsdl"
echo "MtomService WSDL: https://$ROUTE_URL/soap/mtom?wsdl"

# Call HelloService
curl -X POST https://$ROUTE_URL/soap/hello \
  -H "Content-Type: text/xml" \
  -d '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:hel="http://server.it.cxf.quarkiverse.io/">
   <soapenv:Body>
      <hel:hello>
         <arg0>World</arg0>
      </hel:hello>
   </soapenv:Body>
</soapenv:Envelope>'

# Call FruitService - List Fruits
curl -X POST https://$ROUTE_URL/soap/fruits \
  -H "Content-Type: text/xml" \
  -d '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://server.it.cxf.quarkiverse.io/">
   <soapenv:Body>
      <ser:list/>
   </soapenv:Body>
</soapenv:Envelope>'
```

## Prerequisites

The following services require operators to be installed before deployment:

1. **AMQ Broker Operator** (for ActiveMQ Artemis)
2. **Crunchy Data PostgreSQL Operator** (for PostgreSQL)
3. **Tempo Operator** (for Tempo and OpenTelemetry Infrastructure)
4. **OpenTelemetry Operator** (for full OpenTelemetry Infrastructure stack only)

Install operators via OpenShift OperatorHub or using operator subscriptions.

**FHIR**, **Quarkus REST Server**, and **Quarkus CXF Server** do not require any operators - they use standard Kubernetes resources.

## Key Configuration Details

**ActiveMQ Artemis** (`activemq/broker.yaml`):
- Minimal configuration with only instance name
- Operator handles all deployment details

**PostgreSQL** (`postgres/postgres.yaml`):
- PostgreSQL version: 17
- Auto-create user schema enabled
- User: `admin` with database `test`
- Single instance with 1Gi storage
- pgBackRest backup with 1Gi storage

**Tempo Only** (`observability/tempo.yaml`):
- Monolithic deployment mode
- OTLP ingestion enabled (gRPC and HTTP)
- Multi-tenancy enabled with OpenShift mode
- Tenant: `application`
- Memory-backed trace storage
- Resource limits: 2 CPU, 2Gi memory
- Jaeger UI disabled

**OpenTelemetry Infrastructure** (`observability/otel.yaml`):
- Complete observability stack with 7 components
- Includes: Tempo + OpenTelemetry Collector + Auto-instrumentation + RBAC + ServiceMonitor
- OpenTelemetry Collector configuration:
  - OTLP receivers (gRPC + HTTP)
  - Exports to Tempo via OTLP
  - Prometheus metrics endpoint on port 8889
  - Batch processing: 500 records, 10s timeout
- Auto-instrumentation for Java/Camel applications:
  - Java image: `ghcr.io/open-telemetry/opentelemetry-operator/autoinstrumentation-java:2.22.0`
  - Sampling: `always_on`
  - JMX target: Camel
  - Enabled via annotation: `instrumentation.opentelemetry.io/inject-java: "true"`
- RBAC with ClusterRoles for trace read/write access
- Helm hooks ensure proper deployment order (weights 0-5)

**FHIR** (`fhir/*.yaml`):
- Three deployment variants available:
  - `fhir-dstu3-auth.yaml`: FHIR DSTU3 with basic authentication (admin/admin)
  - `fhir-dstu3-noauth.yaml`: FHIR DSTU3 without authentication
  - `fhir-r4-default.yaml`: FHIR R4 without authentication
- Each deployment includes Service, Deployment, and Route resources
- Resource limits: 2Gi memory, 1000m CPU
- Resource requests: 512Mi memory, 250m CPU
- Health probes on `/fhir/metadata` endpoint
- Base path: `/fhir`
- Startup time: 30-60 seconds for bulk job initialization

**Quarkus REST Server** (`quarkus-rest/quarkus-rest-server.yaml`):
- Framework: Quarkus 3.30.0
- Port: 8080
- Health endpoints: `/q/health/live`, `/q/health/ready`
- OpenAPI spec: `/q/openapi`
- Swagger UI: `/q/swagger-ui`
- Resource limits: 2Gi memory, 1000m CPU
- Resource requests: 512Mi memory, 250m CPU
- Services: Fruit CRUD, Slow (1s delay), Faulty (exception handling), Multipart upload/download
- Image: `quay.io/fuse_qe/quarkus-rest-server:1.0.0`

**Quarkus CXF Server** (`quarkus-cxf/quarkus-cxf-server.yaml`):
- Framework: Quarkus 3.30.0, Quarkus CXF 3.30.0
- Port: 8080
- Base path: `/soap`
- Health endpoints: `/q/health/live`, `/q/health/ready`
- Resource limits: 2Gi memory, 1000m CPU
- Resource requests: 512Mi memory, 250m CPU
- SOAP Services:
  - HelloService: `/soap/hello?wsdl`
  - FruitService: `/soap/fruits?wsdl`
  - FaultyHelloService: `/soap/faulty-hello?wsdl`
  - SlowHelloService: `/soap/SlowHelloServiceImpl?wsdl`
  - MtomService: `/soap/mtom?wsdl`
- Image: `quay.io/fuse_qe/quarkus-cxf-server:1.0.0`
- Pretty-print logging enabled for HelloService and FruitService

## Modifying Charts

When updating Helm charts:

1. Edit chart sources in `hackathon/charts/partners/hackathon/<chart-name>/0.1.0/src/`
2. Package the updated chart:
   ```bash
   helm package hackathon/charts/partners/hackathon/<chart-name>/0.1.0/src/ -d hackathon/
   ```
3. Regenerate the repository index:
   ```bash
   helm repo index hackathon/ --url https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
   ```
4. Commit all changes including the `.tgz` file and updated `index.yaml`

## Repository Hosting

The Helm repository is hosted directly from GitHub using raw content URLs. The HelmChartRepository points to:
```
https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
```

This allows OpenShift to fetch the `index.yaml` and chart packages directly from the `main` branch.
