# Apicurio Registry Helm Chart

This Helm chart deploys Red Hat build of Apicurio Registry on OpenShift with in-memory storage.

## Description

Apicurio Registry is a datastore for sharing standard event schemas and API designs across API and event-driven architectures. It provides a REST API and web console to manage artifacts like OpenAPI, AsyncAPI, GraphQL, Apache Avro, Protobuf, JSON Schema, and more.

This chart deploys Apicurio Registry with **in-memory storage** - a simple configuration perfect for development, testing, and demo environments. Note that data will be lost when the pod restarts.

## Prerequisites

- Red Hat build of Apicurio Registry Operator installed in the cluster

## Installation

### Basic Installation

```bash
helm install apicurio-registry hackathon/apicurio-registry -n <namespace>
```

### Customizing Instance Name

```bash
helm install my-registry hackathon/apicurio-registry \
  --set registry.name="my-apicurio-registry" \
  -n <namespace>
```

## Configuration

The following table lists the configurable parameters:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `registry.name` | Name of the Apicurio Registry instance | `apicurio-registry` |
| `registry.deployment.replicas` | Number of replicas (must be 1 for in-memory) | `1` |

## Storage Notes

This chart uses **in-memory storage** which means:
- ✅ No external database or Kafka required
- ✅ Quick setup, perfect for demos and testing
- ⚠️ Data is not persistent - will be lost on pod restart
- ⚠️ Only supports 1 replica

For production use with persistent storage, see the [Apicurio Registry documentation](https://www.apicur.io/registry/docs/apicurio-registry/2.6.x/index.html).

## Uninstallation

```bash
helm uninstall apicurio-registry -n <namespace>
```

## Additional Resources

- [Apicurio Registry Documentation](https://www.apicur.io/registry/)
- [Apicurio Registry GitHub](https://github.com/Apicurio/apicurio-registry)
