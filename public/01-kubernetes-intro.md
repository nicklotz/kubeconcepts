# Lab 01: Kubernetes Intro

## A. (Optional) Install K3D

> If you are using Rancher Desktop or Docker Desktop, you should have Kubernetes installed already and can skip to part **B**.

1. Run the K3D installer script.
```
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```

2. Create a custer.
```
k3d cluster create -p "80:80@loadbalancer" -p "443:443@loadbalancer"
```

3. Follow [these instructions](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) to install kubeclt on your OS and architecture.

4. Verify kubectl can communicate with your k3d cluster.
```
kubectl cluster-info
```

## B. Core Kubernetes Components

> The **kube-system** namespace contains the core services that manages Kubernetes itself.

1. Explore the deployments in the **kube-system** namespace.
```
kubectl get deployments -n kube-system
```

2. Explore the services provisioned in the **kube-system** namespace.
```
kubectl get services -n kube-system
```

3. Explore the pods running in the **kube-system** namespace.
```
kubectl get pods -n kube-system
```

## C. Interacting with the Kubernetes API

> The **kubectl command** is the primary means to **imperatively** manage Kubernetes. <br/>
> Later we'll walk through best-practices for declarative resource management.

1. Create a simple NGINX deployment.
```
kubectl create deployment nginx --image=nginx
```

2. Describe the deployment.
```
kubectl describe deployment nginx
```

3. Inspect the deployment at the pod level.
```
kubectl get pods
```

4. Delete the deployment
```
kubectl delete deployment nginx
```

5. Attempt to query the Kubernetes API directly.
```
curl http://localhost:8001/api/
```

> What error do you see? Why do you see it?

5. Create a temporary local gateway to the Kubernetes API that runs in the background.
```
kubectl proxy > /dev/null 2>&1 &
```

6. Verify the proxy is running in the background.
```
jobs
```

7. Query the Kubernetes API directly.
```
curl http://localhost:8001/api/
```

8. Query information about cluster nodes.
```
curl http://localhost:8001/api/v1/nodes
```

9. Query information about cluster namespaces.
```
curl http://localhost:8001/api/v1/namespaces
```

10. Bring the proxy process back to the foreground.
```
fg 1
```

11. Type **CTRL-C** to kill the process.

12. Attempt to query the API again.
```
curl http://localhost:8001/api/
```

## D. Declarative and Desired State

> Previously, we created an *imperative*, ad hoc deployment of NGINX. Kubernetes is designed to be *declarative* in its management.

> **Imperative**: "Do this, then this, then that, in order."

> **Declarative**: "Here is the end result I want. Kubernetes, make it happen"

1. Create a directory to hold your NGINX deployment configuration.
```
mkdir mynginxdeployment/
```
```
cd mynginxdeployment/
```

2. Populate a file containing the NGINX deployment configuration.
```
cat << EOF > mynginxdeployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
EOF
```

> **Deployment** is the type of Kubernetes resource, i.e. a set of one or more pods representing an application workload.

> **Replicas** are how many instances of the deployment pods to run at a time. Important for scaling.

> **Image** is which container image to use for the deployment.

3. Apply the deployment to the cluster.
```
kubectl apply -f mynginxdeployment.yaml
```

4. Explore the deployment and pods.
```
kubectl get deployments nginx-deployment
```
```
kubectl get pods
```
```
kubectl describe deployments nginx-deployment
```

> Why are there two pods running but only one deployment?

## E. Reconciling State

> A key purpose of declarative configuration is to prevent drift. That is, if Kubernetes detects a change, it will perform the necessary work to bring the state of the cluster back to the source of truth, which is our YAML configuration.

1. Manually remove the pods from the NGINX deployment.
```
kubectl delete pod -l app=nginx
```

2. Observe the number of pods in the deployment.
```
watch kubectl get pods
```

> What action(s) did Kubernetes perform in response to the pod being removed?

3. Explore recent cluster events to see how Kubernetes responded.
```
kubectl get events --sort-by='.lastTimestamp'
```

> What happens if you delete just one pod in the deployment? Try it.

## F. Health Checks

> Health checks are ways to ensure how deployments and pods are reachable and working as expected before they can be used.

> **Liveness** probes determine whether a container should be restarted, i.e. whether or not it is "live" or in a deadlock.

> **Rediness** probes determine whether a container is ready to start accepting traffic.

1. Remove the existing deployment.
```
kubectl delete -f mynginxdeployment.yaml
```

2. Modify mynginxdeployment.yaml.
```
cat << EOF >> mynginxdeployment.yaml
        livenessProbe:
          httpGet:
            path: /healthz
            port: 80
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
EOF
```

3. Reapply the deployment with the updated configuration.
```
kubectl apply -f mynginxdeployment.yaml
```

4. Observe the pods.
```
kubectl get pods
```

> How many pods are available and healthy?

5. Look at recent pod events for the deployment.
```
kubectl describe pods -l app=nginx-deployment
```

> Why are the liveness and readiness checks failing? Do the **/healthz** and **/ready** endpoints exist?

6. Modify the liveness and readiness probes to change which paths are queried in the NGINX deployment.
```
sed -i 's/healthz//g' mynginxdeployment.yaml
```
```
sed -i 's/ready//g' mynginxdeployment.yaml
```

7. Confirm the updated content of **mynginxdeployment.yaml**.
```
cat mynginxdeployment.yaml 
```

8. Update the existing deployment.
```
kubectl apply -f mynginxdeployment.yaml
```

9. Watch the current pods and their readiness.
```
watch kubectl get pods
```

> How did Kubernetes go about updating, and healing, the deployment?

10. Clean up resources.
```
kubectl delete -f mynginxdeployment.yaml
```
