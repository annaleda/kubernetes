
# 📘 CKAD – Esercizi e Soluzioni

---

### 1 Pod Singolo
**Esercizio:** Create a Pod named redis-pod using image redis:7.

<details>
<summary>Soluzione</summary>

```bash
# Normale
kubectl run redis-pod --image=redis:7

# Shortcut
k run redis-pod --image=redis:7
```
</details>

<details>
  <summary>Teoria</summary>
</details>

---


### 2 Deployment con repliche
**Esercizio:** Create a Deployment api with image nginx:1.25 and 3 replicas.

<details>
<summary>Soluzione</summary>

```bash
kubectl create deployment api --image=nginx:1.25 --replicas=3

# Verifica
kubectl get deploy
kubectl get pods
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 3 Scaling
**Esercizio:** Scale deployment api to 5 replicas.

<details>
<summary>Soluzione</summary>

```bash
kubectl scale deployment api --replicas=5

# Verifica
kubectl get pods
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 4 Update Image
**Esercizio:** Update api to use image nginx:1.26.

<details>
<summary>Soluzione</summary>

```bash
kubectl set image deployment/api nginx=nginx:1.26

# Verifica
kubectl rollout status deployment api
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 5 Rollback
**Esercizio:** Rollback previous version of api.

<details>
<summary>Soluzione</summary>

```bash
kubectl rollout undo deployment api

# Verifica
kubectl rollout status deployment api
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 6 Service
**Esercizio:** Expose deployment api internally on port 80.

<details>
<summary>Soluzione</summary>

```bash
kubectl expose deployment api --port=80 --type=ClusterIP

# Attenzione! Il selector deve matchare le label del deployment
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 7 ConfigMap
**Esercizio:** Create ConfigMap app-config with MODE=prod.

<details>
<summary>Soluzione</summary>

```bash
kubectl create configmap app-config --from-literal=MODE=prod

```
</details>


<details>
  <summary>Teoria</summary>
</details>


---


### 8 Secret
**Esercizio:** Create Secret db-secret with password=mypass.

<details>
<summary>Soluzione</summary>

```bash
kubectl create secret generic db-secret --from-literal=password=mypass

```
</details>


<details>
  <summary>Teoria</summary>
</details>


---


### 9 Env da ConfigMap
**Esercizio:** Inject ConfigMap into Deployment as env variable.

<details>
<summary>Soluzione</summary>

```bash
kubectl get deployment api -o yaml > api.yaml
```

Nel container aggiungi:
```bash
env:
- name: MODE
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: MODE
```
Dopo
```bash
kubectl apply -f api.yaml
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 10 Liveness Probe
**Esercizio:** Add HTTP liveness probe on / port 80.

<details>
<summary>Soluzione</summary>

Nel container aggiungi:
```bash
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 11 Resource Limits
**Esercizio:** Add CPU/memory limits.

<details>
<summary>Soluzione</summary>

Nel container aggiungi:
```bash
resources:
  requests:
    cpu: 250m
    memory: 64Mi
  limits:
    cpu: 500m
    memory: 128Mi
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 12 Init Container
**Esercizio:** Add init container printing "init".

<details>
<summary>Soluzione</summary>

Nel container aggiungi:
```bash
initContainers:
- name: init
  image: busybox
  command: ['sh','-c','echo init']
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 13 Multi-container Pod
**Esercizio:** Create Pod with nginx + busybox sidecar.

<details>
<summary>Soluzione</summary>

Nel container aggiungi:
```bash
containers:
- name: nginx
  image: nginx
- name: sidecar
  image: busybox
  command: ['sh','-c','sleep 3600']
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 14 PVC
**Esercizio:** Create PVC 1Gi ReadWriteOnce.

<details>
<summary>Soluzione</summary>

Nel container aggiungi:
```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Dopo:
```bash
kubectl apply -f pvc.yaml
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 15 Mount PVC
**Esercizio:** Mount mypvc in Pod at /data.

<details>
<summary>Soluzione</summary>

Nel container aggiungi:
```bash
volumes:
- name: storage
  persistentVolumeClaim:
    claimName: mypvc

containers:
- name: app
  volumeMounts:
  - mountPath: /data
    name: storage
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 16 Job
**Esercizio:** Create Job that prints "hello".

<details>
<summary>Soluzione</summary>

```bash
kubectl create job hello-job --image=busybox -- echo hello
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 17 CronJob
**Esercizio:** Create CronJob every 5 minutes.
<details>
<summary>Soluzione</summary>

```bash
kubectl create cronjob hello-cron \
  --image=busybox \
  --schedule="*/5 * * * *" \
  -- echo hello
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 18 SecurityContext
**Esercizio:** Add SecurityContext.
<details>
<summary>Soluzione</summary>

```bash
securityContext:
  runAsUser: 1000
  allowPrivilegeEscalation: false
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---


### 19 Namespace
**Esercizio:** Create namespace dev and deploy nginx inside it.
<details>
<summary>Soluzione</summary>

```bash
kubectl create ns dev
kubectl create deployment nginx --image=nginx -n dev
```
</details>

<details>
  <summary>Teoria</summary>
</details>


---

### 20 NetworkPolicy (base)
**Esercizio:** Allow ingress only from pods with label app=frontend.
<details>
<summary>Soluzione</summary>

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```
</details>

<details>
  <summary>Teoria</summary>
</details>

