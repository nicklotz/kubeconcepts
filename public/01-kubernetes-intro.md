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


