# 📘 CKAD – Esercizi e Soluzioni

---

### 81 Deployment con requirements multipli
**Esercizio:** Create a Deployment named payment-api in namespace prod.
The Deployment should:

use image nginx:1.25

have 3 replicas

expose container port 8080

define CPU limit of 300m

define readiness probe on / port 8080
<details>
<summary>Soluzione</summary>

```bash
kubectl create ns prod

kubectl create deployment payment-api \
  --image=nginx:1.25 \
  -n prod \
  --dry-run=client -o yaml > deploy.yaml
```

Modificare il file:

```bash
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: 300m
        readinessProbe:
          httpGet:
            path: /
            port: 8080
```

```bash
kubectl apply -f deploy.yaml
kubectl get deploy -n prod
```
</details>
<details>
  <summary>Teoria</summary>
Deployment completo tipico CKAD:

Namespace corretto

replicas sotto spec

containerPort nel container

resources sotto containers

readinessProbe deve usare la porta corretta

 Se readiness usa porta sbagliata → Pod Ready = False.
</details>

---

### 82 Fix Deployment esistente
**Esercizio:** Update Deployment frontend so that:

image is nginx:1.26

replicas are 4
<details>
<summary>Soluzione</summary>

```bash
kubectl set image deployment/frontend nginx=nginx:1.26
kubectl scale deployment frontend --replicas=4

kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
Non serve riscrivere YAML.

set image modifica container image

scale modifica replicas

 Il nome container deve combaciare con quello nel Deployment.
</details>

---

### 83 ConfigMap montato come volume
**Esercizio:** Create ConfigMap app-config with key config.txt=hello.
Mount inside Pod config-pod at /etc/config.

<details>
<summary>Soluzione</summary>

```bash
kubectl create configmap app-config --from-literal=config.txt=hello
```

Pod:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  volumes:
  - name: config-vol
    configMap:
      name: app-config
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
```
</details>
<details>
  <summary>Teoria</summary>
  Il file verrà creato come:

/etc/config/config.txt

 mountPath è directory, non file.
</details>

---

### 84 Secret come env
**Esercizio:** Create Secret db-secret with password=admin.
Inject into Deployment api as env variable.

<details>
<summary>Soluzione</summary>

```bash
kubectl create secret generic db-secret --from-literal=password=admin

```
```bash
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```
</details>
<details>
  <summary>Teoria</summary>
  key deve combaciare

Secret viene codificato base64

Può essere usato come env o volume
</details>

---

### 85 Job con 3 completions
**Esercizio:** Create Job batch-job using busybox that runs echo done.
Must complete 3 times.

<details>
<summary>Soluzione</summary>

```bash
kubectl create job batch-job --image=busybox --dry-run=client -o yaml > job.yaml
```
```bash
spec:
  completions: 3
  parallelism: 1
  template:
    spec:
      containers:
      - name: batch-job
        image: busybox
        command: ["sh","-c","echo done"]
      restartPolicy: Never
```

```bash
kubectl apply -f job.yaml
```
</details>
<details>
  <summary>Teoria</summary>
  completions = numero totale esecuzioni

parallelism = quante contemporaneamente

da non confondere con replicas.
</details>

---

### 86 CronJob ogni minuto
**Esercizio:** Create CronJob minute-job that runs every minute.

<details>
<summary>Soluzione</summary>

```bash
kubectl create cronjob minute-job \
  --image=busybox \
  --schedule="* * * * *" \
  -- echo hello
```
</details>
<details>
  <summary>Teoria</summary>
  Ogni minuto = * * * * *
</details>

---

### 87 NetworkPolicy namespace specifico
**Esercizio:** Allow ingress to pods app=api only from namespace frontend.

<details>
<summary>Soluzione</summary>

```bash
kubectl label ns frontend name=frontend
```

Policy:

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
    - namespaceSelector:
        matchLabels:
          name: frontend
```
</details>
<details>
  <summary>Teoria</summary>
  NetworkPolicy funziona su label.

 Namespace deve avere la label usata nel selector.

Senza CNI che supporta policy → non funziona.
</details>

---

### 88 Pod con init container
**Esercizio:** Init container crea file prima dell’avvio del container principale.

<details>
<summary>Soluzione</summary>

```bash
volumes:
- name: data
  emptyDir: {}

initContainers:
- name: init
  image: busybox
  command: ["sh","-c","echo hello > /data/file.txt"]
  volumeMounts:
  - name: data
    mountPath: /data

containers:
- name: app
  image: nginx
  volumeMounts:
  - name: data
    mountPath: /data
```
</details>
<details>
  <summary>Teoria</summary>
  Init container gira prima

Deve montare lo stesso volume

emptyDir condiviso tra container
</details>

---

### 89 Limitare memoria
**Esercizio:** Set memory limit 128Mi.

<details>
<summary>Soluzione</summary>

```bash
resources:
  limits:
    memory: 128Mi
