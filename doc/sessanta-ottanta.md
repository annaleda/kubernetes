# 📘 CKAD – Esercizi e Soluzioni

---

### 61 Deployment con replicas
**Esercizio:** Create a Deployment named api with 4 replicas using image nginx.
<details>
<summary>Soluzione</summary>

```bash
kubectl create deployment api --image=nginx --replicas=4

# Verifica
kubectl get deploy api
kubectl get pods
```
</details>
<details>
  <summary>Teoria</summary>
replicas definisce quanti Pod devono essere mantenuti attivi.

Il Deployment crea un ReplicaSet che garantisce il numero desiderato.

Se un Pod muore, viene automaticamente ricreato.
</details>

---

### 62 Scalare un Deployment
**Esercizio:** Scale Deployment api to 2 replicas.
<details>
<summary>Soluzione</summary>

```bash
kubectl scale deployment api --replicas=2

# Verifica
kubectl get pods
```
</details>
<details>
  <summary>Teoria</summary>
kubectl scale modifica il campo:

spec.replicas

Il ReplicaSet elimina o crea Pod per raggiungere il nuovo stato desiderato.
</details>

---


### 63 Rollout status
**Esercizio:** Check rollout status of Deployment webapp.

<details>
<summary>Soluzione</summary>

```bash
kubectl rollout status deployment webapp
```
</details>
<details>
  <summary>Teoria</summary>
  Mostra lo stato dell’aggiornamento in corso.

Utile per verificare se una RollingUpdate è completata con successo.
</details>

---

### 64 Rollback Deployment
**Esercizio:** Rollback Deployment webapp to previous revision.

<details>
<summary>Soluzione</summary>

```bash
kubectl rollout undo deployment webapp
```
</details>
<details>
  <summary>Teoria</summary>
  Il Deployment mantiene uno storico delle revisioni.

rollout undo ripristina la revisione precedente.

Funziona solo se revisionHistoryLimit > 0.
</details>

---

### 65 ConfigMap come env singola chiave
**Esercizio:** Use key APP_MODE from ConfigMap app-config as environment variable.

<details>
<summary>Soluzione</summary>

Nel container:
```bash
env:
- name: APP_MODE
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_MODE
```
</details>
<details>
  <summary>Teoria</summary>
  configMapKeyRef importa una singola chiave.

Differenza:

envFrom → tutte le chiavi
configMapKeyRef → chiave specifica
</details>

---

### 66 Resource request CPU e memoria
**Esercizio:** Add CPU request 200m and memory request 128Mi to a container.

<details>
<summary>Soluzione</summary>

Nel container:
```bash
resources:
  requests:
    cpu: 200m
    memory: 128Mi
```
</details>
<details>
  <summary>Teoria</summary>
  requests influenzano lo scheduling.

Il nodo deve avere risorse disponibili sufficienti.

Se non soddisfatte → Pod resta Pending.
</details>

---

### 67 Liveness Probe HTTP
**Esercizio:** Add HTTP liveness probe on path /health port 8080.

<details>
<summary>Soluzione</summary>

```bash
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```
</details>
<details>
  <summary>Teoria</summary>
  Liveness determina se il container deve essere riavviato.

Se fallisce ripetutamente → Kubernetes esegue restart.

Diversa dalla Readiness che controlla solo il traffico.
</details>

---

### 68 Service ClusterIP manuale
**Esercizio:** Create a Service with type ClusterIP explicitly defined.

<details>
<summary>Soluzione</summary>

```bash
spec:
  type: ClusterIP
```
</details>
<details>
  <summary>Teoria</summary>
  ClusterIP è il tipo default.

Espone il servizio solo internamente al cluster.

Non accessibile dall’esterno.
</details>

---

### 69 Headless Service
**Esercizio:** Create a headless Service for app=db.

<details>
<summary>Soluzione</summary>

Nel service:
```bash
clusterIP: None
selector:
  app: db
```
</details>
<details>
  <summary>Teoria</summary>
  Un headless Service non assegna IP virtuale.

Restituisce direttamente gli IP dei Pod via DNS.

Utile per StatefulSet o discovery avanzato.
</details>

---

### 70 Pod con multiple container
**Esercizio:** Create a Pod with two containers: nginx and busybox.

<details>
<summary>Soluzione</summary>

Nel Pod:
```bash
containers:
- name: nginx
  image: nginx
- name: busybox
  image: busybox
  command: ["sleep", "3600"]
```
</details>
<details>
  <summary>Teoria</summary>
  I container nello stesso Pod:

