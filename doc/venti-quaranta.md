# 📘 CKAD – Esercizi e Soluzioni

---

### 21 Pod in namespace specifico
**Esercizio:**  Create a Pod busybox-pod with image busybox in namespace test-ns.
<details>
<summary>Soluzione</summary>

```bash
kubectl create ns test-ns
kubectl run busybox-pod --image=busybox -n test-ns -- sleep 3600

#Verifica
kubectl get pods -n test-ns
```
</details>
<details>
  <summary>Teoria</summary>
In Kubernetes, i namespace servono per isolare risorse logicamente nello stesso cluster.

Se non specifichi -n, il Pod viene creato nel namespace default.

sleep 3600 mantiene il container attivo (busybox altrimenti termina subito).

Il namespace deve esistere prima della creazione del Pod.

</details>

---

### 22 Deployment con label personalizzata
**Esercizio:**  Create Deployment webapp with label tier=backend.
<details>
<summary>Soluzione</summary>

```bash
kubectl create deployment webapp --image=nginx --dry-run=client -o yaml > webapp.yaml
```

Aggiungere sia in metadata che in spec.template.metadata:

```bash
labels:
  tier: backend
```

```bash
kubectl apply -f webapp.yaml

#Attenzione!La label deve essere nel Deployment e nel template
```
</details>
<details>
  <summary>Teoria</summary>
Un Deployment crea e gestisce ReplicaSet → Pod.

La label deve essere nel:

metadata del Deployment (informativa)

spec.template.metadata (fondamentale!)

Perché?

Il Service e il ReplicaSet selezionano i Pod tramite label selector, che agisce sul template del Pod, non sul Deployment stesso.

</details>

---

### 23 Service con NodePort specifico
**Esercizio:** Expose webapp on NodePort 30007.
<details>
<summary>Soluzione</summary>

```bash
kubectl expose deployment webapp \
  --port=80 \
  --type=NodePort \
  --dry-run=client -o yaml > svc.yaml
```

Nel file aggiungere:

```bash
nodePort: 30007
```
```bash
kubectl apply -f svc.yaml

#Attenzione!Range valido: 30000–32767
```

</details>

<details>
  <summary>Teoria</summary>
Un Service NodePort espone un'applicazione su una porta del nodo.

Flusso traffico:

Client → NodeIP:30007 → Service → Pod

Range valido: 30000–32767.

Se non specifichi nodePort, Kubernetes ne assegna uno automaticamente.
</details>

---

### 24 Cambiare image container con nome diverso

**Esercizio:** Update container nginx-container in deployment webapp.

<details> <summary>Soluzione</summary>

Prima verifico nome container:

```bash
kubectl get deploy webapp -o yaml
```
Poi:
```bash
kubectl set image deployment/webapp nginx-container=nginx:1.26
```

</details>

<details>
  <summary>Teoria</summary>

  kubectl set image aggiorna il campo:

spec.template.spec.containers[].image

Questo modifica il template → genera un nuovo ReplicaSet → avvia una Rolling Update automatica.

Se il nome del container è sbagliato, il comando fallisce.

</details>

---

### 25 Pod con command personalizzato

**Esercizio:** Create Pod custom-pod running echo hello.

<details> <summary>Soluzione</summary>

```bash
kubectl run custom-pod \
  --image=busybox \
  --restart=Never \
  -- echo hello

#Attenzione! --restart=Never per creare Pod e non Deployment
```

</details>

<details>
  <summary>Teoria</summary>
kubectl run crea:

Deployment (default)

Pod se --restart=Never

-- echo hello sovrascrive il CMD dell’immagine.

Senza --restart=Never, verrebbe creato un Deployment.
</details>

---

### 26 Pod con env statico

**Esercizio:** Add env var LOG_LEVEL=debug.

<details> <summary>Soluzione</summary>

```bash
kubectl run env-pod \
  --image=nginx \
  --env="LOG_LEVEL=debug"
```

</details>

<details>
  <summary>Teoria</summary>

Le variabili d’ambiente sono definite in:

spec.containers.env

Servono per:

configurazione runtime

differenziare ambienti

evitare rebuild dell’immagine
</details>

---

### 27 Deployment con Readiness Probe

**Esercizio:** Add HTTP readiness probe.

<details> <summary>Soluzione</summary>

```bash
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10

#Nota: Readiness ≠ Liveness
```

</details>

<details>
  <summary>Teoria</summary>
Readiness ≠ Liveness

Readiness → determina se il Pod può ricevere traffico

Liveness → determina se il Pod deve essere riavviato

Se la readiness fallisce:

Il Pod resta Running

Viene rimosso dagli endpoint del Service
</details>

---

### 28 Limitare solo CPU limit

**Esercizio:** Add CPU limit 500m.

<details> <summary>Soluzione</summary>

```bash
resources:
  limits:
    cpu: 500m

#Nota:Se non specifichi request → spesso default = limit
```

</details>

<details>
  <summary>Teoria</summary>
500m = 0.5 CPU core.

requests → scheduling

limits → massimo consentito

Se superi il CPU limit → throttling (non kill).
</details>

---

### 29 Pod con emptyDir

**Esercizio:** Create a Pod named cache-pod using image nginx that mounts an emptyDir volume at /cache.

<details> <summary>Soluzione</summary>
  
```bash
volumes:
- name: cache
  emptyDir: {}

containers:
- name: nginx
  image: nginx
  volumeMounts:
  - mountPath: /cache
    name: cache
```

</details>

<details>
  <summary>Teoria</summary>
emptyDir:

Creato quando il Pod parte

Distrutto quando il Pod muore

Condiviso tra container nello stesso Pod

È storage ephemeral.
</details>

---

### 30 Job con completions

