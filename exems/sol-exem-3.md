
## Mock Exam-3
---

### Deploy Multi-Container Pod Pattern
Create a pod named metrics-app.

Requirements:

Container 1:
- Name: app-server
- Image: nginx
- Port: 80

Container 2:
- Name: log-agent
- Image: busybox
- Command:

sh -c "while true; do echo monitoring >> /var/log/metrics.log; sleep 10; done"


Validation:
- Both containers must be running.

```
k run metrics-app --image=nginx --port=80 --dry-run=client -o yaml > pod1.yaml

vi pod1.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: metrics-app
  name: metrics-app
spec:
  containers:
  - image: nginx
    name: metrics-app
    ports:
    - containerPort: 80
    resources: {}
  - image: busybox
    name: log-agent
    command:
      - sh
      - -c
      - "while true; do echo monitoring >> /var/log/metrics.log; sleep 10; done"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

k apply -f pod1.yaml

```
---

### Namespace Isolation
Create namespace:

prod-security

```
k create ns prod-security

```
---

### Node Scheduling Advanced
Deploy pod compute-heavy using image httpd.

Scheduling rules:

- Node label required:

performance=high


- Preferred zone:

zone=us-east-1a


Use affinity rules (not nodeSelector).

```
k run compute-heavy --image=httpd  --dry-run=client -o yaml > pod2.yaml

vi pod2.yaml

apiVersion: v1
kind: Pod
metadata:
  name: compute-heavy
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: performance
            operator: In
            values:
            - high
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - us-east-1a
  containers:
  - name: compute-heavy
    image: httpd

k apply -f pod2.yaml
```
---

### Deployment Rolling Update
Create Deployment api-gateway.

Specifications:
- Image: nginx:1.22
- Replicas: 4

Then update image to nginx:1.23 using rolling update.

Validation:
- Minimum 3 replicas available during update.

```
kubectl create deployment api-gateway --image=nginx:1.22 --replicas=4

kubectl edit deployment api-gateway

strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1

kubectl set image deploy api-gateway nginx=nginx:1.23

kubectl rollout status deployment api-gateway
```
---

### Service Selector Logic
Create Deployment backend-app with labels:


app=backend
tier=api


Expose Deployment using ClusterIP Service.

Service name: backend-service
Port mapping: 8080 → 80

```
k create deploy backend-app --dry-run=client -o yaml > svc1.yaml

vi svc1.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
      tier: api
  template:
    metadata:
      labels:
        app: backend
        tier: api
    spec:
      containers:
      - name: backend
        image: nginx
        ports:
        - containerPort: 80

k apply -f svc1.yaml

k expose deploy backend-service --name=backend-service --port=8080 --target-port=80 --dry-run=client -o yaml > svc2.yaml

cat svc2.yaml

apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
    tier: api
  ports:
  - port: 8080
    targetPort: 80

k apply -f svc2.yaml

```
---

### Network Policy Security
Inside namespace prod-security create NetworkPolicy internal-only.

Rules:

- Allow ingress traffic only from pods with label:

role=trusted-client


- Block all other traffic.

Hint:
Use podSelector and ingress rules.

```
vi np.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-only
  namespace: prod-security
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: trusted-client

k apply np.yaml

```
---

### Persistent Volume
Create PersistentVolume pv-log-storage.

Specifications:
- Storage: 300Mi
- AccessMode: ReadWriteOnce
- Path: /data/logs

Create PVC pvc-log-storage.

```
vi pv.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log-storage
spec:
  capacity:
    storage: 300Mi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /data/logs

k apply pv.yaml

vi pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-log-storage
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 300Mi

k apply pvc.yaml
```
---

### Health Monitoring
Update Deployment web-health.

Add probes:

Liveness Probe:
- HTTP GET /healthz
- Port 8080
- Initial delay: 10s

Readiness Probe:
- HTTP GET /ready
- Port 8080


```
k create deploy web-health --image=nginx --replicas=1 -o -yaml > deploy.yaml

vi deploy.yaml

livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080

k apply deploy.yaml
```
---

### Helm Package Task
Use Helm to deploy nginx chart.

Requirements:

- Release name: web-release
- Chart: stable/nginx
- Namespace: helm-test

Verify deployment.

```
helm repo add stable https://charts.helm.sh/stable
helm repo update

kubectl create namespace helm-test

helm install web-release stable/nginx -n helm-test


helm list -n helm-test
kubectl get all -n helm-test

```
---

### Kustomize Task
Create kustomization structure:

base/
overlays/dev/
kustomization.yaml

Base configuration:
- Image: nginx:1.21

Overlay overrides:
- Namespace → dev-overlay
- Image → nginx:1.23

```
vi base/kustom.yaml

resources:
- deployment.yaml

k create deploy nginx-app --image=nginx:1.21 --replicas=1 -o -yaml > base/deployment.yaml

vi base/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
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
        image: nginx:1.21

k apply -f base/deployment.yaml

vi overlays/dev/kustomization.yaml

namespace: dev-overlay

resources:
- ../../base

images:
- name: nginx
  newTag: "1.23"

k apply -k overlays/dev
```
---

### Secret Management
Create secret db-app-secret with data:

- DB_USER=admin
- DB_PASS=pass123

Mount secret inside pod secure-db-app at path:

/etc/db/secret

```
kubectl create secret generic db-app-secret \
--from-literal=DB_USER=admin \
--from-literal=DB_PASS=pass123

k run secure-db-app --image=nginx --dry-run=client -o yaml > pod4.yaml

vi pod4.yaml

apiVersion: v1
kind: Pod
metadata:
  name: secure-db-app
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: db-secret
      mountPath: /etc/db/secret
      readOnly: true
  volumes:
  - name: db-secret
    secret:
      secretName: db-app-secret

k apply -f pod4.yaml

```

---

### Debugging Scenario
Deployment faulty-api has pods restarting continuously.

Task:
- Identify cause
- Fix deployment
- Ensure 2 ready replicas.

```

kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>

kubectl edit deployment faulty-api

kubectl scale deployment faulty-api --replicas=2

kubectl get deployment faulty-api

```
---
