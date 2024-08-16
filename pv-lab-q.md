# Khalid Mohamed Mahmoud Salman 
# ID: 21000070


## pv-lab-q

### 1. Create the Pod from the Provided YAML File

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - env:
    - name: LOG_HANDLERS
      value: file
    image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: event-simulator
```

```bash
kubectl apply -f webapp-pod.yaml
```

### 2. Configure a Volume to Store Logs at `/var/log/webapp` on the Host

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - env:
    - name: LOG_HANDLERS
      value: file
    image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: event-simulator
    volumeMounts:
    - name: webapp-log-volume
      mountPath: /log
  volumes:
  - name: webapp-log-volume
    hostPath:
      path: /var/log/webapp
```

```bash
kubectl apply -f webapp-pod.yaml
```

### 3. Create a Persistent Volume with the Given Specification

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/log
  persistentVolumeReclaimPolicy: Retain
```

```bash
kubectl apply -f pv-log.yaml
```

### 4. Create a Persistent Volume Claim with the Given Specification

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

```bash
kubectl apply -f pvc-log.yaml
```

### 5. Check the State of the Persistent Volume Claim

```bash
kubectl get pvc claim-log-1
```

### 6. Check the State of the Persistent Volume

```bash
kubectl get pv pv-log
```

### 7. Why is the Claim Not Bound to the Available Persistent Volume?

The Persistent Volume Claim is not bound to the available Persistent Volume because the access modes are incompatible. The PVC is requesting `ReadWriteOnce` (RWO), while the PV supports `ReadWriteMany` (RWX). For a claim to bind to a PV, their access modes must be compatible.

### 8. Update the Access Mode on the Claim to Bind it to the PV

```bash
kubectl delete pvc claim-log-1
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany  # Updated to match PV's access mode
  resources:
    requests:
      storage: 50Mi
```

```bash
kubectl apply -f pvc-log.yaml
```

### Recheck Binding

```bash
kubectl get pvc claim-log-1
```