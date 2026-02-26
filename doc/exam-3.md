
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

---

### Namespace Isolation
Create namespace:

prod-security

---

### Node Scheduling Advanced
Deploy pod compute-heavy using image httpd.

Scheduling rules:

- Node label required:

performance=high


- Preferred zone:

zone=us-east-1a


Use affinity rules (not nodeSelector).

---

### Deployment Rolling Update
Create Deployment api-gateway.

Specifications:
- Image: nginx:1.22
- Replicas: 4

Then update image to nginx:1.23 using rolling update.

Validation:
- Minimum 3 replicas available during update.

---

### Service Selector Logic
Create Deployment backend-app with labels:


app=backend
tier=api


Expose Deployment using ClusterIP Service.

Service name: backend-service
Port mapping: 8080 → 80

---

### Network Policy Security
Inside namespace prod-security create NetworkPolicy internal-only.

Rules:

- Allow ingress traffic only from pods with label:

role=trusted-client


- Block all other traffic.

Hint:
Use podSelector and ingress rules.

---

### Persistent Volume
Create PersistentVolume pv-log-storage.

Specifications:
- Storage: 300Mi
- AccessMode: ReadWriteOnce
- Path: /data/logs

Create PVC pvc-log-storage.

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

---

### Helm Package Task
Use Helm to deploy nginx chart.

Requirements:

- Release name: web-release
- Chart: stable/nginx
- Namespace: helm-test

Verify deployment.

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

---

### Secret Management
Create secret db-app-secret with data:

- DB_USER=admin
- DB_PASS=pass123

Mount secret inside pod secure-db-app at path:

/etc/db/secret


---

### Debugging Scenario
Deployment faulty-api has pods restarting continuously.

Task:
- Identify cause
- Fix deployment
- Ensure 2 ready replicas.

---
