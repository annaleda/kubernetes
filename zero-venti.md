
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
  In Kubernetes il Pod è l’unità minima deployabile.

kubectl run crea un Pod (nelle versioni moderne, senza --restart, crea un Pod semplice).

Il Pod contiene uno o più container.

redis:7 specifica il tag dell’immagine → importante per controllo versione.

 Best practice: specificare sempre il tag, evitare latest.
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
  Un Deployment:

Gestisce ReplicaSet

Garantisce numero desiderato di Pod

Supporta rolling update e rollback

Con --replicas=3 Kubernetes mantiene sempre 3 Pod attivi (self-healing).
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
  Lo scaling modifica:

spec.replicas

Il ReplicaSet crea o elimina Pod per raggiungere lo stato desiderato.

È scaling manuale (diverso da HPA).
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
  Aggiornare l’immagine modifica il template del Pod → genera nuovo ReplicaSet → parte una Rolling Update.

Rolling update:

crea nuovi Pod

termina gradualmente i vecchi

zero downtime (di default)
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
  Il Deployment mantiene una revision history.

Rollback:

Ripristina ReplicaSet precedente

Utile se nuova versione fallisce

Puoi vedere revisioni con:

kubectl rollout history deployment api
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
  Un Service ClusterIP:

Espone internamente nel cluster

Crea IP virtuale stabile

Usa label selector per trovare Pod

Flusso:

Pod → Service → Pod target

Il selector deve combaciare con le label del Deployment.
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
  Una ConfigMap contiene configurazione non sensibile.

Può essere montata come:

variabile ambiente

volume

file di configurazione

Permette di separare config dal container image.
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
  Un Secret è simile a ConfigMap ma per dati sensibili.

Base64 encoded (non cifrato di default)

Può essere montato come env o volume

Può essere cifrato at-rest con configurazione cluster
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
  valueFrom permette di leggere dinamicamente valori.

Vantaggi:

Nessuna modifica immagine

Cambio config senza rebuild

Ambiente separato (dev/prod)
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
  La Liveness Probe controlla se l’app è viva.

Se fallisce:

Il container viene riavviato

Diversa dalla Readiness:

Liveness → restart

Readiness → rimozione dal Service
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
  requests → scheduler decide dove mettere il Pod

limits → massimo utilizzabile

Se supera:

CPU → throttling

Memory → OOMKill

Fondamentale in ambienti multi-tenant.
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
  Gli initContainers:

Eseguono prima dei container principali

Devono completare con successo

Eseguono in ordine

Utili per:

Migrazioni DB

Check dipendenze

Setup iniziale
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
  Pattern Sidecar:

Container principale → app

Sidecar → logging, proxy, monitoring

Condividono:

Network namespace

Storage (volumi)
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
  Un PersistentVolumeClaim:

Richiede storage

Si lega a un PersistentVolume

Astrarre storage dal Pod

Access modes:

RWO → un nodo alla volta

RWX → multi nodo
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
  Il PVC viene montato come volume nel Pod.

Separazione chiave:

PVC → richiesta storage
Pod → utilizzo storage

Se il Pod muore, i dati restano.
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
  Un Job esegue task batch fino a completamento.

Diverso da Deployment:

Non mantiene Pod sempre attivi

Si considera completato quando successo
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
  Un CronJob crea Job su schedule cron.

Formato cron:

* * * * *
| | | | |
m h dom mon dow
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
  SecurityContext controlla privilegi container.

runAsUser → evita root

allowPrivilegeEscalation: false → blocca escalation
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
  Namespace:

Isolamento logico

Quote e policy separate

Evita conflitti nomi
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
  Una NetworkPolicy:

È deny-all implicito se presente

Filtra traffico tra Pod

Richiede CNI compatibile

podSelector seleziona i Pod protetti
from definisce chi può entrare
</details>

