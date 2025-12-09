# Lab 02: Kubernetes Controllers and Networking

> Every object in Kubernetes is managed by a **controller** - a control loop that watches the state of your cluster and makes changes to move the current state toward the desired state.

> For example, the NGINX pods in the previous labs are managed by a Deployment controller, which in turn creates a ReplicaSet controller to maintain the desired number of pod replicas.

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
# -o wide shows additional columns including the node where each pod runs
kubectl get pods -o wide
```

> The `-o wide` output shows which node each pod is running on. With a DaemonSet, you should see exactly one pod per node. In our single-node lab setup, you'll see one fluentd pod.

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
      storageClassName: "local-path"
      resources:
        requests:
          storage: 2Gi
EOF
```

> **Persistent Volume Claims (PVCs)** request storage from the cluster. A **Storage Class** defines what type of storage is available (e.g., SSD, network storage). The `local-path` storage class used here provisions storage on the local node's filesystem.

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
```
kubectl delete pvc --all
```

## C. CronJobs

> Kubernetes has a cron scheduler that makes it easy to run tasks on a schedule.

1. Create and change into a new directory to hold a CronJob configuration.
```
mkdir ~/mycronjob/
```
```
cd ~/mycronjob/
```

2. Populate a file containing the CronJob configuration.
```
cat << EOF > mycronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: echo-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: echo
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
EOF
```
3. Apply the configuration.
```
kubectl apply -f mycronjob.yaml
```

4. Watch the task execution.
```
kubectl get jobs --watch
```

5. Clean up resources.
```
kubectl delete -f mycronjob.yaml
```

## D. Networking and Services

> **Services** provide stable network endpoints for accessing pods. Since pods are ephemeral and their IP addresses change, Services provide a consistent DNS name and IP that routes traffic to healthy pod instances.

1. Create a new NGINX deployment.
```
kubectl create deployment nginx --image=nginx
```

2. Expose the NGINX deployment as a service.
```
kubectl expose deployment nginx --port=80 --type=ClusterIP
```

3. Show the service configuration.
```
kubectl get svc nginx
```

4. Scale the NGINX deployment to multiple replicas.
```
kubectl scale deployment nginx --replicas=3
```

5. Test accessing the service via a temporary interactive pod and observe the logs.
```
# --rm automatically removes the pod when you exit
# -it provides interactive terminal access
# -- separates kubectl args from container command
kubectl run -it --rm debug --image=busybox -- sh
```
```
wget -O- nginx
```
> Run the wget command several times, then exit the container and check the logs.
```
exit
```
```
kubectl logs -l app=nginx --tail=10
```

> Which pods are being accessed? What does this say about load balancing?

6. Modify the existing service to a NodePort to expose it externally.
```
# kubectl edit opens the resource in your default editor (usually vi)
# Save and exit to apply changes (:wq in vi)
kubectl edit svc nginx
```
- Find the line `type: ClusterIP` and change it to `type: NodePort`
- Save and exit the editor

7. Check the NodePort configuration.
```
kubectl get svc nginx
```

8. View the service endpoints.
```
kubectl get endpoints nginx
```

9. Create a headless NGINX service.

> **Headless services** (ClusterIP set to `None`) don't provide load balancing. Instead, DNS returns the IP addresses of all pods directly, allowing clients to implement their own load balancing or connect to specific pods.

```
cat << EOF > myheadlessnginxservice.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
EOF
```

10. Apply the configuration.
```
kubectl apply -f myheadlessnginxservice.yaml
```

11. Verify DNS resolution within a pod.
```
kubectl run -it --rm debug --image=busybox -- sh
```
```
nslookup nginx-headless
```
```
exit
```

12. Clean up resources.
```
kubectl delete svc --all
```
```
kubectl delete deployment --all
```




