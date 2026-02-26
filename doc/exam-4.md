## Mock Exam-4 
---

### Multi-Container Logging Architecture
Create pod observability-stack.

Requirements:

Container 1:
- Name: api-service
- Image: nginx
- Port: 80

Container 2:
- Name: metrics-collector
- Image: busybox
- Command:

sh -c "while true; do echo metrics >> /var/log/metrics.txt; sleep 5; done"


Ensure containers share volume at:

/var/log


---

### Namespace and RBAC

Create namespace:

secure-apps


Create ServiceAccount:

app-reader-sa


Grant read-only access to pods in namespace secure-apps.

Use Role and RoleBinding.

---

### Advanced Scheduling (Affinity + AntiAffinity)

Create Deployment high-availability-app.

Requirements:

- Image: nginx:1.23
- Replicas: 3

Rules:

- Prefer nodes with label:

zone=eu-central-1a


- Avoid scheduling pods on nodes where same app is running.

Use podAffinity and podAntiAffinity.

---

### Network Security Hard Task

Inside namespace secure-apps create NetworkPolicy deny-external.

Behavior:

Allow ingress only:

- From namespace secure-apps
- From pods with label:

access=internal


Block:
- External traffic

---

### Storage Multi-Step Task

Create:

PersistentVolume:
- Name: pv-backup-data
- Size: 500Mi
- AccessMode: ReadWriteOnce
- Path: /mnt/backup

Then:

Create PersistentVolumeClaim pvc-backup-data.

Finally:

Create pod backup-worker that mounts PVC at:

/backup


---

### ConfigMap Advanced Injection

Create ConfigMap app-settings.

Data:

APP_MODE=production
CACHE_ENABLED=true
LOG_LEVEL=info


Deploy pod config-app using busybox image.

Inject ConfigMap as environment variables.

---

### Deployment Troubleshooting Scenario (Very Real Exam Trap)

Deployment broken-service exists.

Symptoms:
- Pods are created but stay in Pending state.

Task:

1. Identify root cause.
2. Fix scheduling or resource constraint.
3. Ensure:
   - 2 replicas running.

Check:
- Node selector
- Resource requests
- Taints and tolerations

---

### Helm Advanced Task

Install Helm chart:

Release name:

secure-nginx-release


Chart:

bitnami/nginx


Namespace:

helm-secure


Verify deployment.

---

### Kustomize Multi-Overlay Architecture

Create structure:


base/
dev/
prod/


Base:
- nginx:1.21

Dev overlay:
- namespace dev-env
- image nginx:1.22

Prod overlay:
- namespace prod-env
- image nginx:1.23

---

### Health Probes Advanced

Update deployment monitoring-app.

Add:

Liveness probe:
- HTTP GET /status
- Port 8080
- Initial delay: 15s

Readiness probe:
- HTTP GET /ready
- Port 8080

---

### Ingress Multi-Path Routing

Create Ingress app-router.

Rules:

Host:

app.training.local


Paths:

/api → service api-service port 80
/web → service web-service port 80

---

### Debugging Multi-Container Pod

Pod multi-debug is crashing.

Tasks:

- Find failing container.
- Fix startup command.
- Ensure pod is running.

---
