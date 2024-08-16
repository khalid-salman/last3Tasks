# Khalid Mohamed Mahmoud Salman 
# ID: 21000070


## config-secrets-lab

### 1. List ConfigMaps in the System

```bash
kubectl get configmaps --all-namespaces
```

### 2. List Secrets in the System

```bash
kubectl get secrets --all-namespaces
```

### 3. Create a ConfigMap Named `app-config`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  APP_ENV: production
  LOG_LEVEL: debug
```

```bash
kubectl apply -f app-config.yaml
```

### 4. View Key-Value Pairs in the ConfigMap Named `app-config`

```bash
kubectl get configmap app-config -o yaml
```

```bash
kubectl describe configmap app-config
```

### 5. Create a Secret Named `db-secret`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
  namespace: default
type: Opaque
data:
  username: dXNlcm5hbWU=  # base64 encoded value for 'username'
  password: cGFzc3dvcmQ=  # base64 encoded value for 'password'
```

```bash
kubectl apply -f db-secret.yaml
```

### 6. View Decoded Key-Value Pairs in the Secret Named `db-secret`

```bash
kubectl get secret db-secret -o jsonpath="{.data.username}" | base64 --decode
echo ""
kubectl get secret db-secret -o jsonpath="{.data.password}" | base64 --decode
echo ""
```

### 7. Create a Pod That Uses the ConfigMap and Secret

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  namespace: default
spec:
  containers:
  - name: app-container
    image: busybox
    command: [ "sh", "-c", "env && sleep 3600" ]
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: LOG_LEVEL
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

```bash
kubectl apply -f app-pod.yaml
```

### 8. Verify the Environment Variables in the Pod
```bash
kubectl logs app-pod
```
