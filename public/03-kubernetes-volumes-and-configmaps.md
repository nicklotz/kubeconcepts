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
