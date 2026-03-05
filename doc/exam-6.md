# CKAD Practice Exam Simulation

## Instructions

- Do not create unnecessary resources.
- Use Kubernetes native commands.
- Unless specified, use default namespace.
- Complete all tasks.

---

## Q1 — Multi Container Pod

Create pod **app-observability**.

Requirements:

Container 1:
- Name: api-service
- Image: nginx
- Port: 80

Container 2:
- Name: metrics-agent
- Image: busybox
- Command:

```
while true; do echo metrics >> /var/log/metrics.log; sleep 5; done
```

Share volume between containers:

```
/var/log
```

---

## Q2 — RBAC Security

Create namespace:

```
secure-dev
```

Create ServiceAccount:

```
app-user
```

Grant read-only access to pods inside namespace using:

- Role
- RoleBinding

Permissions required:

- get
- list
- watch

---

## Q3 — Scheduling Constraints

Create Deployment:

Name: high-availability-app

Specifications:

- Image: nginx:1.23
- Replicas: 3

Scheduling rules:

- Prefer nodes with label:
  ```
  zone=eu-central
  ```

- Avoid scheduling multiple pods of same deployment on same node.

Use affinity rules.

---

## Q4 — Network Security

Inside namespace:

```
secure-dev
```

Create NetworkPolicy:

Name:
```
deny-external
```

Rules:

Allow ingress only from:

- Pods with label:
  ```
  access=internal
  ```

- Same namespace traffic

Block external access.

---

## Q5 — Persistent Storage

Create PersistentVolume:

Name:
```
pv-storage
```

Size:
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

Then:

- Create PersistentVolumeClaim `pvc-storage`
- Create pod `storage-app`
- Mount volume at `/data`

---

## Q6 — ConfigMap Injection

Create ConfigMap:

Name:
```
app-config
```

Data:

```
ENV=production
DEBUG=false
MAX_CONNECTIONS=100
```

Create pod `config-loader`.

Inject ConfigMap as environment variables.

---

## Q7 — Probe Configuration

Update deployment:

Name:
```
monitor-app
```

Add probes.

Liveness Probe:
- HTTP GET `/health`
- Port 8080
- Initial delay: 15s

Readiness Probe:
- HTTP GET `/ready`
- Port 8080

---

## Q8 — Ingress Routing

Create Ingress:

Name:
```
app-router
```

Host:
```
app.training.local
```

Routing:

```
/api → api-service (port 80)
/web → web-service (port 80)
```

---

## Q9 — Troubleshooting Pod

Pod:
```
broken-app
```

Problem:

- Pod is restarting or stuck.

Tasks:

- Identify cause.
- Fix configuration.
- Ensure pod runs successfully.

---

## Q10 — CronJob Automation

Create CronJob:

Name:
```
system-audit
```

Schedule:
```
Every 3 minutes
```

Image:
```
busybox
```

Command:
```
echo audit log completed
```

Restart policy:
```
Never
```

---

# End of Exam
