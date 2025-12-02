# Grafana Tempo Monolithic Helm Chart

A Helm chart for deploying Grafana Tempo in monolithic mode on OpenShift.

## Description

This chart deploys Grafana Tempo in monolithic mode for distributed tracing using the Tempo Operator custom resources.

## Prerequisites

- OpenShift cluster with Tempo Operator installed
- Helm 3.x

## Installation

```bash
helm install tempo hackathon/tempo-monolithic
```

## Configuration

The following table lists the configurable parameters:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `tempo.name` | Name of the Tempo instance | `monolitic` |
| `tempo.multitenancy.mode` | Multitenancy mode | `openshift` |
| `tempo.resources.limits.cpu` | CPU limit | `2` |
| `tempo.resources.limits.memory` | Memory limit | `2Gi` |
| `tempo.storage.traces.backend` | Traces storage backend | `memory` |

## Uninstallation

```bash
helm uninstall tempo
```