Condividono rete (localhost)

Possono condividere volumi

Pattern comune: sidecar.
</details>

---

### 71 Init Container
**Esercizio:** Add an initContainer that runs before the main container.

<details>
<summary>Soluzione</summary>

```bash
initContainers:
- name: init-myservice
  image: busybox
  command: ["sh", "-c", "echo init"]
```
</details>
<details>
  <summary>Teoria</summary>
  Gli initContainers:

Eseguono prima dei container principali

Devono completare con successo

Se falliscono → Pod non parte.
</details>

---

### 72 Shared volume tra container
**Esercizio:** Share an emptyDir volume between two containers.

<details>
<summary>Soluzione</summary>

```bash
volumes:
- name: shared-data
  emptyDir: {}

containers:
- name: app1
  volumeMounts:
  - name: shared-data
    mountPath: /data

- name: app2
  volumeMounts:
  - name: shared-data
    mountPath: /data
```
</details>
<details>
  <summary>Teoria</summary>
  emptyDir è condiviso tra container nello stesso Pod.

I dati persistono finché il Pod è in vita.
</details>

---

### 73 Impostare ServiceAccount
**Esercizio:** Configure a Pod to use ServiceAccount custom-sa.

<details>
<summary>Soluzione</summary>

Nel pod spec:
```bash
serviceAccountName: custom-sa
```
</details>
<details>
  <summary>Teoria</summary>
  Ogni Pod usa un ServiceAccount.

Default = default.

Serve per autenticazione verso API Kubernetes.
</details>

---

### 74 Disabilitare automount token
**Esercizio:** Disable automatic mounting of ServiceAccount token in a Pod.

<details>
<summary>Soluzione</summary>

Nel Pod:
```bash
automountServiceAccountToken: false
```
</details>
<details>
  <summary>Teoria</summary>
  Di default il token viene montato come volume.

Disabilitarlo aumenta sicurezza se non serve accesso API.
</details>

---

### 75 LimitRange verifica
**Esercizio:** Check if a LimitRange is affecting Pod creation.

<details>
<summary>Soluzione</summary>

```bash
kubectl describe limitrange -n <namespace>
```
</details>
<details>
  <summary>Teoria</summary>
  LimitRange impone:

Min/max CPU

Min/max memoria

Default requests/limits

Può causare errori se non rispettato.
</details>

---

### 76 PriorityClass
**Esercizio:** Assign a PriorityClass high-priority to a Pod.

<details>
<summary>Soluzione</summary>

Nel Pod:
```bash
priorityClassName: high-priority
```
</details>
<details>
  <summary>Teoria</summary>
  I Pod con priorità più alta:

Vengono schedulati prima

Possono causare preemption di Pod meno prioritari.
</details>

---

### 77 Impostare resource limit memoria
**Esercizio:** Add memory limit 256Mi to a container.

<details>
<summary>Soluzione</summary>

```bash
resources:
  limits:
    memory: 256Mi
```
</details>
<details>
  <summary>Teoria</summary>
  Se il container supera il memory limit:

Viene terminato (OOMKilled).

Diverso dal CPU limit che causa throttling.
</details>

---

### 78 Port-forward
**Esercizio:** Forward local port 8080 to Pod web-pod port 80.

<details>
<summary>Soluzione</summary>

```bash
kubectl port-forward pod/web-pod 8080:80
```
</details>
<details>
  <summary>Teoria</summary>
  Permette accesso temporaneo a un Pod locale.

Utile per debug.

Non modifica Service o configurazione cluster.
</details>

---

### 79 Copiare file dentro Pod
**Esercizio:** Copy file config.yaml into Pod web-pod at /tmp.

<details>
<summary>Soluzione</summary>

```bash
kubectl cp config.yaml web-pod:/tmp
```
</details>
<details>
  <summary>Teoria</summary>
  kubectl cp usa tar internamente.

Il container deve avere tar installato.

Funziona anche al contrario (dal Pod al locale).
</details>

---

### 80 Eliminare Deployment mantenendo Pod
**Esercizio:** Delete Deployment api without deleting its Pods.

<details>
<summary>Soluzione</summary>

```bash
kubectl delete deployment api --cascade=orphan
```
</details>
<details>
  <summary>Teoria</summary>
  --cascade=orphan rimuove solo il Deployment.

I ReplicaSet e Pod restano attivi.

Utile in casi di migrazione o troubleshooting.
</details>

---

