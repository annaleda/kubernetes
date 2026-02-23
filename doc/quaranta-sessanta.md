# 📘 CKAD – Esercizi e Soluzioni

---

### 41 Pod con restartPolicy Never
**Esercizio:**  Create a Pod one-shot with image busybox that runs echo done and does not restart.
<details>
<summary>Soluzione</summary>

```bash
kubectl run one-shot \
  --image=busybox \
  --restart=Never \
  -- echo done

# Verifica
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>
Di default kubectl run può creare un Deployment a seconda delle opzioni.

Con --restart=Never forzi la creazione di un Pod standalone.

Quando il comando termina, il Pod va in stato Completed e non viene riavviato.

Senza --restart=Never → rischio di creare un Deployment.

</details>

---

### 42 Deployment in namespace esistente
**Esercizio:** Deploy nginx in namespace staging.
<details>
<summary>Soluzione</summary>

```bash
kubectl create deployment nginx --image=nginx -n staging

# Verifica
kubectl get pods -n staging
```
</details>
<details>
  <summary>Teoria</summary>
In Kubernetes, tutte le risorse sono create in un namespace.

Se il namespace non esiste, il comando fallisce.

Il flag -n specifica il namespace target.

Alternativa: impostare il namespace nel context kubectl.
</details>

---

### 43 Limitare revisioni rollout
**Esercizio:** Keep only 2 rollout revisions in a Deployment.
<details>
<summary>Soluzione</summary>
  
Nel Deployment aggiungere:

```bash
revisionHistoryLimit: 2
```

Applicare:
```bash
kubectl apply -f deploy.yaml

# Verifica
kubectl rollout history deployment <name>
```
</details>
<details>
  <summary>Teoria</summary>
Un Deployment mantiene uno storico delle revisioni per permettere il rollback.

Il campo revisionHistoryLimit controlla quante revisioni salvare.

Default = 10.

Ridurre questo numero limita l’uso di risorse.
</details>

---

### 44 Multiple env da Secret
**Esercizio:** Import all keys from Secret db-secret as environment variables.
<details>
<summary>Soluzione</summary>
  
Nel container:

```bash
envFrom:
- secretRef:
    name: db-secret
```
</details>
<details>
  <summary>Teoria</summary>
envFrom importa tutte le chiavi del Secret come variabili ambiente.

Differenza:

env → singola chiave

envFrom → tutte le chiavi

Ogni chiave del Secret diventa una variabile con lo stesso nome.

</details>
</details>

---

### 45 Deployment con containerPort
**Esercizio:** 
<details>
<summary>Soluzione</summary>

Nel container:

```bash
ports:
- containerPort: 8080
```
</details>
<details>
  <summary>Teoria</summary>
containerPort è informativo.

Non apre realmente la porta nel cluster.

Serve per:

documentazione

generazione automatica di Service

chiarezza configurazione

Il traffico è comunque gestito dal Service.
</details>

---

### 46 Service con porta diversa
**Esercizio:** Expose a Deployment where the container listens on 8080 but the Service must expose port 80.
<details>
<summary>Soluzione</summary>
Nel Service:
  
```bash
ports:
- port: 80
  targetPort: 8080
```
Applicare:

```bash
kubectl apply -f svc.yaml

# Verifica
kubectl get svc
kubectl describe svc <name>
```
</details>
<details>
  <summary>Teoria</summary>
Nel Service:

port → porta esposta dal Service

targetPort → porta del container

Se targetPort non combacia con la porta del container, il traffico non arriva.

Il Service fa da proxy verso i Pod selezionati tramite label.
</details>

---

### 47 Job con backoffLimit
**Esercizio:** Configure a Job with backoffLimit set to 4.
<details>
<summary>Soluzione</summary>
Nel Job:


```bash
backoffLimit: 4
```

Applicare:

```bash
kubectl apply -f job.yaml

# Verifica
kubectl get job -o yaml
```
</details>
<details>
  <summary>Teoria</summary>
backoffLimit definisce quante volte un Job può fallire prima di essere marcato come Failed.

Default = 6.

Utile per evitare retry infiniti.
</details>

---

### 48 CronJob con history limit
**Esercizio:** Configure a CronJob keeping 3 successful Jobs and 1 failed Job.
<details>
<summary>Soluzione</summary>
Nel CronJob:


```bash
successfulJobsHistoryLimit: 3
failedJobsHistoryLimit: 1
```
Applicare:

```bash
kubectl apply -f cronjob.yaml

# Verifica
kubectl get cronjob -o yaml
```
</details>
<details>
  <summary>Teoria</summary>
Un CronJob crea Job periodicamente.

Senza limiti di history, i Job restano salvati nel cluster.

Questi campi controllano la pulizia automatica.
</details>

---

### 49 Pod con runAsNonRoot
**Esercizio:** Configure a Pod to run as non-root.
<details>
<summary>Soluzione</summary>
Nel Pod o container:


```bash
securityContext:
  runAsNonRoot: true
```
</details>
<details>
  <summary>Teoria</summary>
runAsNonRoot: true forza il container a non usare UID 0.

Se l'immagine richiede root → il Pod fallisce.

È una best practice di sicurezza.
</details>

---

### 50 Aggiungere annotation a Deployment
**Esercizio:** Add annotation owner=team-a to Deployment webapp.
<details>
<summary>Soluzione</summary>

```bash
kubectl annotate deployment webapp owner=team-a

