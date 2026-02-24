# CKAD Pod Design

## Teoria

---

## Workloads in Kubernetes

Kubernetes offre diversi controller per gestire i Pod:

- Pod (singolo, non gestito)
- Deployment (stateless apps)
- Job (task batch)
- CronJob (task schedulati)
- StatefulSet (app stateful - non focus principale CKAD)

---

## Job

Un Job crea uno o più Pod per eseguire un task batch.

Caratteristiche:
- Esegue fino a completamento
- Può essere configurato con più tentativi
- Utile per script, migrazioni DB, batch processing

Parametri importanti:
- `completions`
- `parallelism`
- `backoffLimit`
- `activeDeadlineSeconds`

Esempio:

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

---

## CronJob

Esegue Job schedulati nel tempo (formato cron).

Formato schedule:
```
* * * * *
│ │ │ │ │
│ │ │ │ └─ Giorno settimana
│ │ │ └─── Mese
│ │ └───── Giorno mese
│ └─────── Ora
└───────── Minuto
```

Esempio:

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

Parametri utili:
- `concurrencyPolicy`
- `successfulJobsHistoryLimit`
- `failedJobsHistoryLimit`
- `startingDeadlineSeconds`

---

## Deployment

Gestisce applicazioni stateless.

<img width="1135" height="661" alt="Immagine 2026-02-24 150103" src="https://github.com/user-attachments/assets/91e15fac-5bbf-489a-bee8-267e82e27d58" />


Funzionalità principali:
- ReplicaSet automatici
- Rolling update
- Rollback
- Scaling
- Self-healing

---

## Strategies: Recreate and Rolling Updates

<img width="1645" height="771" alt="Immagine 2026-02-24 150224" src="https://github.com/user-attachments/assets/988dd560-33e2-4204-bacc-25c2f149d009" />

### Recreate

La strategia **Recreate** termina tutti i Pod esistenti prima di creare quelli nuovi.

Funzionamento:

1. Vengono eliminati tutti i Pod della versione corrente
2. Solo dopo vengono creati i Pod della nuova versione

```yaml
strategy:
  type: Recreate
```

###  Rolling Updates
Aggiornamento progressivo dei Pod senza downtime.

Parametri chiave:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

- `maxUnavailable`: quanti pod possono essere non disponibili
- `maxSurge`: quanti pod extra possono essere creati

---

## Rollback

Permette di tornare alla versione precedente.

Comandi utili:

```
kubectl rollout history deployment <name>
kubectl rollout undo deployment <name>
kubectl rollout status deployment <name>
```
<img width="770" height="478" alt="Immagine 2026-02-24 145707" src="https://github.com/user-attachments/assets/fe49f26b-b902-466b-896d-632e59fd5dd3" />

Rollback a revisione specifica:

```
kubectl rollout undo deployment <name> --to-revision=2
```

---

## Scaling

Modificare numero repliche:

```
kubectl scale deployment web --replicas=5
```

Oppure via YAML modificando:

```yaml
spec:
  replicas: 5
```

---

## Restart Policy

Nei Pod:
- `Always` (default per Deployment)
- `OnFailure`
- `Never`

Nei Job:
- `OnFailure` o `Never`

---

## Esercizi

1. Creare un Job che stampa "Hello CKAD".
2. Impostare backoffLimit a 3.
3. Creare un CronJob che esegue ogni minuto.
4. Impostare concurrencyPolicy a Forbid.
5. Creare un Deployment nginx con 3 repliche.
6. Aggiornare l’immagine a una nuova versione.
7. Verificare rollout status.
8. Eseguire rollback alla versione precedente.
9. Scalare il deployment a 5 repliche.

---


