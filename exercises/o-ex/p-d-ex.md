
### Pod Design (24 esercizi)
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

## PD-8 — Da Pod a Deployment

Creare un Deployment chiamato `frontend-deployment`

- Specifiche
  - Image: nginx:1.25
  - Replicas: 2
  - Label: `app=frontend`

- Obiettivo
  - Verificare che i Pod siano gestiti da un ReplicaSet
  - I Pod devono avere label coerenti

- Vincoli
  - Non creare un Pod standalone
  - Usare Deployment

- Validazione
  - `kubectl get deploy`
  - `kubectl get rs`
  - `kubectl get po --show-labels`

---

<details>
<summary>Soluzione</summary>

```sh
k create deployment frontend-deployment --image=nginx:1.25 --replicas=2 --dry-run=client -o yaml > frontend-deployment.yaml
```

Modifica il file per aggiungere la label `app=frontend` se necessario, poi:

```sh
k apply -f frontend-deployment.yaml
k get deploy
k get rs
k get po --show-labels
```

Esempio YAML completo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
```

</details>

---

## PD-9 — Deployment con Strategy Recreate

Creare un Deployment chiamato `recreate-deployment`

- Specifiche
  - Image: nginx:1.24
  - Replicas: 3

- Configurazione
  - Usare strategy `Recreate`

- Obiettivo
  - Aggiornare successivamente l’immagine a `nginx:1.25`

- Validazione
  - `kubectl describe deploy recreate-deployment`
  - Verificare che la strategy sia `Recreate`

---

<details>
<summary>Soluzione</summary>

```sh
k create deployment recreate-deployment --image=nginx:1.24 --replicas=3 --dry-run=client -o yaml > recreate-deployment.yaml
```

Modifica il file e aggiungi:

```yaml
strategy:
  type: Recreate
```

YAML completo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recreate-deployment
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: recreate-deployment
  template:
    metadata:
      labels:
        app: recreate-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
```

Applica e aggiorna:

```sh
k apply -f recreate-deployment.yaml
k set image deployment/recreate-deployment nginx=nginx:1.25
k describe deploy recreate-deployment
```

</details>

---

## PD-10 — Job con Più Completions

Creare un Job chiamato `multi-completion-job`

- Specifiche
  - Image: busybox
  - Command: `sh -c "echo hello from job && sleep 3"`
  - completions: 3
  - parallelism: 1
  - backoffLimit: 1

- Obiettivo
  - Eseguire 3 completamenti totali, uno alla volta

- Validazione
  - `kubectl get jobs`
  - `kubectl describe job multi-completion-job`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: multi-completion-job
spec:
  completions: 3
  parallelism: 1
  backoffLimit: 1
  template:
    spec:
      containers:
      - name: worker
        image: busybox
        command:
        - sh
        - -c
        - echo hello from job && sleep 3
      restartPolicy: Never
```

```sh
k apply -f multi-completion-job.yaml
k get jobs
k describe job multi-completion-job
```

</details>

---

## PD-11 — CronJob con Concurrency Policy

Creare un CronJob chiamato `report-cronjob`

- Specifiche
  - Schedule: ogni minuto
  - Image: busybox
  - Command: `sh -c "date; echo report generated"`
  - concurrencyPolicy: Forbid

- Configurazione
  - successfulJobsHistoryLimit: 2
  - failedJobsHistoryLimit: 1

- Obiettivo
  - Evitare esecuzioni concorrenti

- Validazione
  - `kubectl get cronjob`
  - `kubectl describe cronjob report-cronjob`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: report-cronjob
spec:
  schedule: "* * * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 2
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: reporter
            image: busybox
            command:
            - sh
            - -c
            - date; echo report generated
          restartPolicy: OnFailure
```

```sh
k apply -f report-cronjob.yaml
k get cronjob
k describe cronjob report-cronjob
```

</details>

---

## PD-12 — DaemonSet Base

Creare un DaemonSet chiamato `node-agent`

- Specifiche
  - Image: busybox
  - Command: `sh -c "while true; do echo agent running; sleep 30; done"`

- Obiettivo
  - Eseguire un Pod su ogni nodo

- Vincoli
  - Non usare Deployment
  - Usare `DaemonSet`

- Validazione
  - `kubectl get ds`
  - `kubectl get po -o wide`
  - Numero Pod uguale al numero di nodi schedulabili

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-agent
spec:
  selector:
    matchLabels:
      app: node-agent
  template:
    metadata:
      labels:
        app: node-agent
    spec:
      containers:
      - name: agent
        image: busybox
        command:
        - sh
        - -c
        - while true; do echo agent running; sleep 30; done