# Verifica
kubectl get deploy webapp -o yaml
```
</details>
<details>
  <summary>Teoria</summary>
Le annotation sono metadata non usati per selezione o scheduling.

Differenza:

Label → usate nei selector

Annotation → informazioni descrittive

Non influenzano il comportamento del Deployment.
</details>

---

### 51 Taint & Toleration
**Esercizio:** Allow a Pod to tolerate a NoSchedule taint.
<details>
<summary>Soluzione</summary>
Nel Pod:


```bash
tolerations:
- key: "key1"
  operator: "Exists"
  effect: "NoSchedule"
```
</details>
<details>
  <summary>Teoria</summary>
Se un nodo ha un taint:

key1=value1:NoSchedule

I Pod senza toleration non vengono schedulati su quel nodo.

La toleration permette al Pod di ignorare il taint.
</details>

---

### 52 NodeSelector
**Esercizio:** Schedule a Pod only on nodes with label disktype=ssd.
<details>
<summary>Soluzione</summary>
Nel Pod:


```bash
nodeSelector:
  disktype: ssd


#Verifica
kubectl get nodes --show-labels
```
</details>
<details>
  <summary>Teoria</summary>
nodeSelector è il metodo più semplice di scheduling su nodi specifici.

Se nessun nodo ha quella label → Pod resta Pending.

Funziona solo su label dei nodi.
</details>

---

### 53 Pod Anti-Affinity
**Esercizio:** Avoid scheduling Pods with label app=web on the same node. 
<details>
<summary>Soluzione</summary>
Nel Pod:
  
```bash
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: web
      topologyKey: "kubernetes.io/hostname"
```
</details>
<details>
  <summary>Teoria</summary>
Pod Anti-Affinity evita che Pod con certe label siano sullo stesso nodo.

topologyKey definisce il dominio (di solito hostname).

requiredDuringScheduling = obbligatorio.
</details>

---

### 54 Verificare ResourceQuota
**Esercizio:** Investigate why a Pod cannot be created in namespace dev.
<details>
<summary>Soluzione</summary>

```bash
kubectl describe quota -n dev
```
</details>
<details>
  <summary>Teoria</summary>
Una ResourceQuota limita:

Numero Pod

CPU

Memoria

PVC

Se superi il limite → errore di creazione.

Molto comune nei task di troubleshooting.
</details>

---

### 55 imagePullPolicy Always
**Esercizio:** Force container to always pull image.
<details>
<summary>Soluzione</summary>
Nel container:
  
```bash
imagePullPolicy: Always
```
</details>
<details>
  <summary>Teoria</summary>
Default behavior:

Se tag = latest → Always

Se tag specifico → IfNotPresent

Impostare Always forza il pull ad ogni restart.
</details>

---

### 56 ConfigMap da file
**Esercizio:** Create a ConfigMap from config.txt file.
<details>
<summary>Soluzione</summary>

```bash
kubectl create configmap app-config --from-file=config.txt

# Verifica
kubectl describe configmap app-config
```
</details>
<details>
  <summary>Teoria</summary>
--from-file crea una chiave con nome uguale al file.

Il contenuto del file diventa il valore della chiave.

Utile per configurazioni multi-linea.
</details>

---

### 57 Secret da file
**Esercizio:** Create Secret tls-secret from file tls.crt.
<details>
<summary>Soluzione</summary>

```bash
kubectl create secret generic tls-secret --from-file=tls.crt

# Verifica
kubectl get secret tls-secret -o yaml
```
</details>
<details>
  <summary>Teoria</summary>
Il contenuto del file viene codificato in base64.

Può essere montato come volume o usato in un Ingress TLS.

Non è cifrato di default, solo encoded.
</details>

---

### 58 Patch veloce
**Esercizio:** Add label env=prod to Deployment api.
<details>
<summary>Soluzione</summary>

```bash
kubectl patch deployment api \
  -p '{"metadata":{"labels":{"env":"prod"}}}'

# Verifica
kubectl get deploy api --show-labels
```
</details>
<details>
  <summary>Teoria</summary>
kubectl patch permette modifiche rapide senza edit completo.

Attenzione:

Patch su metadata.labels modifica solo il Deployment

Non modifica automaticamente spec.template.metadata.labels
</details>

---

### 59 Container in CrashLoop
**Esercizio:** Investigate why a container is crashing.
<details>
<summary>Soluzione</summary>

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
```
</details>
<details>
  <summary>Teoria</summary>
Debug flow tipico:

Stato Pod

Eventi (describe)

Logs container

Errori comuni:

Image name sbagliato

Porta errata

Variabile mancante

OOMKilled
</details>

---

### 60 Service non raggiungibile
**Esercizio:** Fix a Service that is not routing traffic.
<details>
<summary>Soluzione</summary>

```bash
kubectl get endpoints
kubectl describe svc <name>
kubectl get pods --show-labels
```
</details>
<details>
  <summary>Teoria</summary>
Se un Service non funziona:

I Pod sono Running?

Le label combaciano?

targetPort corretto?

Namespace corretto?

Se kubectl get endpoints mostra <none> → selector errato.

Il Service non guarda il Deployment, ma solo i Pod tramite selector.
</details>

---

