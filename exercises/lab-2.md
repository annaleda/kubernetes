# CKAD Lightning Lab — Part 2

---

# Q1 — Troubleshoot Pod + Liveness Probe

## Task

Several pods are deployed across multiple namespaces.

1. Identify the **pod that is not in Ready state**.
2. Troubleshoot and fix the issue.
3. Add a **check that restarts the container** if the command below fails:

```bash
ls /var/www/html/file_check
```

Probe configuration:

| Field         | Value      |
| ------------- | ---------- |
| Initial Delay | 10 seconds |
| Period        | 60 seconds |

You may **delete and recreate the pod** if necessary.

Ignore warnings from the probe.

---

## Solution

The pod **`nginx1401`** in namespace **`dev1401`** is not Ready due to a failing **readinessProbe**.

### Fixed Pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx1401
  namespace: dev1401
  labels:
    run: nginx

spec:
  containers:
  - name: nginx
    image: kodekloud/nginx
    imagePullPolicy: IfNotPresent

    ports:
    - containerPort: 9080
      protocol: TCP

    readinessProbe:
      httpGet:
        path: /
        port: 9080

    livenessProbe:
      exec:
        command:
        - ls
        - /var/www/html/file_check
      initialDelaySeconds: 10
      periodSeconds: 60
```

---

# Q2 — CronJob

## Task

Create a **CronJob** called:

```bash
dice
```

Specifications:

| Field        | Value                |
| ------------ | -------------------- |
| Schedule     | Every 1 minute       |
| Image        | kodekloud/throw-dice |
| Parallelism  | Non-parallel         |
| Completions  | 1                    |
| backoffLimit | 25                   |
| Timeout      | 20 seconds           |

The pod template is located at:

```bash
/root/throw-a-dice
```

The container returns a number between **1 and 6**.

* **6 → success**
* **others → failure**

You do **not need to wait for completion**, only create the CronJob correctly.

---

## Solution

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: dice

spec:
  schedule: "*/1 * * * *"

  jobTemplate:
    spec:

      completions: 1
      backoffLimit: 25
      activeDeadlineSeconds: 20

      template:
        spec:
          containers:
          - name: dice
            image: kodekloud/throw-dice

          restartPolicy: Never
```

---

# Q3 — Pod with Secret Volume

## Task

Create a pod with the following specifications:

| Field          | Value      |
| -------------- | ---------- |
| Pod Name       | my-busybox |
| Namespace      | dev2406    |
| Image          | busybox    |
| Container Name | secret     |

The container should:

```bash
sleep 3600
```

Mount a **read-only secret volume**:

| Volume Name   | Secret         | Mount Path         |
| ------------- | -------------- | ------------------ |
| secret-volume | dotfile-secret | /etc/secret-volume |

Schedule the pod **only on the node `controlplane`**.

---

## Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-busybox
  namespace: dev2406
  labels:
    run: my-busybox

spec:

  nodeSelector:
    kubernetes.io/hostname: controlplane

  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret

  containers:
  - name: secret
    image: busybox

    command:
    - sleep
    args:
    - "3600"

    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secret-volume
      readOnly: true
```

---

# Q4 — Ingress Virtual Host Routing

## Task

Create a single ingress resource:

```bash
ingress-vh-routing
```

Routing rules:

| Host                    | Path   | Service          |
| ----------------------- | ------ | ---------------- |
| watch.ecom-store.com    | /video | video-service    |
| apparels.ecom-store.com | /wear  | apparels-service |

Ingress Controller Port:

```bash
30093
```

Add the annotation:

```yaml
nginx.ingress.kubernetes.io/rewrite-target: /
```

---

## Solution

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: ingress-vh-routing

  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /

spec:

  rules:

  - host: watch.ecom-store.com
    http:
      paths:
      - path: /video
        pathType: Prefix
        backend:
          service:
            name: video-service
            port:
              number: 8080

  - host: apparels.ecom-store.com
    http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
            name: apparels-service
            port:
              number: 8080
```

---

# Q5 — Logs Filtering

## Task

A pod called:

```bash
dev-pod-dind-878516
```

is deployed in the **default namespace**.

Inspect logs for the container:

```bash
log-x
```

Extract **only WARNING messages** and redirect them to:

```bash
/opt/dind-878516_logs.txt
```

on the **controlplane node**.

---

## Solution

```bash
kubectl logs dev-pod-dind-878516 -c log-x | grep WARNING > /opt/dind-878516_logs.txt
```
