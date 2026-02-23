# 📘 CKAD – Esercizi e Soluzioni con Alias / Shortcut

*(contiene esercizi CKAD principali e comandi shortcut.)*

---

## 🔹 1. Core Concepts

### 1.1 Pod Singolo
**Esercizio:** Crea un Pod `nginx-pod` con immagine nginx.

<details>
<summary>Soluzione</summary>

```bash
# Normale
kubectl run nginx-pod --image=nginx

# Shortcut
k run nginx-pod --image=nginx
```
</details>

### 1.2 Deployment e Scaling

**Esercizio:** Deployment web-deploy con 3 repliche, scala a 5.

<details> <summary>Soluzione</summary>
  
```bash
# Deployment YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx
```
```bash
#Scalare Deployment
kubectl scale deployment web-deploy --replicas=5

#Shortcut
kd scale web-deploy --replicas=5
```
</details>

### 1.3 Namespace

**Esercizio:** Creare namespace dev e deploy pod dentro.

<details> <summary>Soluzione</summary>

```bash
# Normale
kubectl create ns dev
kubectl run nginx-dev --image=nginx -n dev

# Shortcut
k create ns dev
k run nginx-dev --image=nginx -n dev
```

</details>

## 🔹 2. Configuration

### 2.1 ConfigMap

**Esercizio:** Creare ConfigMap app-config con variabile ENV=prod.

<details> <summary>Soluzione</summary>

```bash  
# Normale
kubectl create configmap app-config --from-literal=ENV=prod

# Shortcut
kdry "create configmap app-config --from-literal=ENV=prod"
```

</details>

### 2.2 Secret

**Esercizio:** Creare Secret db-secret con password=xyz.

<details> <summary>Soluzione</summary>

```bash
# Normale
kubectl create secret generic db-secret --from-literal=password=xyz

# Shortcut
kdry "create secret generic db-secret --from-literal=password=xyz"
```

</details>

### 2.3 Montaggio ConfigMap / Secret

**Esercizio:** Montare ConfigMap e Secret come volume in un Pod.

<details> <summary>Soluzione</summary>
  
```bash  
volumes:
- name: config-volume
  configMap:
    name: app-config
- name: secret-volume
  secret:
    secretName: db-secret

volumeMounts:
- name: config-volume
  mountPath: /etc/config
- name: secret-volume
  mountPath: /etc/secret
```

</details>

### 2.4 Resource Requests & Limits

**Esercizio:** Definire requests e limits su un Pod.

<details> <summary>Soluzione</summary>
  
```bash  
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```
</details>


## 🔹 3. Multi-Container Pods

### 3.1 Sidecar + InitContainer

**Esercizio:** Pod con nginx + busybox sidecar, initContainer crea file.

<details> <summary>Soluzione</summary>

```bash   
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
  
</details>

## 🔹 4. Services & Networking

### 4.1 ClusterIP & NodePort

**Esercizio:** Esporre deployment web-deploy.

<details> <summary>Soluzione</summary>

```bash   
# ClusterIP
kubectl expose deployment web-deploy --type=ClusterIP --port=80
ks expose deployment web-deploy --type=ClusterIP --port=80

# NodePort
kubectl expose deployment web-deploy --type=NodePort --port=80
ks expose deployment web-deploy --type=NodePort --port=80
```

</details>

### 4.2 Ingress

**Esercizio:** Creare regole HTTP.

<details> <summary>Soluzione</summary>

```bash   
kubectl apply -f ingress.yaml
kapply ingress.yaml
```

</details>

## 🔹 5. Storage

### 5.1 emptyDir

**Esercizio:** Pod con volume temporaneo.

<details> <summary>Soluzione</summary>

```bash   
volumes:
- name: temp-storage
  emptyDir: {}
  
```
  
</details>

### 5.2 PVC + Mount

**Esercizio:** PVC 1Gi montato in un Pod.

<details> <summary>Soluzione</summary>
  
```bash   
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

volumes:
- name: my-storage
  persistentVolumeClaim:
    claimName: my-pvc
volumeMounts:
- name: my-storage
  mountPath: /data

# Shortcut
kdry "create pvc my-pvc --storage=1Gi --access-modes=ReadWriteOnce" > pvc.yaml
kapply pvc.yaml

```
</details>

## 🔹 6. Observability

### 6.1 Liveness / Readiness / Startup

**Esercizio:** Aggiungi probe su nginx.

<details> <summary>Soluzione</summary>

```bash   
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5

startupProbe:
  httpGet:
    path: /
    port: 80
  failureThreshold: 30
  periodSeconds: 10
```

</details>

### 6.2 Logs / Events

**Esercizio:** Vedere logs e eventi.

<details> <summary>Soluzione</summary>

```bash   
kubectl logs nginx-pod
kubectl get events

# Shortcut
kp logs nginx-pod
k get events
```

</details>

## 🔹 7. Pod Design

### 7.1 Job

**Esercizio:** Job stampa "Hello CKAD".

<details> <summary>Soluzione</summary>

```bash   
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

# Shortcut
kdry "create job hello-job --image=busybox -- date"
```

</details>

### 7.2 CronJob

**Esercizio:** CronJob ogni minuto.

<details> <summary>Soluzione</summary>

```bash   
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

# Shortcut
kdry "create cronjob hello-cron --image=busybox --schedule='*/1 * * * *'"
```
</details>

## 🔹 8. Security
### 8.1 ServiceAccount

**Esercizio:** Creare SA app-sa.

<details> <summary>Soluzione</summary>

```bash   
kubectl create serviceaccount app-sa
kdry "create sa app-sa"
```
</details>

### 8.2 Role / RoleBinding

**Esercizio:** Role pod-reader e binding a app-sa.

<details> <summary>Soluzione</summary>

```bash   
kdry "create role pod-reader --verb=get,list,watch --resource=pods"
kdry "create rolebinding read-pods --role=pod-reader --serviceaccount=default:app-sa"
```

</details>

### 8.3 SecurityContext

**Esercizio:** Pod non-root.

<details> <summary>Soluzione</summary>

```bash  
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```
</details>
