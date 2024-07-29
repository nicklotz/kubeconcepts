# Kubernetes Volumes, ConfigMaps, and Secrets

> We'll begin our storage discussion with walking through the several different volume types supported in Kubernetes.

## A. Temporary Working Volumes

> A volume of type `emptyDir` exists only for the life of a pod and is intended to be temporary working space.

1. Create a new pod manifest.
```
mkdir ~/myemptydirpod/
```
```
cd ~/myemptydirpod/
```
```
cat << EOF > myemptydirpod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: test-container
    image: nginx
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
EOF
```

2. Apply and inspect the configuration.
```
kubectl apply -f myemptydirpod.yaml
```
```
kubectl describe pod test-pod
```

3. Clean up.
```
kubectl delete -f myemptydirpod.yaml
```

## B. HostPath Volume

> A HostPath volume mounts a directory from the host node on which a pod is running.

1. Create a new pod manifest.
```
mkdir ~/myhostpathpod/
```
```
cd ~/myhostpathpod/
```
```
cat << EOF > myhostpathpod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - name: test-container
    image: nginx
    volumeMounts:
    - mountPath: /data
      name: host-data
  volumes:
  - name: host-data
    hostPath:
      path: /mnt/data
      type: DirectoryOrCreate
EOF
```

2. Apply and inspect the configuration.
```
kubectl apply -f myhostpathpod.yaml
```
```
kubectl describe pod hostpath-pod
```

3. Clean up.
```
kubectl delete -f myhostpathpod.yaml
```

## C. Persistent Volumes and Persistent Volume Claims

> A persistent volume allocates a chunk of storage. A persistent volume claim tells Kubernetes that a pod or deployment is claiming a PV, so no one else can.

> Let's create a unified NGINX deployemnt that writes to persistent storage.

1. Create a populate a unified manifest.
```
mkdir ~/myunifiednginx/
```
```
cd ~/myunifiednginx/
```
```
cat << EOF > myunifiednginx.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nginx-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-path
  hostPath:
    path: /mnt/nginx

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 1Gi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
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
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-storage
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nginx-storage
        persistentVolumeClaim:
          claimName: nginx-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30007
    protocol: TCP
  selector:
    app: nginx
EOF
```

2. Apply the manifest.
```
kubectl apply -f myunifiednginx.yaml
```

3. Check the provisioned resources.
```
kubectl get pv
```
```
kubectl get pvc
```
```
kubectl get deployments
```
```
kubectl get pods
```
```
kubectl get svc
```

4. Clean up.
```
kubectl delete -f myunifiednginx.yaml
```
```
kubectl delete pvc --all
```
```
kubectl delete pv --all
```
