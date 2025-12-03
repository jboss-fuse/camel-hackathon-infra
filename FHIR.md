# FHIR Mock Server

This document provides instructions for deploying and using the HAPI FHIR mock servers for the Camel Hackathon.

## Overview

The FHIR mock servers provide HL7 FHIR-compliant APIs for testing healthcare integration scenarios. Three different configurations are available:

- **FHIR R4 (No Auth)** - Modern FHIR R4 standard without authentication
- **FHIR DSTU3 (Auth)** - FHIR DSTU3 with basic authentication
- **FHIR DSTU3 (No Auth)** - FHIR DSTU3 without authentication

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

#### Step 2: Install FHIR Servers

Choose your namespace and install the FHIR servers:

```bash
# Set your namespace
NAMESPACE=<your-namespace>

# Install FHIR R4 (No Auth) - Default configuration
helm install fhir-r4-default hackathon/fhir \
  --set fhir.name=fhir-r4-default \
  --set fhir.version=R4 \
  --set fhir.auth.enabled=false \
  -n $NAMESPACE

# Install FHIR DSTU3 (Auth)
helm install fhir-dstu3-auth hackathon/fhir \
  --set fhir.name=fhir-dstu3-auth \
  --set fhir.version=DSTU3 \
  --set fhir.auth.enabled=true \
  --set fhir.auth.username=admin \
  --set fhir.auth.password=admin \
  -n $NAMESPACE

# Install FHIR DSTU3 (No Auth)
helm install fhir-dstu3-noauth hackathon/fhir \
  --set fhir.name=fhir-dstu3-noauth \
  --set fhir.version=DSTU3 \
  --set fhir.auth.enabled=false \
  -n $NAMESPACE
```

#### Step 3: Verify Installation

```bash
# Check if pods are running
oc get pods -n $NAMESPACE | grep fhir

# Check if routes are created
oc get routes -n $NAMESPACE | grep fhir
```

### Option 2: Install via Direct YAML Manifests

```bash
# Set your namespace
NAMESPACE=<your-namespace>

# Deploy all FHIR servers at once
oc apply -f fhir/ -n $NAMESPACE

# Or deploy individually
oc create -f fhir/fhir-r4-default.yaml -n $NAMESPACE
oc create -f fhir/fhir-dstu3-auth.yaml -n $NAMESPACE
oc create -f fhir/fhir-dstu3-noauth.yaml -n $NAMESPACE
```

## Retrieving Server URLs

After installation, retrieve the route URLs for your FHIR servers:

```bash
# Set your namespace
NAMESPACE=<your-namespace>

# Get all FHIR routes
echo "FHIR R4 (No Auth):"
oc get route fhir-r4-default -n $NAMESPACE -o jsonpath='https://{.spec.host}/fhir' && echo

echo "FHIR DSTU3 (Auth):"
oc get route fhir-dstu3-auth -n $NAMESPACE -o jsonpath='https://{.spec.host}/fhir' && echo

echo "FHIR DSTU3 (No Auth):"
oc get route fhir-dstu3-noauth -n $NAMESPACE -o jsonpath='https://{.spec.host}/fhir' && echo
```

## Available FHIR Servers

Once deployed, you can access your FHIR servers using the routes. Use the table below as a reference (replace the Server URLs with your actual routes):

| **FHIR Mock Server** | **Auth** | **Server URL** | **FHIR Version** | **User** | **Password** |
|----------------------|----------|----------------|------------------|----------|--------------|
| FHIR R4 Default | No Auth | `https://<your-route>/fhir` | R4 | - | - |
| FHIR DSTU3 Auth | Auth | `https://<your-route>/fhir` | DSTU3 | admin | admin |
| FHIR DSTU3 No Auth | No Auth | `https://<your-route>/fhir` | DSTU3 | - | - |

To get your actual routes, run:

```bash
NAMESPACE=<your-namespace>

echo "| **FHIR Mock Server** | **Auth** | **Server URL** | **FHIR Version** | **User** | **Password** |"
echo "|----------------------|----------|----------------|------------------|----------|--------------|"
echo "| FHIR R4 Default | No Auth | https://$(oc get route fhir-r4-default -n $NAMESPACE -o jsonpath='{.spec.host}')/fhir | R4 | - | - |"
echo "| FHIR DSTU3 Auth | Auth | https://$(oc get route fhir-dstu3-auth -n $NAMESPACE -o jsonpath='{.spec.host}')/fhir | DSTU3 | admin | admin |"
echo "| FHIR DSTU3 No Auth | No Auth | https://$(oc get route fhir-dstu3-noauth -n $NAMESPACE -o jsonpath='{.spec.host}')/fhir | DSTU3 | - | - |"
```
