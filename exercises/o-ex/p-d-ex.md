
### Pod Design (6 esercizi)
---

## PD-1 — Pod Singolo (non gestito)

Creare un Pod chiamato `standalone-pod`

- Container
  - Image: nginx
  - Porta: 80

- Configurazione
  - restartPolicy: Always (default)

- Vincoli
  Non usare Deployment

- Validazione
  - Il Pod è Running
  - Eliminando il Pod non viene ricreato automaticamente

```

```
---

## PD-2 — Deployment con Scaling

Creare un Deployment chiamato `web-deployment`

- Specifiche

Image: nginx:1.22
  - Replicas: 3

- Task successivo
  - Scalare a 5 repliche

- Validazione
  - kubectl get pods mostra 5 pod
  - ReplicaSet aggiornato

```

```
---

## PD-3 — Rolling Update + Rollback

Creare Deployment `api-deployment`

- Specifiche iniziali
  - Image: nginx:1.21
  - Replicas: 4

Aggiornare l’immagine a nginx:1.23. Poi eseguire rollback alla versione precedente

- Vincoli
  - Usare rolling update (default strategy)

- Validazione
  - kubectl rollout history deployment api-deployment
  - Verificare versione immagine dopo rollback

```

```
---

## PD-4 — Job (Batch Task)

Creare un Job chiamato `batch-job`

- Container
  - Image: busybox
  - Command: `sh -c "echo Job executed && sleep 5"`

- Configurazione
  - completions: 1
  - backoffLimit: 2

- Validazione
  - Job completato con stato Complete
  - Pod termina correttamente

```

```
---

## PD-5 — Parallel Job

Creare un Job chiamato `parallel-job`

- Specifiche
  - Image: busybox
  - Command: sleep 10
  - completions: 4
  - parallelism: 2

- Obiettivo
  - Eseguire 2 pod alla volta fino a 4 completamenti

- Validazione
  - Non più di 2 pod Running contemporaneamente
  - Job termina con 4 completamenti

```

```
---

## PD-6 — CronJob (Scheduled Task)

Creare un CronJob chiamato `scheduled-task`

- Specifiche
  - Schedule: ogni 2 minuti
  - Image: busybox
  - Command: date

- Configurazione
  - successfulJobsHistoryLimit: 3
  - failedJobsHistoryLimit: 1

- Validazione
  - Viene creato almeno un Job
  - kubectl get cronjob mostra schedule corretto

```

```
---

## PD-7 — StatefulSet con Storage Persistente

StatefulSet: `db-stateful`

- Specifiche
  - Replicas: 2
  - ServiceName: `db-headless`
  - Image: nginx
  - Porta: 80

- Service richiesto
  - Nome: `db-headless`
  - ClusterIP: None
  - Selector coerente con il StatefulSet

- Storage
  - Utilizzare `volumeClaimTemplates`
  - Nome claim: data
  - Storage richiesto: 100Mi
  - AccessMode: ReadWriteOnce
  - StorageClass: standard

- Obiettivo
  - Ogni replica deve avere il proprio PVC
  - I pod devono avere nomi prevedibili:
    - db-stateful-0
    - db-stateful-1

- Vincoli
  - Non usare Deployment
  - Non creare manualmente i PVC
  - I PVC devono essere generati automaticamente

- Validazione
  - kubectl get pods mostra nomi ordinali
  - kubectl get pvc mostra:
    - data-db-stateful-0
    - data-db-stateful-1
  - Eliminando db-stateful-0 viene ricreato con lo stesso nome
  - Il PVC associato rimane Bound

```

```
---