**Esercizio:** Create a Job named batch-job using image busybox that runs 3 completions sequentially.

<details> <summary>Soluzione</summary>

```bash
kubectl create job batch-job --image=busybox \
  --dry-run=client -o yaml > job.yaml
```

Aggiungere:

```bash
completions: 3
parallelism: 1
```  

```bash
kubectl apply -f job.yaml
```  

</details>

<details>
  <summary>Teoria</summary>
In un Job:

completions → quante volte deve completare con successo

parallelism → quanti Pod contemporaneamente

Con parallelism: 1 → esecuzione sequenziale.
</details>

---

### 31 CronJob sospeso

**Esercizio:** Create a CronJob that is initially suspended.

<details> <summary>Soluzione</summary>

Nel spec:

```bash
suspend: true

#Molto comune al CKAD
```  


</details>

<details>
  <summary>Teoria</summary>
Un CronJob crea Job basati su schedule cron.

suspend: true:

Non esegue nuovi Job

Mantiene la configurazione pronta
</details>

---

### 32 Trovare perché Pod non parte

**Esercizio:** Investigate why a Pod is not starting and identify the root cause.

<details> <summary>Soluzione</summary>

```bash
kubectl describe pod <name>
kubectl logs <name>

#CKAD può chiedere: “Fix the pod”
```    

</details>

<details>
  <summary>Teoria</summary>
describe → eventi (ImagePullBackOff, CrashLoopBackOff, etc.)

logs → output applicazione

Debug flow tipico:

Stato Pod

Eventi

Logs

Config (env, volumi, image)
</details>

---

### 33 Deployment con strategy Recreate

**Esercizio:** Configure a Deployment to use Recreate strategy instead of RollingUpdate.

<details> <summary>Soluzione</summary>

```bash
strategy:
  type: Recreate
```

</details>

<details>
  <summary>Teoria</summary>
Strategie disponibili:

RollingUpdate (default)

Recreate

Recreate:

Termina tutti i Pod

Crea nuovi Pod

Causa downtime, ma evita versioni miste.
</details>

---

### 34 Rolling Update configurato

**Esercizio:** Configure a Deployment with RollingUpdate strategy allowing maxUnavailable=1 and maxSurge=1.

<details> <summary>Soluzione</summary>

```bash
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

</details>

<details>
  <summary>Teoria</summary>
maxUnavailable → quanti Pod possono essere down

maxSurge → quanti Pod extra possono essere creati

Controlla disponibilità e velocità di rollout.
</details>

---

### 35 Secret montato come volume

**Esercizio:** Mount the Secret db-secret inside a Pod at path /etc/secret.

<details> <summary>Soluzione</summary>

```bash
volumes:
- name: secret-vol
  secret:
    secretName: db-secret

containers:
- name: app
  volumeMounts:
  - name: secret-vol
    mountPath: /etc/secret
```    

</details>

<details>
  <summary>Teoria</summary>
Un Secret può essere:

variabile d’ambiente

volume

Come volume:

Ogni chiave → file

Aggiornamenti propagati automaticamente
</details>

---

### 36 Service con selector manuale

**Esercizio:** Troubleshoot a Service that is not routing traffic to Pods.

<details> <summary>Soluzione</summary>

Se il Service non funziona:

```bash
kubectl get pods --show-labels

#Controllare che il selector del Service combaci con le label dei Pod.
```  


</details>

<details>
  <summary>Teoria</summary>
Un Service funziona solo se:

spec.selector

corrisponde alle label dei Pod.

Se non combacia → Endpoints vuoti → niente traffico.
</details>

---

### 37 NetworkPolicy solo Egress

**Esercizio:** Create a NetworkPolicy that only allows egress traffic to Pods with label app=db.

<details> <summary>Soluzione</summary>

```bash
policyTypes:
- Egress

egress:
- to:
  - podSelector:
      matchLabels:
        app: db

#Molti dimenticano policyTypes
```    

</details>

<details>
  <summary>Teoria</summary>
Una NetworkPolicy:

È deny-all implicito se definita

Serve CNI compatibile (Calico, Cilium…)

Se dimentichi policyTypes, può non comportarsi come previsto.
</details>

---

### 38 Limitare accesso a specifica porta

**Esercizio:** Allow ingress traffic only on TCP port 80 within a NetworkPolicy.

<details> <summary>Soluzione</summary>

Dentro regola ingress:

```bash
ports:
- protocol: TCP
  port: 80
```    

</details>

<details>
  <summary>Teoria</summary>
Le regole di NetworkPolicy filtrano su:

Pod selector

Namespace selector

Porte

Protocollo

Se specifichi la porta → tutto il resto viene bloccato.
</details>

---

### 39 Impostare namespace di default nel contesto

**Esercizio:** Set the current kubectl context default namespace to dev.

<details> <summary>Soluzione</summary>

```bash
kubectl config set-context --current --namespace=dev
```      

</details>

<details>
  <summary>Teoria</summary>
kubectl usa:

context = cluster + user + namespace

Cambiare namespace nel contesto evita di scrivere -n ogni volta.
</details>

---

### 40 Fix label mismatch

**Esercizio:** Fix a Service that cannot reach its target Pods due to label mismatch.

Problema: Service non raggiunge Pod.

<details> <summary>Soluzione</summary>

Controlla selector del Service

Controlla spec.template.metadata.labels del Deployment

Allinea le label

```bash
kubectl get svc <name> -o yaml
kubectl get deploy <name> -o yaml
```    

</details>

<details>
  <summary>Teoria</summary>
Il Service non guarda il Deployment.

Guarda solo i Pod tramite selector.

Se le label non combaciano:

Nessun endpoint

Nessun traffico
</details>

