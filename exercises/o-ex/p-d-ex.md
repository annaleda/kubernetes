- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 
### Pod Design (24 esercizi)
---

## PD-1 — Pod Singolo (non gestito)

Creare un Pod chiamato `standalone-pod`

- Usa nginx
- Deve essere accessibile sulla porta 80
- Usa il comportamento standard di riavvio

- Vincoli  
  Non deve essere gestito da altre risorse

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

- Usa nginx versione 1.22
- Deve partire con 3 istanze

- Task successivo
  - Portare le istanze a 5

- Validazione
  - Sono presenti 5 Pod
  - Il sistema di replica riflette il nuovo numero

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

- 4 istanze
- nginx versione 1.21

Aggiornare a nginx:1.23 e poi tornare alla versione precedente

- Vincoli
  - L’aggiornamento deve avvenire gradualmente

- Validazione
  - Verificare storico aggiornamenti
  - Verificare versione dopo rollback

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

- Usa busybox
- Esegue:
  `sh -c "echo Job executed && sleep 5"`

- Deve completarsi una sola volta
- Ritentare massimo 2 volte

- Validazione
  - Stato completato
  - Il Pod termina correttamente

---

<details>
<summary>Soluzione</summary>
  
```
k create job batch-job --image=busybox --dry-run=client -o yaml > job.yaml -- sh -c 'echo Job execcuted && sleep 5'
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
k logs job/batch-job
```
</details>

---

## PD-5 — Parallel Job

Creare un Job chiamato `parallel-job`

- Usa busybox
- Esegue `sleep 10`

- Deve completarsi 4 volte
- Massimo 2 esecuzioni contemporanee

- Validazione
  - Non più di 2 Pod Running contemporaneamente

---

<details>
<summary>Soluzione</summary>
  
```
 k create job parallel-job --image=busybox --dry-run=client -o yaml > parallel-job.yaml -- sh -c 'sleep 10'

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

- Usa busybox
- Esegue `date`
- Deve partire ogni 2 minuti

- Configurazione
  - Conservare al massimo 3 esecuzioni riuscite
  - Conservare al massimo 1 fallita

- Validazione
  - Viene creato almeno un Job
  - Lo schedule è corretto
---
<details>
<summary>Soluzione</summary>


```
k create cronjob scheduled-task --image=busybox --schedule="*/2 * * * *" --dry-run=client -o yaml > cron-job.yaml -- sh -c date

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

- 2 istanze
- nginx
- porta 80

- Service richiesto
  - Nome: `db-headless`
  - Senza IP cluster
  - Deve collegarsi correttamente alle istanze

- Storage
  - Ogni istanza deve avere il proprio volume
  - Dimensione: 100Mi
  - Accesso da un solo nodo
  - Classe standard

- Obiettivo
  - Nomi prevedibili:
    - db-stateful-0
    - db-stateful-1

- Vincoli
  - Non creare manualmente i volumi

- Validazione
  - I volumi sono separati
  - Eliminando una replica mantiene lo stesso storage

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

## PD-8 — Deployment

Creare un Deployment chiamato `frontend-deployment`

- nginx versione 1.25
- 2 istanze
- Tutte le istanze devono avere etichetta `app=frontend`

- Obiettivo
  - Le istanze devono essere gestite automaticamente

- Validazione
  - Presenza del sistema di replica
  - Etichette corrette sui Pod

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

## PD-9 — Deployment con aggiornamento distruttivo

Creare un Deployment chiamato `recreate-deployment`

- nginx versione 1.24
- 3 istanze

- Configurazione
  - Durante aggiornamenti, eliminare prima tutte le istanze e poi ricrearle

- Obiettivo
  - Aggiornare successivamente a nginx 1.25

- Validazione
  - Il comportamento segue quanto richiesto

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

## PD-10 — Job con esecuzione sequenziale

Creare un Job chiamato `multi-completion-job`

- Usa busybox
- Esegue:
  `sh -c "echo hello from job && sleep 3"`

- Configurazione
  - Deve completarsi 3 volte
  - Una esecuzione alla volta
  - Ritentare massimo 1 volta

- Validazione
  - Numero corretto di completamenti

---

<details>
<summary>Soluzione</summary>

