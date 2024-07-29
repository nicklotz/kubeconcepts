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
      storageClassName: "local-path"
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

> In Kubernetes, Services are resources that expose application networking to other pods in the cluster, or to resources outside the cluster.

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
kubectl run -it --rm debug --image=busybox -- sh wget -O- nginx
```
```
kubectl logs -l app=nginx --tail=10
```

> Which pods are being accessed? What does this say about load balancing?

6. Modify the existing service to a NodePort to expose it externally.
```
kubectl edit svc nginx
```
- Change `type: ClusterIP` to `type: NodePort`

7. Check the NodePort configuration.
```
kubectl get svc nginx
```

8. View the service endpoints.
```
kubectl get endpoints nginx
```

9. Create a headless NGINX service.

> Headless services allow clients to connect to a pod directly. This is accomplished by setting the ClusterIP to `none`.

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
kubectl apply -f nginx-headless.yaml
```

11. Verify DNS resolution within a pod.
```
kubectl run -it --rm debug --image=busybox -- sh
```
```
nslookup nginx-headless
```

12. Clean up resources.
```
kubectl delete svc --all
```
```
kubectl delete deployment --all
```




