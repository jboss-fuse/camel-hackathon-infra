# camel-hackathon-infra - Artemis Chapter

The operator is installed already, so in your OCP project run.

```
oc create -f broker.yaml
```

Then run

```
kubectl exec artemis-broker-ss-0 --container artemis-broker-container -- amq-broker/bin/artemis address show
```

In this way you'll get the Broker URL

```
+NOTE: Picked up JDK_JAVA_OPTIONS: -Dbroker.properties=/amq/extra/secrets/artemis-broker-props/broker.properties
Connection brokerURL = tcp://artemis-broker-ss-0.artemis-broker-hdls-svc.amq-broker.svc.cluster.local:61616
```

You can then use the URL for connecting to the broker.