```yaml
 k create job multi-completion-job --image=busybox -o yaml --dry-run=client > multi-completion-job.yaml -- sh -c 'echo hello from job && sleep 3'

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

## PD-11 — CronJob senza esecuzioni parallele

Creare un CronJob chiamato `report-cronjob`

- Deve partire ogni minuto
- Usa busybox
- Esegue:
  `sh -c "date; echo report generated"`

- Configurazione
  - Non deve permettere esecuzioni contemporanee
  - Conservare 2 successi e 1 fallimento

- Validazione
  - Nessuna sovrapposizione tra esecuzioni

---

<details>
<summary>Soluzione</summary>

```yaml
 k create cronjob report-cronjob --image=busybox --schedule="* * * * *" -o yaml --dry-run=client > report-cronjob.yaml -- sh -c 'date; echo report generated'

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

- Usa busybox
- Esegue continuamente:
  `echo agent running` ogni 30 secondi

- Obiettivo
  - Un’istanza per ogni nodo

- Validazione
  - Numero Pod uguale al numero di nodi

---

<details>
<summary>Soluzione</summary>

```yaml
 k create deploy node-agent --image=busybox -oyaml --dry-run=client > node-agent.yaml -- sh -c "while true; do echo agent running; sleep 30; done"

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

## PD-13 — Deployment con pausa e ripresa aggiornamenti

Creare un Deployment chiamato `paused-deployment`

- nginx versione 1.24
- 2 istanze

- Task
  - Mettere in pausa gli aggiornamenti
  - Aggiornare a nginx 1.25
  - Riprendere aggiornamento

- Validazione
  - Versione finale corretta

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

## PD-14 — Deployment con controllo rollout

Creare un Deployment chiamato `controlled-rollout`

- nginx versione 1.25
- 4 istanze

- Configurazione
  - Durante aggiornamenti:
    - massimo 1 istanza non disponibile
    - massimo 2 istanze extra temporanee

- Validazione
  - Comportamento corretto durante rollout

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

- nginx
- 3 istanze
- Etichetta `app=web-rs`

- Obiettivo
  - Creazione diretta senza Deployment

- Validazione
  - Pod con etichetta corretta

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

- nginx
- 3 istanze

- Service richiesto
  - Nome: `cache-headless`
  - Senza IP cluster

- Validazione
  - Nomi:
    - cache-stateful-0
    - cache-stateful-1
    - cache-stateful-2

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

## PD-17 — Job con limite temporale

Creare un Job chiamato `deadline-job`

- busybox
- Esegue `sleep 30`

- Configurazione
  - Deve essere terminato dopo 10 secondi

- Validazione
  - Interruzione forzata del Job

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

## PD-18 — Job completamente parallelo

Creare un Job chiamato `fast-parallel-job`

- busybox
- Esegue:
  `sh -c "echo run && sleep 5"`

- Configurazione
  - 3 completamenti
  - Tutti contemporaneamente

- Validazione
  - Fino a 3 Pod attivi insieme

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

- Ogni minuto
- busybox
- Esegue `date`

- Configurazione
  - Deve essere sospeso

- Validazione
  - Nessun Job creato

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

## PD-20 — DaemonSet con etichette

Creare un DaemonSet chiamato `log-agent`

- busybox
- Esegue continuamente:
  `echo logging` ogni 20 secondi
- Etichetta `app=log-agent`

- Validazione
  - Pod con etichette corrette

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

## PD-21 — Debug selector incoerente

Creare un Deployment chiamato `broken-selector`

- Problema
  - Le etichette usate per selezionare e quelle nei Pod non coincidono

- Obiettivo
  - Correggere il problema

- Validazione
  - I Pod vengono creati correttamente

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

## PD-22 — Job con retry

Creare un Job chiamato `retry-job`

- busybox
- Esegue un comando che fallisce sempre

- Configurazione
  - Ritentare fino a 3 volte

- Validazione
  - Tentativi multipli visibili

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

## PD-23 — StatefulSet con storage (1 replica)

Creare uno StatefulSet chiamato `single-db`

- 1 istanza
- nginx

- Service richiesto
  - Nome: `single-db-headless`

- Storage
  - Volume automatico
  - 50Mi
  - Accesso da un solo nodo

- Validazione
  - Nome `single-db-0`
  - Volume creato automaticamente

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

## PD-24 — CronJob con deadline

Creare un CronJob chiamato `deadline-cronjob`

- Ogni minuto
- busybox
- Esegue `date`

- Configurazione
  - Deve partire solo se non supera 30 secondi di ritardo

- Validazione
  - Comportamento corretto rispetto al tempo

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
