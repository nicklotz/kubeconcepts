# Lab 01: Kubernetes Intro

# A. (Optional) Install K3D

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

# B. Core Kubernetes Components

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

# C. Interacting with the Kubernetes API

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



