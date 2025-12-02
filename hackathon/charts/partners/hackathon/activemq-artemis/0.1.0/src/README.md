# ActiveMQ Artemis Helm Chart

A Helm chart for deploying ActiveMQ Artemis Broker on OpenShift.

## Description

This chart deploys an ActiveMQ Artemis broker using the AMQ Broker Operator custom resources.

## Prerequisites

- OpenShift cluster with AMQ Broker Operator installed
- Helm 3.x

## Installation

```bash
helm install artemis-broker hackathon/activemq-artemis
```

## Configuration

The following table lists the configurable parameters:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `broker.name` | Name of the Artemis broker | `artemis-broker` |
| `broker.apiVersion` | API version for ActiveMQArtemis resource | `broker.amq.io/v1beta1` |

## Uninstallation

```bash
helm uninstall artemis-broker
```
