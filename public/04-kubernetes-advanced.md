# Selected Advanced Kubernetes Topics

## A. Multi-Container Pods
### Summary of design patterns for multi-container pods
| **Pattern**           | **Use Case**                                                        | **Description**                                                                                 |
|-----------------------|---------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|
| **Sidecar**           | Auxiliary tasks (logging, monitoring)                               | One container supports another by providing supplementary features                               |
| **Ambassador**        | Manage external communication                                       | Proxy container manages outbound connections to external services                            |
| **Adapter**           | Standardize or reformat data                                        | Transforms data between containers, adapts input/output formats                              |
| **Init Container**    | Setup tasks before main container starts                            | Runs initialization tasks like configuration or file fetching    |
| **Leader-Follower**   | Coordinate leadership in replicated systems                         | One container takes the lead; others act as backups in case of failure                        |
| **Watcher**           | Respond to configuration or state changes                           | Monitors changes and dynamically updates main container config                |
| **Proxy**             | Isolate external communication complexities                         | Manages traffic control, load balancing, and SSL termination                 |
| **Deployer**          | Perform deployment-related tasks                                    | Manages application deployment tasks such as migrations or dynamic setup|
| **Work Queue**        | Asynchronously process tasks from a queue                           | Containers share and process tasks from a central queue                         |
| **Logging Sidecar**   | Centralized log management                                          | Processes logs from the main container and forwards them to external services.|

1. Create a manifest called **mymulticontainerdeployment**.
```
cat << EOF > mymulticontainerdeployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mymulticontainerdeployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mymulticontainerapp
  template:
    metadata:
      labels:
        app: mymulticontainerapp
    spec:
      containers:
      - name: mynginxcontainer
        image: nginx:latest
        volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
        ports:
        - containerPort: 80
      - name: mylogsidecar
        image: busybox
        args: [/bin/sh, -c, 'while true; do cat /var/log/nginx/access.log; sleep 5; done']
        volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
      volumes:
      - name: shared-logs
        emptyDir: {}
EOF
```

2. Create the deployment.
```
kubectl apply -f mymulticontainerdeployment.yaml
```

3. Observe the deployment and pods.
```
kubectl get deployments
kubectl get pods
```

4. Observe the logs in the sidecar container from one of the pods.
```
kubectl logs $(kubectl get pods -l app=mymulticontainerapp -o jsonpath='{.items[0].metadata.name}') -c mylogsidecar
```

##. Traffic Management with Istio

- Refer to the [Istio getting started guide](https://istio.io/latest/docs/setup/getting-started/) for installing Istio and deploying a sample app.
