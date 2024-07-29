# Lab 02: Kubernetes Controllers and Networking

> Every object in Kubernetes is managed by a **controller** that ensures the object matches its desired state configuration.

> For example, the NGINX pods in the previous labs are managed by a deployment controller, ensuring the resource configurations match the deployment YAML.

> We will now proceed to learn about other types of Kubernetes controllers their corresponding resources.

## A. DaemonSets

> DaemonSets are designed such that a single instance of a particular pod runs on each node in the cluster. These are generally processes that perform some sort of background or system tasks.

> Examples of the tasks performed by DaemonSets include logging, security monitoring, and infrastructure management that is reliant on system heartbeat messages.

> In this exercise we will deploy an open data collector called **fluentd** as a DaemonSet.

1. Create and change into a new configuration directory.
```
mkdir ~/myfluentddaemonset/
```
```
cd ~/myfluentddaemonset
```

2. Populate a file containing the DaemonSet configuration.
```
cat << EOF > myfluentddaemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:latest
EOF
```

3. Apply the configuration to your cluster.
```
kubectl apply -f myfluentddaemonset.yaml
```

4. Inspect the pod(s) that are deployed.
```
kubectl get pods -o wide
```

> The above command is useful for examining that identical **fluentd** pods are deployed to each node. Of course, in our lab setup we have only one node in the cluster.

5. Explore the DaemonSet details
```
kubectl describe daemonset fluentd
```

6. Clean up the resources.
```
kubectl delete -f myfluentddaemonset.yaml
```


