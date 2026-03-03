
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

---
<details>
<summary>Soluzione</summary>
  
```
k run standalone-pod --image=nginx --port=80 --dry-run=client -o yaml > standalone-pod.yaml
k apply -f standalone-pod.yaml
```
</details>

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

---
<details>
<summary>Soluzione</summary>

```
k create deploy web-deployment --image=nginx:1.22 --replicas=3

k scale deploy web-deployment --replicas=5

k get po -o wide
k get rs -o wide
```
</details>

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

---
<details>
<summary>Soluzione</summary>
  
```
k create deploy api-deployment --image=nginx:1.21 --replicas=4 --dry-run=client -o yaml > api-deploy.yaml
k apply -f api-deploy.yaml
k set image deploy api-deployment nginx=nginx:1.23

k rollout undo deploy api-deployment
k rollout history deploy api-deployment
```
</details>

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

---

<details>
<summary>Soluzione</summary>
  
```
k create job batch-job --image=busybox --dry-run=client -o yaml > job.yaml
vi job.yaml

apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: batch-job
spec:
  completions: 1
  backoffLimit: 2
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: busybox
        name: batch-job
        resources: {}
        command: ["sh", "-c", "echo Job executed && sleep 5"]
      restartPolicy: Never
status: {}

k apply -f job.yaml
```
</details>

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

---

<details>
<summary>Soluzione</summary>
  
```
 k create job parallel-job --image=busybox --dry-run=client -o yaml > parallel-job.yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-job

spec:
  completions: 4
  parallelism: 2
  backoffLimit: 2

  template:
    spec:
      containers:
      - name: parallel-container
        image: busybox
        command:
        - sh
        - -c
        - sleep 10

      restartPolicy: Never
```
</details>

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
---
<details>
<summary>Soluzione</summary>


```
k create cronjob scheduled-task --image=busybox --schedule="*/2 * * * *" --dry-run=client -o yaml > cron-job.yaml

apiVersion: batch/v1
kind: CronJob
metadata:
  name: scheduled-task

spec:
  schedule: "*/2 * * * *"

  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1

  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cron-container
            image: busybox
            command:
            - sh
            - -c
            - date

          restartPolicy: OnFailure
```

| Espressione   | Significato                  |
| ------------- | ---------------------------- |
| `*/2 * * * *` | Ogni 2 minuti                |
| `* * * * *`   | Ogni minuto                  |
| `0 */2 * * *` | Ogni 2 ore, allo zero minuto |

</details>

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

---

<details>
<summary>Soluzione</summary>
  
```
k create service clusterip db-headless --tcp=80:80 --dry-run=client -o yaml > db-headless.yaml
k create statefulset db-stateful --image=nginx --replicas=2 --dry-run=client -o yaml > db-stateful.yaml

vi db-headless.yaml

apiVersion: v1
kind: Service
metadata:
  name: db-headless
spec:
  clusterIP: None # modificato
  selector:
    app: db-stateful
  ports:
    - port: 80
      targetPort: 80



vi db-stateful.yaml



apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db-stateful
spec:
  serviceName: db-headless # aggiunto
  replicas: 2
  selector:
    matchLabels:
      app: db-stateful

  template:
    metadata:
      labels:
        app: db-stateful

    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80

          volumeMounts:   # aggiunto
            - name: data
              mountPath: /usr/share/nginx/html

  volumeClaimTemplates:   # aggiunto
    - metadata:
        name: data
      spec:
        accessModes:
          - ReadWriteOnce

        storageClassName: standard

        resources:
          requests:
            storage: 100Mi
```
</details>

---
