# SOAP Web Service Server

This document provides instructions for deploying and using the Quarkus CXF SOAP Web Service Server for the Camel Hackathon.

## Overview

The SOAP Web Service Server is a Quarkus-based application using Apache CXF to provide SOAP/WSDL web services for integration scenarios. It demonstrates various SOAP patterns including standard services, fault handling, delayed responses, and MTOM for binary data transfer.

**Source Code**: https://github.com/Croway/quarkus-cxf-server

The server provides five SOAP web services:
- **HelloService**: Simple greeting service
- **FruitService**: CRUD operations for fruits using JAXB
- **FaultyHelloService**: Demonstrates SOAP fault handling
- **SlowHelloService**: Service with 1-second delay for testing timeouts
- **MtomService**: Binary data transfer using MTOM (Message Transmission Optimization Mechanism)

All services are available under the `/soap` base path with pretty-print logging enabled for debugging.

## Installation

### Prerequisites

Ensure you have:
- Access to an OpenShift cluster
- `oc` CLI installed and configured
- `helm` CLI installed (for Helm installation method)

### Option 1: Install via Helm Chart (Recommended)

#### Step 1: Add the Helm Repository

```bash
helm repo add hackathon https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
helm repo update
```

#### Step 2: Install SOAP Web Service Server

Choose your namespace and install:

```bash
# Set your namespace
NAMESPACE=<your-namespace>

# Install with default configuration
helm install quarkus-cxf hackathon/quarkus-cxf-server -n $NAMESPACE

# Or install with custom name
helm install quarkus-cxf hackathon/quarkus-cxf-server \
  --set cxfServer.name=my-soap-service \
  -n $NAMESPACE

# Or install with custom resource limits
helm install quarkus-cxf hackathon/quarkus-cxf-server \
  --set cxfServer.resources.limits.memory=4Gi \
  --set cxfServer.resources.limits.cpu=2000m \
  -n $NAMESPACE
```

#### Step 3: Verify Installation

```bash
# Check if pods are running
oc get pods -l app=quarkus-cxf-server -n $NAMESPACE

# Check if route is created
oc get routes -n $NAMESPACE | grep quarkus-cxf
```

### Option 2: Install via Direct YAML Manifests

```bash
# Set your namespace
NAMESPACE=<your-namespace>

# Deploy the SOAP server
oc create -f quarkus-cxf/quarkus-cxf-server.yaml -n $NAMESPACE
```

## Retrieving Server URL

After installation, retrieve the route URL:

```bash
# Set your namespace
NAMESPACE=<your-namespace>

# Get the SOAP Web Service route
SOAP_URL=$(oc get route quarkus-cxf-server -n $NAMESPACE -o jsonpath='{.spec.host}')
echo "SOAP Web Service URL: https://$SOAP_URL"
```

## Available SOAP Services

Once deployed, you can access the following SOAP services and their WSDLs:

| Service | Endpoint | WSDL | Description |
|---------|----------|------|-------------|
| **MTOM Service** | `/soap/mtom` | `?wsdl` | Binary data transfer with MTOM |
| **Fruit Service** | `/soap/fruits` | `?wsdl` | Fruit CRUD operations (JAXB) |
| **Faulty Hello Service** | `/soap/faulty-hello` | `?wsdl` | SOAP fault handling demo |
| **Slow Hello Service** | `/soap/SlowHelloServiceImpl` | `?wsdl` | Delayed response (1 second) |
| **Hello Service** | `/soap/hello` | `?wsdl` | Simple greeting service |

### WSDL URLs

```bash
# Get route URL
SOAP_URL=$(oc get route quarkus-cxf-server -n $NAMESPACE -o jsonpath='{.spec.host}')

# MTOM Service WSDL
echo "MTOM WSDL: https://$SOAP_URL/soap/mtom?wsdl"

# Fruit Service WSDL
echo "Fruits WSDL: https://$SOAP_URL/soap/fruits?wsdl"

# Faulty Hello Service WSDL
echo "Faulty Hello WSDL: https://$SOAP_URL/soap/faulty-hello?wsdl"

# Slow Hello Service WSDL
echo "Slow Hello WSDL: https://$SOAP_URL/soap/SlowHelloServiceImpl?wsdl"

# Hello Service WSDL
echo "Hello WSDL: https://$SOAP_URL/soap/hello?wsdl"
```

