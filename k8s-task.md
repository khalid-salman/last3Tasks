# Khalid Mohamed Mahmoud Salman 
# ID: 21000070


## k8s-task

### 1. Create the Python Script

```python
# modify_file.py
file_path = "/mnt/data/output.txt"

with open(file_path, "a") as file:
    file.write("New entry added at every 3-minute interval.\n")
```

### 2. Create a ConfigMap to Store the Script

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: script-configmap
data:
  modify_file.py: |
    file_path = "/mnt/data/output.txt"

    with open(file_path, "a") as file:
        file.write("New entry added at every 3-minute interval.\n")
```

```bash
kubectl apply -f configmap.yaml
```

### 3. Create a Persistent Volume (PV)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

```bash
kubectl apply -f pv.yaml
```

### 4. Create a Persistent Volume Claim (PVC)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```bash
kubectl apply -f pvc.yaml
```

### 5. Create a CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: python-cronjob
spec:
  schedule: "*/3 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: script-runner
            image: python:3.9
            command: ["python", "/scripts/modify_file.py"]
            volumeMounts:
            - name: script-volume
              mountPath: /scripts
            - name: persistent-storage
              mountPath: /mnt/data
          volumes:
          - name: script-volume
            configMap:
              name: script-configmap
          - name: persistent-storage
            persistentVolumeClaim:
              claimName: pvc-claim
          restartPolicy: OnFailure
```

```bash
kubectl apply -f cronjob.yaml
```

### 6. Modify the Python Script

```bash
kubectl apply -f configmap.yaml
```

### 7. Verify the Changes
```bash
kubectl exec -it <pod-name> -- cat /mnt/data/output.txt
```

```bash
kubectl get pods --selector=job-name=<job-name>
```

