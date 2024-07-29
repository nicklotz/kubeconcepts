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

## D. ConfigMaps and Secrets

> ConfigMaps are Kubernetes resources designed to store *non-sensitive*, *structured* data.

> Secrets are Kubernetes resources designed to store *sensitive* data.

> By default, both are stored in an unencrypted key-value store called **etcd**.

> It is strongly recommended to use a third party service like Vault that stores secrets encrypted at rest.

1. First, let's create a ConfigMap from literal, passed-in values.

```
kubectl create configmap myliteralappconfig --from-literal=app.color=hazel --from-literal=app.env=production
```
```
kubectl get configmaps myliteralappconfig -o yaml
```

2. You can also create a ConfigMap from a file.
```
mkdir ~/myappconfigmapfile
```
```
cd ~/myappconfigmapfile
```
```
cat << EOF > app.properties
background=colorful
server=nginxprod
EOF
```
```
kubectl create configmap myappfileconfigmap --from-file=app.properties
```
```
kubectl describe configmaps myappfileconfigmap
```

3. Next, we can create a deployment that uses a ConfigMap.
```
mkdir ~/mydeploymentwithconfigmap
```
```
cd ~/mydeploymentwithconfigmap
```
```
cat << EOF > mydeploymentwithconfigmap.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: nginx
        env:
          - name: APP_COLOR
            valueFrom:
              configMapKeyRef:
                name: myliteralappconfig
                key: app.color
          - name: APP_ENV
            valueFrom:
              configMapKeyRef:
                name: myliteralappconfig
                key: app.env
      restartPolicy: Always
EOF
```
```
kubectl apply -f mydeploymentwithconfigmap.yaml
```
```
kubectl describe mydeploymentwithconfigmap.yaml
```

4. Now let's work with secrets. Create a decode an application secret.
```
kubectl create secret generic appsecret --from-literal=username=admin --from-literal=password=secret
```
```
kubectl get secrets
```
```
kubectl get secret appsecret -o jsonpath="{.data.username}" | base64 --decode
echo
```
```
kubectl get secret appsecret -o jsonpath="{.data.password}" | base64 --decode
echo
```