**Example WSDL URLs**:
```
https://quarkus-cxf-server-camel-hackathon.camel-dev-eu-de-1-bx2-4x1-b0521fcfdb6f3f868b0758fbda095bbe-0000.eu-de.containers.appdomain.cloud/soap/mtom?wsdl
https://quarkus-cxf-server-camel-hackathon.camel-dev-eu-de-1-bx2-4x1-b0521fcfdb6f3f868b0758fbda095bbe-0000.eu-de.containers.appdomain.cloud/soap/fruits?wsdl
https://quarkus-cxf-server-camel-hackathon.camel-dev-eu-de-1-bx2-4x1-b0521fcfdb6f3f868b0758fbda095bbe-0000.eu-de.containers.appdomain.cloud/soap/faulty-hello?wsdl
https://quarkus-cxf-server-camel-hackathon.camel-dev-eu-de-1-bx2-4x1-b0521fcfdb6f3f868b0758fbda095bbe-0000.eu-de.containers.appdomain.cloud/soap/SlowHelloServiceImpl?wsdl
```

## Usage Examples

### HelloService - Simple Greeting

**SOAP Request**:
```bash
curl -X POST https://$SOAP_URL/soap/hello \
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

**SOAP Response**:
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
   <soap:Body>
      <ns2:helloResponse xmlns:ns2="http://server.it.cxf.quarkiverse.io/">
         <return>Hello World!</return>
      </ns2:helloResponse>
   </soap:Body>
</soap:Envelope>
```

### FruitService - CRUD Operations

**List All Fruits**:
```bash
curl -X POST https://$SOAP_URL/soap/fruits \
  -H "Content-Type: text/xml" \
  -d '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://server.it.cxf.quarkiverse.io/">
   <soapenv:Header/>
   <soapenv:Body>
      <ser:list/>
   </soapenv:Body>
</soapenv:Envelope>'
```

**Add a Fruit**:
```bash
curl -X POST https://$SOAP_URL/soap/fruits \
  -H "Content-Type: text/xml" \
  -d '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://server.it.cxf.quarkiverse.io/">
   <soapenv:Header/>
   <soapenv:Body>
      <ser:add>
         <fruit>
            <name>Banana</name>
            <description>A yellow tropical fruit</description>
         </fruit>
      </ser:add>
   </soapenv:Body>
</soapenv:Envelope>'
```

**Delete a Fruit**:
```bash
curl -X POST https://$SOAP_URL/soap/fruits \
  -H "Content-Type: text/xml" \
  -d '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://server.it.cxf.quarkiverse.io/">
   <soapenv:Header/>
   <soapenv:Body>
      <ser:delete>
         <fruit>
            <name>Banana</name>
            <description>A yellow tropical fruit</description>
         </fruit>
      </ser:delete>
   </soapenv:Body>
</soapenv:Envelope>'
```

### SlowHelloService - Testing Timeouts

This service introduces a 1-second delay before responding:

```bash
curl -X POST https://$SOAP_URL/soap/SlowHelloServiceImpl \
  -H "Content-Type: text/xml" \
  -d '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:hel="http://server.it.cxf.quarkiverse.io/">
   <soapenv:Header/>
   <soapenv:Body>
      <hel:hello>
         <arg0>Patient Client</arg0>
      </hel:hello>
   </soapenv:Body>
</soapenv:Envelope>'
```

Returns after 1 second: `"Hello Slow Patient Client!"`

### FaultyHelloService - SOAP Fault Handling

This service always throws a SOAP fault to demonstrate error handling:

```bash
curl -X POST https://$SOAP_URL/soap/faulty-hello \
  -H "Content-Type: text/xml" \
  -d '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:hel="http://server.it.cxf.quarkiverse.io/">
   <soapenv:Header/>
   <soapenv:Body>
      <hel:faultyHello>
         <arg0>Test</arg0>
      </hel:faultyHello>
   </soapenv:Body>
</soapenv:Envelope>'
```

Returns a SOAP fault with `GreetingException` details.

## Configuration

The deployment uses the following default configuration:

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

### Customizing the Deployment

You can customize the deployment using Helm values:

```bash
# Custom instance name
helm install my-soap hackathon/quarkus-cxf-server \
  --set cxfServer.name=my-custom-soap \
  -n $NAMESPACE

# Custom resource limits
helm install my-soap hackathon/quarkus-cxf-server \
  --set cxfServer.resources.limits.memory=4Gi \
  --set cxfServer.resources.limits.cpu=2000m \
  -n $NAMESPACE

# Custom replica count
helm install my-soap hackathon/quarkus-cxf-server \
  --set cxfServer.replicas=3 \
  -n $NAMESPACE
```

## Integration with Apache Camel

The SOAP Web Service can be easily integrated with Apache Camel routes using the CXF component:

### Simple SOAP Client Example

