

## Mock Exam-5 
---

# Q1 — Multi-Container Monitoring Architecture

## Task

Create a Pod:

```
monitoring-stack
```

### Container 1

| Field | Value       |
| ----- | ----------- |
| Name  | app-backend |
| Image | nginx       |
| Port  | 80          |

### Container 2

| Field | Value        |
| ----- | ------------ |
| Name  | log-exporter |
| Image | busybox      |

Command:

```
sh -c "while true; do echo exporting logs >> /var/log/export.log; sleep 8; done"
```

Both containers must share a volume mounted at:

```
/var/log
```

---

## Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: monitoring-stack

spec:
  volumes:
  - name: logs
    emptyDir: {}

  containers:

  - name: app-backend
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: logs
      mountPath: /var/log

  - name: log-exporter
    image: busybox
    command:
    - sh
    - -c
    - while true; do echo exporting logs >> /var/log/export.log; sleep 8; done
    volumeMounts:
    - name: logs
      mountPath: /var/log
```

---

# Q2 — Namespace + ServiceAccount Security

## Task

Create namespace:

```
secure-monitoring
```

Create ServiceAccount:

```
monitor-sa
```

Grant permissions to:

* Read Pods
* Read Services

Use:

* Role
* RoleBinding

---

## Solution

Create namespace

```bash
kubectl create ns secure-monitoring
```

Create ServiceAccount

```bash
kubectl create sa monitor-sa -n secure-monitoring
```

### Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: monitor-reader
  namespace: secure-monitoring

rules:
- apiGroups: [""]
  resources: ["pods","services"]
  verbs: ["get","list","watch"]
```

### RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: monitor-binding
  namespace: secure-monitoring

subjects:
- kind: ServiceAccount
  name: monitor-sa
  namespace: secure-monitoring

roleRef:
  kind: Role
  name: monitor-reader
  apiGroup: rbac.authorization.k8s.io
```

---

# Q3 — Advanced Scheduling Strategy

## Task

Create Deployment:

```
resilient-app
```

Specifications

| Field    | Value      |
| -------- | ---------- |
| Image    | nginx:1.23 |
| Replicas | 4          |

Scheduling rules

Must run on nodes with label:

```
environment=production
```

Prefer nodes in zone:

```
zone=eu-west-1b
```

Avoid nodes where same app already runs.

Use:

* nodeAffinity
* podAntiAffinity

---

## Solution

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: resilient-app

spec:
  replicas: 4

  selector:
    matchLabels:
      app: resilient-app

  template:
    metadata:
      labels:
        app: resilient-app

    spec:

      affinity:

        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: environment
                operator: In
                values:
                - production

          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: zone
                operator: In
                values:
                - eu-west-1b

        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: resilient-app
              topologyKey: kubernetes.io/hostname

      containers:
      - name: nginx
        image: nginx:1.23
```

---

# Q4 — Network Policy Isolation

## Task

Inside namespace:

```
secure-monitoring
```

Create NetworkPolicy:

```
strict-isolation
```

Allow ingress only from:

Pods with label

```
team=trusted
```

And from the **same namespace**.

Default behavior:

Block all external traffic.

---

## Solution

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: strict-isolation
  namespace: secure-monitoring

spec:
  podSelector: {}

  policyTypes:
  - Ingress

  ingress:
  - from:

    - podSelector:
        matchLabels:
          team: trusted

    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: secure-monitoring
```

---

# Q5 — Persistent Storage Multi-Step

## Task

Create PersistentVolume

| Field      | Value          |
| ---------- | -------------- |
| Name       | pv-app-storage |
| Size       | 1Gi            |
| AccessMode | ReadWriteOnce  |
| Path       | /mnt/appdata   |

Create PVC

```
pvc-app-storage
```

Create Pod:

```
storage-worker
```

Mount volume at

```
/data/work
```

---

## Solution

### PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: pv-app-storage

spec:
  capacity:
    storage: 1Gi

  accessModes:
  - ReadWriteOnce

  hostPath:
    path: /mnt/appdata
```

---

### PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: pvc-app-storage

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
  name: storage-worker

spec:
  containers:
  - name: worker
    image: busybox
    command:
    - sleep
    - "3600"

    volumeMounts:
    - name: storage
      mountPath: /data/work

  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-app-storage
```

---

# Q6 — ConfigMap Environment Injection

## Task

Create ConfigMap:

```
system-config
```

Data

```
APP_ENV=production
DEBUG=false
MAX_CONNECTIONS=200
```

Create Pod:

```
config-worker
```

Image:

```
busybox
```

Inject ConfigMap as **environment variables**.

---

## Solution

Create ConfigMap

```bash
kubectl create configmap system-config \
--from-literal=APP_ENV=production \
--from-literal=DEBUG=false \
--from-literal=MAX_CONNECTIONS=200
```

Create Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: config-worker

spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep","3600"]

    envFrom:
    - configMapRef:
        name: system-config
```

---

# Q7 — Deployment CrashLoopBackOff Troubleshooting

## Task

Deployment:

```
api-failure
```

Pods show:

```
CrashLoopBackOff
```

Fix the issue and ensure:

```
2 replicas Ready
```

Check:

* container command
* environment variables
* resource limits
* image availability

---

## Debug Steps

Inspect pods

```
kubectl describe pod <pod-name>
```

Check logs

```
kubectl logs <pod> -c <container>
```

Edit deployment

```
kubectl edit deployment api-failure
```

Common fixes:

* incorrect command
* wrong env variables
* invalid image
* insufficient resources

---

# Q8 — Helm Production Deployment

## Task

Install Helm chart

| Field     | Value            |
| --------- | ---------------- |
| Release   | prod-web-release |
| Chart     | bitnami/nginx    |
| Namespace | production-apps  |

---

## Solution

```bash
kubectl create ns production-apps

helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm install prod-web-release bitnami/nginx -n production-apps
```

Verify

```
helm list -n production-apps
```

---

# Q9 — Kustomize Overlay Architecture

## Task

Structure

```
base/
staging/
production/
```

Base image

```
nginx:1.21
```

Staging overlay

```
namespace staging-env
image nginx:1.22
```

Production overlay

```
namespace prod-env
image nginx:1.23
```

---

## Solution

### Base kustomization

```yaml
resources:
- deployment.yaml
```

---

### Staging overlay

```yaml
namespace: staging-env

images:
- name: nginx
  newTag: "1.22"
```

---

### Production overlay

```yaml
namespace: prod-env

images:
- name: nginx
  newTag: "1.23"
```

---

# Q10 — Ingress Routing

## Task

Create Ingress:

```
production-router
```

Host

```
app.company.local
```

Paths

| Path      | Service          |
| --------- | ---------------- |
| /backend  | backend-service  |
| /frontend | frontend-service |

---

## Solution

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: production-router

spec:
  rules:
  - host: app.company.local

    http:
      paths:

      - path: /backend
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 80

      - path: /frontend
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

---

# Q11 — Security Context Hardening

## Task

Update pod:

```
secure-node-app
```

Requirements

* Run container as user **1000**
* Disable **privilege escalation**
* Add capability **SYS_TIME**

---

## Solution

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: secure-node-app

spec:
  containers:
  - name: app
    image: nginx

    securityContext:
      runAsUser: 1000
      allowPrivilegeEscalation: false
      capabilities:
        add:
        - SYS_TIME
```
---
