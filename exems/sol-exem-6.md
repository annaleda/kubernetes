# CKAD Practice Exam — Solutions

---

## Q1 — Multi Container Pod

### Task

Create pod **app-observability**.

Containers:

### Container 1
- Name: api-service
- Image: nginx
- Port: 80

### Container 2
- Name: metrics-agent
- Image: busybox

Command:

```
while true; do echo metrics >> /var/log/metrics.log; sleep 5; done
```

Volume mount:

```
/var/log
```

---

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-observability

spec:
  volumes:
  - name: shared-log
    emptyDir: {}

  containers:
  - name: api-service
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-log
      mountPath: /var/log

  - name: metrics-agent
    image: busybox
    command:
    - sh
    - -c
    - while true; do echo metrics >> /var/log/metrics.log; sleep 5; done
    volumeMounts:
    - name: shared-log
      mountPath: /var/log
```

---

## Q2 — RBAC Security

Namespace:
```
secure-dev
```

ServiceAccount:
```
app-user
```

Permissions:
- get pods
- list pods
- watch pods

---

### Solution

```bash
kubectl create ns secure-dev
kubectl create sa app-user -n secure-dev
```

Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: secure-dev
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
  namespace: secure-dev
subjects:
- kind: ServiceAccount
  name: app-user
  namespace: secure-dev
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## Q3 — Scheduling Constraints

Deployment:
```
high-availability-app
```

Image:
```
nginx:1.23
```

Replicas:
```
3
```

Rules:

- Prefer nodes with label:
  ```
  zone=eu-central
  ```

- Avoid colocating pods of same deployment.

---

### Solution

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: high-availability-app

spec:
  replicas: 3
  selector:
    matchLabels:
      app: high-availability-app

  template:
    metadata:
      labels:
        app: high-availability-app

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
                - eu-central

        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: high-availability-app
              topologyKey: kubernetes.io/hostname
```

---

## Q4 — Network Policy

Namespace:
```
secure-dev
```

NetworkPolicy:
```
deny-external
```

Allow ingress only from:

- Pods with label:
```
access=internal
```

---

### Solution

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-external
  namespace: secure-dev

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

## Q5 — Storage

PersistentVolume:

Name:
```
pv-storage
```

Capacity:
```
500Mi
```

AccessMode:
```
ReadWriteOnce
```

Path:
```
/mnt/storage
```

---

### Solution

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage

spec:
  capacity:
    storage: 500Mi

  accessModes:
  - ReadWriteOnce

  hostPath:
    path: /mnt/storage
```

PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-storage

spec:
  accessModes:
  - ReadWriteOnce

  resources:
    requests:
      storage: 500Mi
```

Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-app

spec:
  volumes:
  - name: storage-volume
    persistentVolumeClaim:
      claimName: pvc-storage

  containers:
  - name: busybox
    image: busybox
    command: ["sleep","3600"]

    volumeMounts:
    - name: storage-volume
      mountPath: /data
```

---

## Q6 — ConfigMap Injection

```bash
kubectl create configmap app-config \
--from-literal=ENV=production \
--from-literal=DEBUG=false \
--from-literal=MAX_CONNECTIONS=100
```

Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-loader

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

## Q7 — Probe Configuration

Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: monitor-app

spec:
  replicas: 1
  selector:
    matchLabels:
      app: monitor-app

  template:
    metadata:
      labels:
        app: monitor-app

    spec:
      containers:
      - name: nginx
        image: nginx

        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15

        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
```

---

## Q8 — Ingress Routing

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

## Q9 — Troubleshooting Pod

Check:

```bash
kubectl describe pod broken-app
kubectl logs broken-app
```

Fix configuration error (command/env/image).

---

## Q10 — CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: system-audit

spec:
  schedule: "*/3 * * * *"

  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: audit
            image: busybox
            command:
            - sh
            - -c
            - echo audit log completed

          restartPolicy: Never
```
