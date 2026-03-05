
## Mock Exam-4 

---

# Q1 — Multi-Container Logging Architecture

## Task

Create a Pod:

```
observability-stack
```

### Container 1

| Field | Value       |
| ----- | ----------- |
| Name  | api-service |
| Image | nginx       |
| Port  | 80          |

### Container 2

| Field | Value             |
| ----- | ----------------- |
| Name  | metrics-collector |
| Image | busybox           |

Command:

```
sh -c "while true; do echo metrics >> /var/log/metrics.txt; sleep 5; done"
```

Both containers must share the same volume mounted at:

```
/var/log
```

---

## Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: observability-stack

spec:
  volumes:
  - name: logs
    emptyDir: {}

  containers:

  - name: api-service
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: logs
      mountPath: /var/log

  - name: metrics-collector
    image: busybox
    command:
    - sh
    - -c
    - while true; do echo metrics >> /var/log/metrics.txt; sleep 5; done
    volumeMounts:
    - name: logs
      mountPath: /var/log
```

---

# Q2 — Namespace + RBAC

## Task

Create namespace:

```
secure-apps
```

Create ServiceAccount:

```
app-reader-sa
```

Grant **read-only access to Pods** inside namespace `secure-apps`.

Use:

* Role
* RoleBinding

---

## Solution

### Create Namespace

```bash
kubectl create ns secure-apps
```

### Create ServiceAccount

```bash
kubectl create sa app-reader-sa -n secure-apps
```

### Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: secure-apps

rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list","watch"]
```

### RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: secure-apps

subjects:
- kind: ServiceAccount
  name: app-reader-sa
  namespace: secure-apps

roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

# Q3 — Advanced Scheduling (Affinity)

## Task

Create Deployment:

```
high-availability-app
```

Specifications:

| Field    | Value      |
| -------- | ---------- |
| Image    | nginx:1.23 |
| Replicas | 3          |

Scheduling Rules:

Prefer nodes with label:

```
zone=eu-central-1a
```

Avoid scheduling pods on nodes where the **same app is already running**.

Use:

* podAffinity
* podAntiAffinity

---

## Solution

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: high-availability-app

spec:
  replicas: 3

  selector:
    matchLabels:
      app: ha-app

  template:
    metadata:
      labels:
        app: ha-app

    spec:

      affinity:

        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: zone
                operator: In
                values:
                - eu-central-1a

        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: ha-app
              topologyKey: kubernetes.io/hostname

      containers:
      - name: nginx
        image: nginx:1.23
```

---

# Q4 — Network Security

## Task

Inside namespace:

```
secure-apps
```

Create NetworkPolicy:

```
deny-external
```

Allow ingress only from:

* namespace `secure-apps`
* pods with label

```
access=internal
```

Block all external traffic.

---

## Solution

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: deny-external
  namespace: secure-apps

spec:
  podSelector: {}

  policyTypes:
  - Ingress

  ingress:
  - from:

    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: secure-apps

    - podSelector:
        matchLabels:
          access: internal
```

---

# Q5 — Storage Multi-Step

## Task

Create PersistentVolume:

| Field      | Value          |
| ---------- | -------------- |
| Name       | pv-backup-data |
| Size       | 500Mi          |
| AccessMode | ReadWriteOnce  |
| Path       | /mnt/backup    |

Create PVC:

```
pvc-backup-data
```

Create Pod:

```
backup-worker
```

Mount volume at:

```
/backup
```

---

## Solution

### PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: pv-backup-data

spec:
  capacity:
    storage: 500Mi

  accessModes:
  - ReadWriteOnce

  hostPath:
    path: /mnt/backup
```

---

### PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: pvc-backup-data

spec:
  accessModes:
  - ReadWriteOnce

  resources:
    requests:
      storage: 200Mi
```

---

### Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: backup-worker

spec:
  containers:
  - name: worker
    image: busybox
    command:
    - sleep
    - "3600"

    volumeMounts:
    - name: backup
      mountPath: /backup

  volumes:
  - name: backup
    persistentVolumeClaim:
      claimName: pvc-backup-data
```

---

# Q6 — ConfigMap Environment Injection

## Task

Create ConfigMap:

```
app-settings
```

Data:

```
APP_MODE=production
CACHE_ENABLED=true
LOG_LEVEL=info
```

Create Pod:

```
config-app
```

Use image:

```
busybox
```

Inject ConfigMap as **environment variables**.

---

## Solution

### ConfigMap

```bash
kubectl create configmap app-settings \
--from-literal=APP_MODE=production \
--from-literal=CACHE_ENABLED=true \
--from-literal=LOG_LEVEL=info
```

### Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: config-app

spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep","3600"]

    envFrom:
    - configMapRef:
        name: app-settings
```

---

# Q7 — Deployment Troubleshooting

## Task

Deployment:

```
broken-service
```

Problem:

Pods remain in:

```
Pending
```

Fix scheduling issue and ensure:

```
2 replicas running
```

Check:

* nodeSelector
* resource requests
* taints/tolerations

---

## Possible Fix Example

```bash
kubectl edit deployment broken-service
```

Remove incorrect nodeSelector or adjust resources.

Example corrected section:

```yaml
spec:
  replicas: 2
```

---

# Q8 — Helm Advanced

## Task

Install Helm chart:

| Field     | Value                |
| --------- | -------------------- |
| Release   | secure-nginx-release |
| Chart     | bitnami/nginx        |
| Namespace | helm-secure          |

---

## Solution

```bash
kubectl create ns helm-secure
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm install secure-nginx-release bitnami/nginx -n helm-secure
```

Verify:

```bash
helm list -n helm-secure
```

---

# Q9 — Kustomize Multi Overlay

## Task

Structure:

```
base/
dev/
prod/
```

Base image:

```
nginx:1.21
```

Dev overlay:

```
namespace: dev-env
image: nginx:1.22
```

Prod overlay:

```
namespace: prod-env
image: nginx:1.23
```

---

## Solution

### Base kustomization

```yaml
resources:
- deployment.yaml
```

---

### Dev overlay

```yaml
namespace: dev-env

images:
- name: nginx
  newTag: "1.22"
```

---

### Prod overlay

```yaml
namespace: prod-env

images:
- name: nginx
  newTag: "1.23"
```

---

# Q10 — Health Probes

## Task

Update Deployment:

```
monitoring-app
```

Add:

### Liveness Probe

```
HTTP GET /status
port 8080
initialDelaySeconds 15
```

### Readiness Probe

```
HTTP GET /ready
port 8080
```

---

## Solution

```yaml
livenessProbe:
  httpGet:
    path: /status
    port: 8080
  initialDelaySeconds: 15

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

---

# Q11 — Ingress Multi-Path

## Task

Create Ingress:

```
app-router
```

Host:

```
app.training.local
```

Paths:

| Path | Service     |
| ---- | ----------- |
| /api | api-service |
| /web | web-service |

---

## Solution

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: app-router

spec:
  rules:
  - host: app.training.local

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

---

# Q12 — Debugging Multi-Container Pod

## Task

Pod:

```
multi-debug
```

Problem:

Pod is **crashing**.

Tasks:

1. Identify failing container
2. Fix startup command
3. Ensure pod runs correctly

---

## Debug Steps

Check pod:

```bash
kubectl describe pod multi-debug
```

Check logs:

```bash
kubectl logs multi-debug -c <container-name>
```

Edit pod:

```bash
kubectl edit pod multi-debug
```

Fix incorrect command or arguments.

---
