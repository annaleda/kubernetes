# CKAD Soluzioni Esercizi

*(Questa pagina contiene tutte le soluzioni agli esercizi sopra elencati, esempi YAML e comandi pronti da eseguire.)*

---

## Core Concepts

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
# Scalare il deployment
kubectl scale deployment nginx-deploy --replicas=5

# Creare namespace e pod
kubectl create ns dev
kubectl run nginx-dev --image=nginx -n dev
```

---

## Configuration

### ConfigMap

```bash
kubectl create configmap my-config --from-literal=key=value
kubectl create configmap my-config-file --from-env-file=app.env
```

### Secret

```bash
kubectl create secret generic my-secret --from-literal=password=abc123
kubectl create secret generic my-secret-file --from-env-file=secret.env
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

## Multi-Container Pods

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

## Services & Networking

```bash
# ClusterIP interno
kubectl expose deployment nginx --type=ClusterIP --port=80

# NodePort esposto su nodo
kubectl expose deployment nginx --type=NodePort --port=80

# Esempio Ingress (se controller installato)
kubectl apply -f ingress.yaml
```

---

## Storage

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

## Observability

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
kubectl logs <pod>
kubectl logs -f <pod>
kubectl logs <pod> -c <container>
kubectl get events
kubectl top pod
kubectl top node
```

---

## Pod Design

### Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  backoffLimit: 3
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: hello
        image: busybox
        command: ["echo", "Hello CKAD"]
```

### CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: hello
            image: busybox
            command: ["echo", "Hello CKAD"]
```

### Deployment e Rollout

```bash
kubectl rollout status deployment nginx-deploy
kubectl rollout undo deployment nginx-deploy
kubectl scale deployment nginx-deploy --replicas=5
```

---

## Security

### ServiceAccount

```bash
kubectl create serviceaccount my-sa
```

```yaml
spec:
  serviceAccountName: my-sa
```

### Role / RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: my-sa
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
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