```

```sh
k apply -f node-agent.yaml
k get ds
k get po -o wide
```

</details>

---

## PD-13 — Deployment con Pause e Resume Rollout

Creare un Deployment chiamato `paused-deployment`

- Specifiche iniziali
  - Image: nginx:1.24
  - Replicas: 2

- Task
  - Mettere in pausa il rollout
  - Aggiornare immagine a `nginx:1.25`
  - Riprendere il rollout

- Validazione
  - `kubectl rollout status deployment paused-deployment`
  - L’immagine finale è `nginx:1.25`

---

<details>
<summary>Soluzione</summary>

```sh
k create deployment paused-deployment --image=nginx:1.24 --replicas=2
k rollout pause deployment paused-deployment
k set image deployment/paused-deployment nginx=nginx:1.25
k rollout resume deployment paused-deployment
k rollout status deployment paused-deployment
```

</details>

---

## PD-14 — Deployment con Replica minime e maxSurge

Creare un Deployment chiamato `controlled-rollout`

- Specifiche
  - Image: nginx:1.25
  - Replicas: 4

- Configurazione
  - strategy: RollingUpdate
  - maxUnavailable: 1
  - maxSurge: 2

- Validazione
  - `kubectl describe deployment controlled-rollout`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: controlled-rollout
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 2
  selector:
    matchLabels:
      app: controlled-rollout
  template:
    metadata:
      labels:
        app: controlled-rollout
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
```

```sh
k apply -f controlled-rollout.yaml
k describe deployment controlled-rollout
```

</details>

---

## PD-15 — ReplicaSet Base

Creare un ReplicaSet chiamato `web-rs`

- Specifiche
  - Image: nginx
  - Replicas: 3
  - Label: `app=web-rs`

- Obiettivo
  - Creare direttamente un ReplicaSet senza Deployment

- Validazione
  - `kubectl get rs`
  - `kubectl get pods --show-labels`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-rs
  template:
    metadata:
      labels:
        app: web-rs
    spec:
      containers:
      - name: nginx
        image: nginx
```

```sh
k apply -f web-rs.yaml
k get rs
k get pods --show-labels
```

</details>

---

## PD-16 — StatefulSet con 3 repliche

Creare uno StatefulSet chiamato `cache-stateful`

- Specifiche
  - Image: nginx
  - Replicas: 3
  - ServiceName: `cache-headless`

- Service richiesto
  - Nome: `cache-headless`
  - ClusterIP: None

- Validazione
  - I Pod si chiamano:
    - `cache-stateful-0`
    - `cache-stateful-1`
    - `cache-stateful-2`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cache-headless
spec:
  clusterIP: None
  selector:
    app: cache-stateful
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: cache-stateful
spec:
  serviceName: cache-headless
  replicas: 3
  selector:
    matchLabels:
      app: cache-stateful
  template:
    metadata:
      labels:
        app: cache-stateful
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```sh
k apply -f cache-stateful.yaml
k get pods
```

</details>

---

## PD-17 — Job con activeDeadlineSeconds

Creare un Job chiamato `deadline-job`

- Specifiche
  - Image: busybox
  - Command: `sh -c "sleep 30"`
  - activeDeadlineSeconds: 10

- Obiettivo
  - Il Job deve essere terminato dopo 10 secondi

- Validazione
  - `kubectl describe job deadline-job`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: deadline-job
spec:
  activeDeadlineSeconds: 10
  template:
    spec:
      containers:
      - name: worker
        image: busybox
        command:
        - sh
        - -c
        - sleep 30
      restartPolicy: Never
```

```sh
k apply -f deadline-job.yaml
k describe job deadline-job
```

</details>

---

## PD-18 — Job con completions e parallelism uguali

Creare un Job chiamato `fast-parallel-job`

- Specifiche
  - Image: busybox
  - Command: `sh -c "echo run && sleep 5"`
  - completions: 3
  - parallelism: 3

- Obiettivo
  - Tutti i Pod partono insieme

- Validazione
  - `kubectl get jobs`
  - Fino a 3 Pod contemporaneamente

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: fast-parallel-job
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
      - name: runner
        image: busybox
        command:
        - sh
        - -c
        - echo run && sleep 5
      restartPolicy: Never
