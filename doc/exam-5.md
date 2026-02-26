
## Mock Exam-5 
---

### Multi-Container Monitoring Architecture

Create pod monitoring-stack.

Requirements:

Container 1:
- Name: app-backend
- Image: nginx
- Port: 80

Container 2:
- Name: log-exporter
- Image: busybox
- Command:

sh -c "while true; do echo exporting logs >> /var/log/export.log; sleep 8; done"


Share volume between containers at:

/var/log


---

### Namespace + ServiceAccount Security Setup

Create namespace:

secure-monitoring


Create ServiceAccount:

monitor-sa


Grant permissions:

- Read pods
- Read services

Use:
- Role
- RoleBinding

---

### Advanced Scheduling Strategy

Deploy deployment resilient-app.

Specifications:

- Image: nginx:1.23
- Replicas: 4

Scheduling rules:

1. Must run on nodes with label:

environment=production


2. Prefer zone:

zone=eu-west-1b


3. Avoid nodes where same app already runs.

Use:
- node affinity
- pod anti-affinity

---

### Network Policy Isolation (Production Style)

Inside namespace secure-monitoring create NetworkPolicy strict-isolation.

Rules:

Allow ingress only:

- Pods with label:

team=trusted


- Same namespace traffic

Default behavior:
- Deny external traffic

---

### Persistent Storage Multi-Step Scenario

Create PersistentVolume:

Name:

pv-app-storage


Size:
- 1Gi

AccessMode:
- ReadWriteOnce

Path:

/mnt/appdata


Then create:

PersistentVolumeClaim pvc-app-storage.

Then deploy pod storage-worker mounting volume at:

/data/work


---

### ConfigMap + Environment Injection

Create ConfigMap system-config.

Data:

APP_ENV=production
DEBUG=false
MAX_CONNECTIONS=200


Deploy pod config-worker using busybox.

Inject ConfigMap as environment variables.

---

### Deployment Failure Troubleshooting (Hard Scenario)

Deployment api-failure exists.

Symptoms:
- Pods restart continuously
- Deployment shows CrashLoopBackOff

Task:

1. Identify failure cause.
2. Fix configuration.
3. Ensure:
   - 2 replicas ready.

Check:
- Container command
- Environment variables
- Resource limits
- Image availability

---

### Helm Production Deployment

Deploy chart using Helm:

Release:

prod-web-release


Chart:

bitnami/nginx


Namespace:

production-apps


Verify deployment health.

---

### Kustomize Overlay GitOps Structure

Create structure:


base/
staging/
production/


Base:
- nginx:1.21

Staging overlay:
- namespace staging-env
- image nginx:1.22

Production overlay:
- namespace prod-env
- image nginx:1.23

---

### Ingress Routing Production Scenario

Create Ingress production-router.

Host:

app.company.local


Paths:

/backend → backend-service port 80
/frontend → frontend-service port 80

---

### Security Context Hardening

Update pod secure-node-app.

Requirements:

- Run container as user 1000
- Disable privilege escalation
- Add capability SYS_TIME

---
