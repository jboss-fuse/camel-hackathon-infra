# Quarkus CXF Server

This directory provides deployment resources for the Quarkus CXF Server, a demonstration application exposing SOAP web services using Apache CXF.

## Overview

The Quarkus CXF Server provides the following SOAP web services:

- **HelloService**: Simple greeting service (`/soap/hello`)
- **FruitService**: CRUD operations for fruits (`/soap/fruits`)
- **FaultyHelloService**: Demonstrates SOAP fault handling (`/soap/faulty-hello`)
- **SlowHelloService**: Service with 1-second delay (`/soap/SlowHelloServiceImpl`)
- **MtomService**: MTOM for efficient binary data transfer (`/soap/mtom`)

All services are available under the `/soap` base path with pretty-print logging enabled for HelloService and FruitService.

## Deployment

### Prerequisites

- Access to an OpenShift cluster
- `oc` CLI installed and logged in
- Target namespace/project created

### Deploy via Direct YAML

Deploy the Quarkus CXF server:

```bash
# Set your namespace
NAMESPACE=<your-namespace>

# Deploy the CXF server
oc create -f quarkus-cxf/quarkus-cxf-server.yaml -n $NAMESPACE
```

### Verify Deployment

Check deployment status:

```bash
# Check pod status
oc get pods -l app=quarkus-cxf-server -n $NAMESPACE

# Get the route URL
oc get route quarkus-cxf-server -n $NAMESPACE -o jsonpath='{.spec.host}'
```

### Access the Services

Once deployed, retrieve the route URL:

```bash
ROUTE_URL=$(oc get route quarkus-cxf-server -n $NAMESPACE -o jsonpath='{.spec.host}')
echo "CXF Server URL: https://$ROUTE_URL"
```

Access the WSDL files:

- **HelloService**: `https://$ROUTE_URL/soap/hello?wsdl`
- **FruitService**: `https://$ROUTE_URL/soap/fruits?wsdl`
- **FaultyHelloService**: `https://$ROUTE_URL/soap/faulty-hello?wsdl`
- **SlowHelloService**: `https://$ROUTE_URL/soap/SlowHelloServiceImpl?wsdl`
- **MtomService**: `https://$ROUTE_URL/soap/mtom?wsdl`
- **Health Check**: `https://$ROUTE_URL/q/health`

### Example SOAP Calls

**HelloService**:
```bash
curl -X POST https://$ROUTE_URL/soap/hello \
  -H "Content-Type: text/xml" \
  -d '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:hel="http://server.it.cxf.quarkiverse.io/">
   <soapenv:Header/>
   <soapenv:Body>
      <hel:hello>
         <arg0>World</arg0>
      </hel:hello>
   </soapenv:Body>
</soapenv:Envelope>'
```

**FruitService - List Fruits**:
```bash
curl -X POST https://$ROUTE_URL/soap/fruits \
  -H "Content-Type: text/xml" \
  -d '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://server.it.cxf.quarkiverse.io/">
   <soapenv:Header/>
   <soapenv:Body>
      <ser:list/>
   </soapenv:Body>
</soapenv:Envelope>'
```

**FruitService - Add Fruit**:
```bash
curl -X POST https://$ROUTE_URL/soap/fruits \
  -H "Content-Type: text/xml" \
  -d '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://server.it.cxf.quarkiverse.io/">
   <soapenv:Header/>
   <soapenv:Body>
      <ser:add>
         <fruit>
            <name>Banana</name>
            <description>Tropical fruit</description>
         </fruit>
      </ser:add>
   </soapenv:Body>
</soapenv:Envelope>'
```

## Configuration

The deployment uses the following defaults:

| Parameter | Value |
|-----------|-------|
| Container Image | `quay.io/fuse_qe/quarkus-cxf-server:1.0.0` |
| HTTP Port | 8080 |
| Base Path | `/soap` |
| Replicas | 1 |
| Memory Request | 512Mi |
| Memory Limit | 2Gi |
| CPU Request | 250m |
| CPU Limit | 1000m |
| Health Endpoints | `/q/health/live`, `/q/health/ready` |

## Available SOAP Services

| Service | Endpoint | WSDL | Description |
|---------|----------|------|-------------|
| HelloService | `/soap/hello` | `?wsdl` | Simple greeting service |
| FruitService | `/soap/fruits` | `?wsdl` | Fruit CRUD operations |
| FaultyHelloService | `/soap/faulty-hello` | `?wsdl` | SOAP fault handling demo |
| SlowHelloService | `/soap/SlowHelloServiceImpl` | `?wsdl` | Delayed response (1s) |
| MtomService | `/soap/mtom` | `?wsdl` | Binary data with MTOM |

## Uninstall

Remove the deployment:

```bash
oc delete -f quarkus-cxf/quarkus-cxf-server.yaml -n $NAMESPACE
```

## Building Custom Images

If you want to build a custom version of the CXF server:

```bash
# Clone or navigate to the source repository
cd /path/to/quarkus-cxf-server

# Build the container image
mvn clean package \
  -Dquarkus.container-image.build=true \
  -Dquarkus.container-image.group=<your-registry> \
  -Dquarkus.container-image.name=quarkus-cxf-server \
  -Dquarkus.container-image.tag=<your-tag>

# Push to your registry
docker push <your-registry>/quarkus-cxf-server:<your-tag>

# Update the image reference in quarkus-cxf-server.yaml
```

## Technical Details

- **Framework**: Quarkus 3.30.0
- **CXF Version**: Quarkus CXF 3.30.0
- **Runtime**: JVM mode (Java 17)
- **Build Tool**: Maven
- **Dependencies**: quarkus-cxf
- **Health Checks**: Quarkus SmallRye Health (`/q/health/live`, `/q/health/ready`)
- **Logging**: Pretty-print logging enabled for HelloService and FruitService
- **JAXB**: XML binding for Fruit entity