```

```sh
k apply -f fast-parallel-job.yaml
k get jobs
k get pods
```

</details>

---

## PD-19 — CronJob sospeso

Creare un CronJob chiamato `suspended-cronjob`

- Specifiche
  - Schedule: ogni minuto
  - Image: busybox
  - Command: `date`

- Configurazione
  - suspend: true

- Obiettivo
  - Il CronJob esiste ma non esegue Job

- Validazione
  - `kubectl get cronjob`
  - Nessun Job creato finché è sospeso

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: suspended-cronjob
spec:
  schedule: "* * * * *"
  suspend: true
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: sleeper
            image: busybox
            command:
            - sh
            - -c
            - date
          restartPolicy: OnFailure
```

```sh
k apply -f suspended-cronjob.yaml
k get cronjob
k get jobs
```

</details>

---

## PD-20 — DaemonSet con label selector

Creare un DaemonSet chiamato `log-agent`

- Specifiche
  - Image: busybox
  - Command: `sh -c "while true; do echo logging; sleep 20; done"`

- Configurazione
  - Label: `app=log-agent`

- Validazione
  - `kubectl get ds`
  - `kubectl get pods --show-labels`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      containers:
      - name: agent
        image: busybox
        command:
        - sh
        - -c
        - while true; do echo logging; sleep 20; done
```

```sh
k apply -f log-agent.yaml
k get ds
k get pods --show-labels
```

</details>

---

## PD-21 — Deployment con selector e template incoerenti (debug)

Creare un Deployment chiamato `broken-selector`

- Problema
  - selector: `app=broken`
  - template label: `app=wrong`

- Obiettivo
  - Identificare e correggere il problema

- Validazione
  - I Pod vengono creati correttamente dopo il fix

---

<details>
<summary>Soluzione</summary>

Manifest errato:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken-selector
spec:
  replicas: 2
  selector:
    matchLabels:
      app: broken
  template:
    metadata:
      labels:
        app: wrong
    spec:
      containers:
      - name: nginx
        image: nginx
```

Fix:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken-selector
spec:
  replicas: 2
  selector:
    matchLabels:
      app: broken
  template:
    metadata:
      labels:
        app: broken
    spec:
      containers:
      - name: nginx
        image: nginx
```

</details>

---

## PD-22 — Job che fallisce e ritenta

Creare un Job chiamato `retry-job`

- Specifiche
  - Image: busybox
  - Command: `sh -c "exit 1"`
  - backoffLimit: 3

- Obiettivo
  - Kubernetes deve ritentare il Job fino al limite

- Validazione
  - `kubectl describe job retry-job`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: retry-job
spec:
  backoffLimit: 3
  template:
    spec:
      containers:
      - name: failer
        image: busybox
        command:
        - sh
        - -c
        - exit 1
      restartPolicy: Never
```

```sh
k apply -f retry-job.yaml
k describe job retry-job
```

</details>

---

## PD-23 — StatefulSet con PVC automatici e 1 replica

Creare uno StatefulSet chiamato `single-db`

- Specifiche
  - Replicas: 1
  - Image: nginx
  - ServiceName: `single-db-headless`

- Storage
  - volumeClaimTemplates
  - Claim name: `data`
  - Storage: `50Mi`
  - AccessMode: `ReadWriteOnce`

- Validazione
  - Il Pod si chiama `single-db-0`
  - Il PVC viene creato automaticamente

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: single-db-headless
spec:
  clusterIP: None
  selector:
    app: single-db
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: single-db
spec:
  serviceName: single-db-headless
  replicas: 1
  selector:
    matchLabels:
      app: single-db
  template:
    metadata:
      labels:
        app: single-db
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 50Mi
```

```sh
k apply -f single-db.yaml
k get pods
k get pvc
```

</details>

---

## PD-24 — CronJob con deadline di partenza

Creare un CronJob chiamato `deadline-cronjob`

- Specifiche
  - Schedule: ogni minuto
  - Image: busybox
  - Command: `date`

- Configurazione
  - startingDeadlineSeconds: 30

- Obiettivo
  - Il Job deve partire solo se non “perde” la finestra di avvio

- Validazione
  - `kubectl describe cronjob deadline-cronjob`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: deadline-cronjob
spec:
  schedule: "* * * * *"
  startingDeadlineSeconds: 30
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: reporter
            image: busybox
            command:
            - sh
            - -c
            - date
          restartPolicy: OnFailure
```

```sh
k apply -f deadline-cronjob.yaml
k describe cronjob deadline-cronjob
```

</details>

---
---
