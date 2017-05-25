
# Kafka as Kubernetes StatefulSet

Example of three Kafka brokers depending on five Zookeeper instances.

To get consistent service DNS names `kafka-N.broker.kafka`(`.svc.cluster.local`), run everything in a [namespace](http://kubernetes.io/docs/admin/namespaces/walkthrough/):

## Set up volume claims

You may add [storage class](http://kubernetes.io/docs/user-guide/persistent-volumes/#storageclasses)
to the kafka StatefulSet declaration to enable automatic volume provisioning.

## Set up Zookeeper

There is a Zookeeper+StatefulSet [blog post](http://blog.kubernetes.io/2016/12/statefulset-run-scale-stateful-applications-in-kubernetes.html) and [example](https://github.com/kubernetes/contrib/tree/master/statefulsets/zookeeper),
but it appears tuned for workloads heavier than Kafka topic metadata.

If you lose your zookeeper cluster, kafka will be unaware that persisted topics exist.
The data is still there, but you need to re-create topics.

## Start Kafka

Assuming you have enabled automatic provisioning (see above), go ahead and:

```
kubectl create -f ./
```

You might want to verify in logs that Kafka found its own DNS name(s) correctly. Look for records like:
```
kubectl logs kafka-0 | grep "Registered broker"
# INFO Registered broker 0 at path /brokers/ids/0 with addresses: PLAINTEXT -> EndPoint(kafka-0.broker.kafka.svc.cluster.local,9092,PLAINTEXT)
```

## Testing manually

There's a Kafka pod that doesn't start the server, so you can invoke the various shell scripts.
```
kubectl create -f test/99testclient.yaml
```

See `./test/test.sh` for some sample commands.

## Automated test, while going chaosmonkey on the cluster

This is WIP, but topic creation has been automated. Note that as a [Job](http://kubernetes.io/docs/user-guide/jobs/), it will restart if the command fails, including if the topic exists :(
```
kubectl create -f test/11topic-create-test1.yaml
```

Pods that keep consuming messages (but they won't exit on cluster failures)
```
kubectl create -f test/21consumer-test1.yaml
```

## Teardown & cleanup

Testing and retesting... delete the namespace. PVs are outside namespaces so delete them too.
```
kubectl delete namespace kafka
```
