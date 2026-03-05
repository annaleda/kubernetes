# Mock Lab-3

## Multi-Container Pod Logging

Create pod **logging-stack**.

### Requirements

Container 1:

- Name: `web-server`
- Image: `nginx`
- Port: `80`

Container 2:

- Name: `log-writer`
- Image: `busybox`

Command:

```bash
sh -c "while true; do echo log entry >> /var/log/app.log; sleep 10; done"
```

Both containers must share volume mounted at:

```
/var/log
```

---

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logging-stack
spec:
  volumes:
  - name: shared-logs
    emptyDir: {}

  containers:
  - name: web-server
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log

  - name: log-writer
    image: busybox
    command:
    - sh
    - -c
    - while true; do echo log entry >> /var/log/app.log; sleep 10; done
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
```

---

## Namespace and RBAC

Create namespace:

```
dev-team
```

Create ServiceAccount:

```
dev-reader
```

Grant **read-only access to pods** in namespace `dev-team`.

Use:

- Role
- RoleBinding

---

### Solution

```bash
kubectl create ns dev-team
kubectl create sa dev-reader -n dev-team
```

Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev-team
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list","watch"]
```

RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: dev-team
subjects:
- kind: ServiceAccount
  name: dev-reader
  namespace: dev-team
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## Deployment with Node Affinity

Create Deployment:

```
zone-app
```

Specifications:

Image:

```
nginx:1.23
```

Replicas:

```
3
```

Scheduling rules:

Prefer nodes with label:

```
zone=eu-west
```

Use **nodeAffinity**.

---

### Solution

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: zone-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: zone-app
  template:
    metadata:
      labels:
        app: zone-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.23

      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: zone
                operator: In
                values:
                - eu-west
```

---

## NetworkPolicy Security

Inside namespace `dev-team` create NetworkPolicy:

```
internal-only
```

Requirements:

Allow ingress only from pods with label:

```
access=internal
```

Block all other traffic.

---

### Solution

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-only
  namespace: dev-team
spec:
  podSelector: {}
  policyTypes:
  - Ingress

  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: internal
```

---

# Mock Lab-4

## Persistent Storage Scenario

Create PersistentVolume.

Specifications:

Name:

```
pv-data
```

Size:

```
1Gi
```

AccessMode:

```
ReadWriteOnce
```

Path:

```
/mnt/data
```

Then create:

```
PersistentVolumeClaim pvc-data
```

Finally create pod:

```
data-app
```

Mount PVC at:

```
/data
```

---

### Solution

PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /mnt/data
```

PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-app
spec:
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: pvc-data

  containers:
  - name: busybox
    image: busybox
    command: ["sleep","3600"]

    volumeMounts:
    - name: data-volume
      mountPath: /data
```

---

## ConfigMap Environment Injection

Create ConfigMap:

```
app-config
```

Data:

```
MODE=production
LOG_LEVEL=debug
CACHE=true
```

Create pod:

```
config-test
```

Use **busybox** image and inject variables from ConfigMap.

---

### Solution

```bash
kubectl create configmap app-config \
--from-literal=MODE=production \
--from-literal=LOG_LEVEL=debug \
--from-literal=CACHE=true
```

Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-test
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep","3600"]
    envFrom:
    - configMapRef:
        name: app-config
```

---

## CronJob Task

Create CronJob:

```
system-check
```

Schedule:

```
every 2 minutes
```

Container:

```
busybox
```

Command:

```
echo system healthy
```

RestartPolicy:

```
Never
```

---

### Solution

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: system-check
spec:
  schedule: "*/2 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: check
            image: busybox
            command:
            - sh
            - -c
            - echo system healthy
          restartPolicy: Never
```

---

## Ingress Routing

Create Ingress:

```
web-routing
```

Host:

```
training.local
```

Routes:

```
/api -> api-service:80
/web -> web-service:80
```

---

### Solution

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-routing
spec:
  rules:
  - host: training.local
    http:
      paths:

      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80

      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```