```
</details>
<details>
  <summary>Teoria</summary>
  Unità valide:

Mi

Gi

Senza unità → errore di validazione.
</details>

---

### 90 Service per Deployment
**Esercizio:** Expose Deployment backend internally on port 8080.

<details>
<summary>Soluzione</summary>

```bash
kubectl expose deployment backend \
  --port=8080 \
  --type=ClusterIP
```
</details>
<details>
  <summary>Teoria</summary>
  port = Service

targetPort = container (default stesso valore)

 Se container usa altra porta → specificare targetPort.
</details>

---

### 91 Rimuovere Pod stuck
**Esercizio:** Force delete a Pod named mypod that is stuck in Terminating state.

<details>
<summary>Soluzione</summary>

```bash
kubectl delete pod mypod --grace-period=0 --force
```
</details>
<details>
  <summary>Teoria</summary>
  --grace-period=0 → salta il tempo di attesa
--force → rimuove immediatamente l'oggetto dal cluster

 Usare solo quando il Pod è bloccato in Terminating.
</details>

---

### 92 Aggiungere annotation rollout
**Esercizio:** Add annotation description="v2 release" to Deployment api.

<details>
<summary>Soluzione</summary>

```bash
kubectl annotate deployment api description="v2 release"
```
</details>
<details>
  <summary>Teoria</summary>
  Le annotation:

Non sono usate nei selector

Non influenzano scheduling

Servono come metadata descrittivi
</details>

---

### 93 Impostare nodeSelector
**Esercizio:** Configure a Pod to run only on nodes labeled disktype=ssd.

<details>
<summary>Soluzione</summary>

Nel Pod:

```bash
nodeSelector:
  disktype: ssd
```

Se necessario:

```bash
kubectl label node <node-name> disktype=ssd
```
</details>
<details>
  <summary>Teoria</summary>
  nodeSelector forza lo scheduling su nodi con quella label.

 Se nessun nodo ha la label → Pod resta Pending.
</details>

---

### 94 Strategy Recreate
**Esercizio:** Update a Deployment so that it uses Recreate strategy instead of RollingUpdate.

<details>
<summary>Soluzione</summary>

Nel Deployment:

```bash
strategy:
  type: Recreate
```
</details>
<details>
  <summary>Teoria</summary>
  Recreate:

Termina tutti i Pod esistenti

Poi crea i nuovi

RollingUpdate (default) invece aggiorna gradualmente.
</details>

---

### 95 Exec in container specifico
**Esercizio:** Execute a shell inside container sidecar in a multi-container Pod named podname.

<details>
<summary>Soluzione</summary>

```bash
kubectl exec -it podname -c sidecar -- sh
```
</details>
<details>
  <summary>Teoria</summary>
  In Pod multi-container:

-c è obbligatorio

Senza -c → errore se ci sono più container
</details>

---

### 96 PVC 2Gi
**Esercizio:** Create a PersistentVolumeClaim requesting 2Gi of storage.

<details>
<summary>Soluzione</summary>

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
      storage: 2Gi
```
</details>
<details>
  <summary>Teoria</summary>
  storage: 2Gi deve avere unità valida.

 Il PVC verrà Bound solo se esiste una StorageClass compatibile.
</details>

---

### 97 Montare PVC
**Esercizio:** Mount a PersistentVolumeClaim named mypvc inside a Pod.

<details>
<summary>Soluzione</summary>

Nel Pod:
```bash
volumes:
- name: data
  persistentVolumeClaim:
    claimName: mypvc

containers:
- name: app
  image: nginx
  volumeMounts:
  - name: data
    mountPath: /data
```
</details>
<details>
  <summary>Teoria</summary>
  Il PVC deve essere:

kubectl get pvc

Status = Bound.
</details>

---

### 98 imagePullPolicy Always
**Esercizio:** Configure a container to always pull the image.

<details>
<summary>Soluzione</summary>

```bash
imagePullPolicy: Always
```
</details>
<details>
  <summary>Teoria</summary>
  Default:

latest → Always

tag specifico → IfNotPresent

Forzare Always garantisce aggiornamento ad ogni restart.
</details>

---

### 99 Ridurre revisionHistoryLimit
**Esercizio:** Limit a Deployment to keep only 1 old ReplicaSet revision.

<details>
<summary>Soluzione</summary>

Nel Deployment:

```bash
revisionHistoryLimit: 1
```
</details>
<details>
  <summary>Teoria</summary>
  Riduce il numero di ReplicaSet salvati per rollback.

Utile per cluster con risorse limitate.
</details>

---

### 100 Debug Service non funzionante
**Esercizio:** Investigate why a Service is not routing traffic to its Pods.

<details>
<summary>Soluzione</summary>

```bash
kubectl get pods --show-labels
kubectl get svc
kubectl get endpoints
kubectl describe svc <service-name>
```
</details>
<details>
  <summary>Teoria</summary>
  Se:

endpoints: <none>

→ Il selector del Service non combacia con le label dei Pod.

Checklist rapida:

Pod Running?

Label corrette?

targetPort corretto?

Namespace corretto?

Il Service non guarda il Deployment, ma solo i Pod tramite selector.
</details>

---
