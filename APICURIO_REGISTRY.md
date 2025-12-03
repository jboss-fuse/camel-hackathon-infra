# Apicurio Registry Deployment Guide

This guide explains how to deploy Red Hat build of Apicurio Registry on OpenShift with in-memory storage.

## Overview

Apicurio Registry is a datastore for sharing standard event schemas and API designs across API and event-driven architectures. It provides:

- REST API and web console for artifact management
- Support for multiple formats: OpenAPI, AsyncAPI, GraphQL, Apache Avro, Protobuf, JSON Schema, and more
- Schema validation and evolution rules
- Compatibility with Confluent and IBM Schema Registry APIs

This deployment uses **in-memory storage** - perfect for development, testing, and demo environments. Data will be lost when the pod restarts.

## Deployment Options

### Option 1: Direct YAML Manifest

Deploy using the direct YAML manifest from the `apicurio-registry/` directory.

```bash
oc create -f apicurio-registry/apicurio-registry.yaml -n <namespace>
```

### Option 2: Helm Chart

Deploy using the packaged Helm chart.

#### Add the Helm Repository

```bash
helm repo add hackathon https://raw.githubusercontent.com/jboss-fuse/camel-hackathon-infra/refs/heads/main/hackathon/
helm repo update
```

#### Install

```bash
helm install apicurio-registry hackathon/apicurio-registry -n <namespace>
```

#### Customize Instance Name

```bash
helm install my-registry hackathon/apicurio-registry \
  --set registry.name="my-apicurio-registry" \
  -n <namespace>
```

## Accessing Apicurio Registry

### Get the Route URL

After deployment, OpenShift automatically creates a Route for the registry:

```bash
# Get the route URL
REGISTRY_URL=$(oc get route apicurio-registry -n <namespace> -o jsonpath='{.spec.host}')
echo "Apicurio Registry URL: https://$REGISTRY_URL"
```

If you customized the instance name:
```bash
REGISTRY_URL=$(oc get route <your-registry-name> -n <namespace> -o jsonpath='{.spec.host}')
echo "Apicurio Registry URL: https://$REGISTRY_URL"
```

### Access the Web Console (UI Endpoint)

The web console is accessible at the root path of your registry URL:

```bash
# Open in browser
echo "Web Console: https://$REGISTRY_URL/ui"
```

Or visit directly:
```
https://<your-registry-route>/ui
```

The web console provides:
- Artifact browser and search
- Upload and manage artifacts (schemas, APIs)
- View artifact versions and metadata
- Configure validation and compatibility rules
- API documentation

### Access the REST API

The REST API is available at:
```
https://<your-registry-route>/apis/registry/v2
```

Example API calls:

```bash
# Get the route
REGISTRY_URL=$(oc get route apicurio-registry -n <namespace> -o jsonpath='{.spec.host}')

# List all artifacts
curl https://$REGISTRY_URL/apis/registry/v2/groups/default/artifacts

# Create a new artifact (JSON Schema example)
curl -X POST https://$REGISTRY_URL/apis/registry/v2/groups/default/artifacts \
  -H "Content-Type: application/json" \
  -H "X-Registry-ArtifactId: my-schema" \
  -H "X-Registry-ArtifactType: JSON" \
  -d '{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "type": "object",
    "properties": {
      "name": {"type": "string"},
      "age": {"type": "number"}
    }
  }'

# Get artifact metadata
curl https://$REGISTRY_URL/apis/registry/v2/groups/default/artifacts/my-schema/meta

# Get artifact content
curl https://$REGISTRY_URL/apis/registry/v2/groups/default/artifacts/my-schema
```
