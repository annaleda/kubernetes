# CKAD Practice Labs

---

# Q1 — PersistentVolume, PVC e Pod

## Task

Create a **PersistentVolume** named `log-volume` with the following specifications:

* **StorageClassName:** `manual` (already created — do not create it again)
* **AccessModes:** `ReadWriteMany (RWX)`
* **Capacity:** `1Gi`
* **hostPath:** `/opt/volume/nginx`

Next, create a **PersistentVolumeClaim** named `log-claim` that:

* Requests at least **200Mi** of storage
* Uses the storage class `manual`

Finally, create a **Pod** named `logger` that:

* Uses the image `nginx:alpine`
* Mounts the claim at `/var/www/nginx`

---

## Solution

### PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: manual
  hostPath:
    path: /opt/volume/nginx
```

---

### PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-claim
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: manual
  resources:
    requests:
      storage: 200Mi
```

---

### Verify Binding

```bash
kubectl get pv,pvc
```

---

### Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logger
  labels:
    run: logger
spec:
  containers:
  - name: logger
    image: nginx:alpine
    volumeMounts:
    - name: log
      mountPath: /var/www/nginx
  volumes:
  - name: log
    persistentVolumeClaim:
      claimName: log-claim
```

---

# Q2 — Troubleshoot NetworkPolicy

## Task

Existing resources:

* Pod: `secure-pod`
* Service: `secure-service`
* Pod: `webapp-color`

Currently, **both incoming and outgoing connections fail**.

Your task is to fix the configuration so that:

```
webapp-color → secure-pod (TCP 80)
```

works correctly.

---

## Solution

Incoming or outgoing connections are not working because of a **default-deny NetworkPolicy**.

Create a NetworkPolicy allowing traffic from `webapp-color`.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: secure-pod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: webapp-color
    ports:
    - protocol: TCP
      port: 80
```

---

### Test connectivity

```bash
kubectl exec -it webapp-color -- sh
```

Then inside the pod:

```bash
nc -v -z -w 5 secure-service 80
```

---

# Q3 — ConfigMap + Logging Pod

## Task

Create a namespace:

```
dvl1987
```

Create a **ConfigMap** named:

```
time-config
```

with:

```
TIME_FREQ=10
```

Create a **Pod** named `time-check` that:

* Uses the image `busybox`
* Runs the command:

```
while true; do date; sleep $TIME_FREQ; done
```

* Writes output to:

```
/opt/time/time-check.log
```

* Mounts a volume at `/opt/time`

---

## Solution

### Create Namespace

```bash
kubectl create namespace dvl1987
```

---

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: time-config
  namespace: dvl1987
data:
  TIME_FREQ: "10"
```

---

### Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: time-check
  namespace: dvl1987
  labels:
    run: time-check
spec:
  volumes:
  - name: log-volume
    emptyDir: {}

  containers:
  - name: time-check
    image: busybox

    env:
    - name: TIME_FREQ
      valueFrom:
        configMapKeyRef:
          name: time-config
          key: TIME_FREQ

    volumeMounts:
    - name: log-volume
      mountPath: /opt/time

    command:
    - "/bin/sh"
    - "-c"
    - "while true; do date; sleep $TIME_FREQ; done > /opt/time/time-check.log"
```

---

# Q4 — Deployment Rolling Update

## Task

Create a **Deployment** called:

```
nginx-deploy
```

Specifications:

* Image: `nginx:1.16`
* Replicas: `4`
* Strategy: `RollingUpdate`
* `maxSurge: 1`
* `maxUnavailable: 2`

Then:

1. Upgrade to **nginx:1.17**
2. Roll back to the previous version.

---

## Solution

### Generate manifest

```bash
kubectl create deployment nginx-deploy \
--image=nginx:1.16 \
--replicas=4 \
--dry-run=client -o yaml > nginx-deploy.yaml
```

---

### Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx-deploy
spec:
  replicas: 4

  selector:
    matchLabels:
      app: nginx-deploy

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2

  template:
    metadata:
      labels:
        app: nginx-deploy

    spec:
      containers:
      - name: nginx
        image: nginx:1.16
```

---

### Apply

```bash
kubectl apply -f nginx-deploy.yaml
```

---

### Upgrade

```bash
kubectl set image deployment nginx-deploy nginx=nginx:1.17
```

---

### Rollback

```bash
kubectl rollout undo deployment nginx-deploy
```

---

# Q5 — Redis Deployment

## Task

Create a Deployment named:

```
redis
```

Specifications:

| Field          | Value        |
| -------------- | ------------ |
| Image          | redis:alpine |
| Replicas       | 1            |
| Label          | app=redis    |
| Container Port | 6379         |
| CPU Request    | 200m         |

Volumes:

| Volume       | Type      | Mount              |
| ------------ | --------- | ------------------ |
| data         | emptyDir  | /redis-master-data |
| redis-config | ConfigMap | /redis-master      |

The ConfigMap **already exists**.

---

## Solution

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  labels:
    app: redis

spec:
  replicas: 1

  selector:
    matchLabels:
      app: redis

  template:
    metadata:
      labels:
        app: redis

    spec:
      volumes:
      - name: data
        emptyDir: {}

      - name: redis-config
        configMap:
          name: redis-config

      containers:
      - name: redis
        image: redis:alpine

        ports:
        - containerPort: 6379

        resources:
          requests:
            cpu: "200m"

        volumeMounts:
        - name: data
          mountPath: /redis-master-data

        - name: redis-config
          mountPath: /redis-master
```
