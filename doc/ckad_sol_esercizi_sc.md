# 📘 CKAD – Soluzioni Esercizi con Alias / Shortcut

*(Soluzioni agli esercizi CKAD con YAML e comandi semplificati usando alias e shortcut.)*

---

## 1️⃣ Core Concepts

### Pod singolo

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
```

### Deployment e scaling

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
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
        image: nginx
```

```bash
# Scalare il deployment usando alias
kd scale nginx-deploy --replicas=5

# Creare namespace e pod
k create ns dev
kp run nginx-dev --image=nginx -n dev
```

---

## 2️⃣ Configuration

### ConfigMap

```bash
kdry "create configmap my-config --from-literal=key=value"
kdry "create configmap my-config-file --from-env-file=app.env"
```

### Secret

```bash
kdry "create secret generic my-secret --from-literal=password=abc123"
kdry "create secret generic my-secret-file --from-env-file=secret.env"
```

### Montaggio ConfigMap / Secret come volume

```yaml
volumes:
- name: config-volume
  configMap:
    name: my-config
- name: secret-volume
  secret:
    secretName: my-secret
```

```yaml
volumeMounts:
- name: config-volume
  mountPath: /etc/config
- name: secret-volume
  mountPath: /etc/secret
```

### Resource Requests & Limits

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

---

## 3️⃣ Multi-Container Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-example
spec:
  initContainers:
  - name: init-myfile
    image: busybox
    command: ['sh', '-c', 'echo hello > /data/message']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'tail -f /dev/null']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  volumes:
  - name: shared-data
    emptyDir: {}
```

---

## 4️⃣ Services & Networking

```bash
# ClusterIP interno
ks expose deployment nginx --type=ClusterIP --port=80

# NodePort esposto su nodo
ks expose deployment nginx --type=NodePort --port=80

# Esempio Ingress (se controller installato)
kapply ingress.yaml
```

---

## 5️⃣ Storage

### emptyDir

```yaml
volumes:
- name: temp-storage
  emptyDir: {}
```

### PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### Montaggio PVC

```yaml
volumes:
- name: my-storage
  persistentVolumeClaim:
    claimName: my-pvc

volumeMounts:
- name: my-storage
  mountPath: /data
```

---

## 6️⃣ Observability

### Liveness Probe

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
```

### Readiness Probe

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Startup Probe

```yaml
startupProbe:
  httpGet:
    path: /
    port: 80
  failureThreshold: 30
  periodSeconds: 10
```

### Logs e metrics

```bash
kp logs <pod>
kp logs -f <pod>
kp logs -c <container>
k get events
k top pod
k top node
```

---

## 7️⃣ Pod Design

### Job

```bash
kdry "create job hello-job --image=busybox -- echo Hello CKAD"
```

### CronJob

```bash
kdry "create cronjob hello-cron --image=busybox --schedule='*/1 * * * *' -- echo Hello CKAD"
```

### Deployment e Rollout

```bash
kd rollout status nginx-deploy
kd rollout undo nginx-deploy
kd scale nginx-deploy --replicas=5
```

---

## 8️⃣ Security

### ServiceAccount

```bash
kdry "create sa my-sa"
```

```yaml
spec:
  serviceAccountName: my-sa
```

### Role / RoleBinding

```bash
kdry "create role pod-reader --verb=get,list,watch --resource=pods"
kdry "create rolebinding read-pods --role=pod-reader --serviceaccount=default:my-sa"
```

### SecurityContext (non-root)

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```

### NetworkPolicy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      role: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

---

> usa sempre gli alias e shortcut. Se serve YAML, genera con `kdry "<comando>"` e applica con `kapply <file>`.

