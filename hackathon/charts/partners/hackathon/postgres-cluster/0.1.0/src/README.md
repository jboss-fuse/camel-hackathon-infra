# PostgreSQL Cluster Helm Chart

A Helm chart for deploying PostgreSQL Cluster on OpenShift using Crunchy Data Operator.

## Description

This chart deploys a PostgreSQL cluster using the Crunchy Data PostgreSQL Operator custom resources.

## Prerequisites

- OpenShift cluster with Crunchy Data PostgreSQL Operator installed
- Helm 3.x

## Installation

```bash
helm install postgres hackathon/postgres-cluster
```

## Configuration

The following table lists the configurable parameters:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `cluster.name` | Name of the PostgreSQL cluster | `hippo` |
| `cluster.postgresVersion` | PostgreSQL version | `17` |
| `cluster.instances[0].storage.size` | Storage size for instance | `1Gi` |
| `cluster.backups.storage.size` | Storage size for backups | `1Gi` |

## Uninstallation

```bash
helm uninstall postgres
```
