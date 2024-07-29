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

## B. StatefulSets

> In Kubernetes, **Deployments** are intended to be stateless and lightweight. Each replica pod is meant to be identical and interchangeable.

> Sometimes that's not what we want. For example, databases can be corrupted if there are multiple writable instances. We therefore should use persistent storage and networking so DB services can control which instances are read-write and which are read-only.

1. Create and change into a new directory to hold a StatefulSet configuration.
```
mkdir ~/mysqlstatefulset/
```
```
cd ~/mysqlstatefulset/
```

2. Populate a file containing the StatefulSet configuration.
```
cat << EOF > mysqlstatefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:9.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: mypassword
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "standard"
      resources:
        requests:
          storage: 2Gi
EOF
```

> We haven't yet discussed persistent storage in Kubernetes. In this example, the StatefulSet will assign dedicated storage (called a Persistent Volume Claim) to each pod from whatever Storage Class is available to the cluster.

3. Deploy the MySQL StatefulSet.
```
kubectl apply -f mysqlstatefulset.yaml
```

4. Explore the deployed StatefulSet.
```
kubectl get statefulset
```
```
kubectl get pods -l app=mysql
```
```
kubectl describe statefulset mysql
```

5. Explore provision storage.
```
kubectl get pvc
```

6. Clean up resources.
```
kubectl delete -f mysqlstatefulset.yaml
```