```java
// Camel route calling the HelloService
from("direct:callSoapHello")
    .to("cxf://https://" + soapUrl + "/soap/hello"
        + "?serviceClass=io.quarkiverse.cxf.it.server.HelloService"
        + "&wsdlURL=https://" + soapUrl + "/soap/hello?wsdl");
```

### FruitService Integration with Proxy

```java
// Create a CXF proxy for the FruitService
from("direct:listFruits")
    .to("cxf://https://" + soapUrl + "/soap/fruits"
        + "?serviceClass=io.quarkiverse.cxf.it.server.FruitService"
        + "&wsdlURL=https://" + soapUrl + "/soap/fruits?wsdl"
        + "&dataFormat=POJO");
```

### MTOM File Transfer Example

```java
// Upload binary data using MTOM
from("direct:uploadWithMtom")
    .to("cxf://https://" + soapUrl + "/soap/mtom"
        + "?serviceClass=io.quarkiverse.cxf.it.server.MtomService"
        + "&mtomEnabled=true");
```

### Error Handling with Faulty Service

```java
// Handle SOAP faults
from("direct:testFaultHandling")
    .doTry()
        .to("cxf://https://" + soapUrl + "/soap/faulty-hello")
    .doCatch(org.apache.cxf.binding.soap.SoapFault.class)
        .log("Caught SOAP Fault: ${exception.message}")
        .setBody(constant("Fault handled gracefully"))
    .end();
```

## Uninstallation

### Uninstall Helm Release

```bash
helm uninstall quarkus-cxf -n $NAMESPACE
```

### Uninstall Direct YAML

```bash
oc delete -f quarkus-cxf/quarkus-cxf-server.yaml -n $NAMESPACE
```

## Troubleshooting

### Check Pod Status

```bash
oc get pods -l app=quarkus-cxf-server -n $NAMESPACE
oc describe pod <pod-name> -n $NAMESPACE
```

### Check Logs

```bash
oc logs -l app=quarkus-cxf-server -n $NAMESPACE
```

### Verify Health Endpoints

```bash
# Liveness probe
curl https://$SOAP_URL/q/health/live

# Readiness probe
curl https://$SOAP_URL/q/health/ready

# Overall health
curl https://$SOAP_URL/q/health
```

### Verify WSDL Accessibility

```bash
# Test WSDL endpoints
curl https://$SOAP_URL/soap/hello?wsdl
curl https://$SOAP_URL/soap/fruits?wsdl
curl https://$SOAP_URL/soap/mtom?wsdl
```

## SOAP Service Details

### HelloService

- **Endpoint**: `/soap/hello`
- **Operation**: `hello(String text)`
- **Returns**: `"Hello {text}!"`
- **Features**: Pretty-print logging enabled

### FruitService

- **Endpoint**: `/soap/fruits`
- **Operations**:
  - `list()`: Returns all fruits
  - `add(Fruit fruit)`: Adds a new fruit
  - `delete(Fruit fruit)`: Removes a fruit
- **Data Binding**: JAXB for Fruit entity
- **Features**: Pretty-print logging enabled

### FaultyHelloService

- **Endpoint**: `/soap/faulty-hello`
- **Operation**: `faultyHello(String text)`
- **Behavior**: Always throws `GreetingException`
- **Purpose**: Testing SOAP fault handling

### SlowHelloService

- **Endpoint**: `/soap/SlowHelloServiceImpl`
- **Operation**: `hello(String text)`
- **Returns**: `"Hello Slow {text}!"` after 1 second
- **Purpose**: Testing timeout scenarios

### MtomService

- **Endpoint**: `/soap/mtom`
- **Operation**: `echoDataHandler(DHRequest request)`
- **Features**:
  - MTOM enabled for efficient binary transfer
  - SOAP handler validates XOP attachments
  - Supports large binary payloads

## Technical Details

- **Framework**: Quarkus 3.30.0
- **CXF Version**: Quarkus CXF 3.30.0
- **Runtime**: JVM mode (Java 17)
- **Build Tool**: Maven
- **Dependencies**: quarkus-cxf
- **Health Checks**: Quarkus SmallRye Health
- **Logging**: Pretty-print logging for HelloService and FruitService
- **Route**: OpenShift Route with edge TLS termination

## Additional Resources

- Source Code: https://github.com/Croway/quarkus-cxf-server
- Quarkus CXF Documentation: https://docs.quarkiverse.io/quarkus-cxf/dev/
- Apache CXF Documentation: https://cxf.apache.org/
- SOAP 1.2 Specification: https://www.w3.org/TR/soap12/
