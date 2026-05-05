- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 
### CronJob — Scheduling, JobTemplate e Pattern CKAD (20 esercizi)
---

## CJ-1 — CronJob ogni minuto

Creare un CronJob chiamato `every-minute-cron`

- Usa busybox
- Esegue:
  `date`
- Deve partire ogni minuto

- Validazione
  - Il CronJob è presente
  - Lo schedule è corretto
  - Viene creato almeno un Job

---
<details>
<summary>Soluzione</summary>
  
```
k create cronjob every-minute-cron --image=busybox --schedule="* * * * *" --dry-run=client -o yaml > every-minute-cron.yaml -- date
k apply -f every-minute-cron.yaml

k get cronjob
k get jobs
k describe cronjob every-minute-cron
```

YAML completo:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: every-minute-cron
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: every-minute-cron
            image: busybox
            command:
            - date
          restartPolicy: OnFailure
```
</details>

---

## CJ-2 — CronJob ogni 5 minuti

Creare un CronJob chiamato `five-minutes-cron`

- Usa busybox
- Esegue:
  `sh -c "echo run every 5 minutes; date"`
- Deve partire ogni 5 minuti

- Validazione
  - Schedule uguale a `*/5 * * * *`

---
<details>
<summary>Soluzione</summary>

```
k create cronjob five-minutes-cron --image=busybox --schedule="*/5 * * * *" --dry-run=client -o yaml > five-minutes-cron.yaml -- sh -c 'echo run every 5 minutes; date'
k apply -f five-minutes-cron.yaml

k get cronjob five-minutes-cron
k describe cronjob five-minutes-cron
```

YAML completo:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: five-minutes-cron
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: five-minutes-cron
            image: busybox
            command:
            - sh
            - -c
            - echo run every 5 minutes; date
          restartPolicy: OnFailure
```
</details>

---

## CJ-3 — CronJob ogni 15 minuti

Creare un CronJob chiamato `quarter-hour-cron`

- Usa busybox
- Esegue:
  `echo quarter`
- Deve partire ogni 15 minuti

- Validazione
  - Lo schedule usa lo step sui minuti

---
<details>
<summary>Soluzione</summary>

```
k create cronjob quarter-hour-cron --image=busybox --schedule="*/15 * * * *" --dry-run=client -o yaml > quarter-hour-cron.yaml -- sh -c 'echo quarter'
k apply -f quarter-hour-cron.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: quarter-hour-cron
spec:
  schedule: "*/15 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: quarter-hour-cron
            image: busybox
            command:
            - sh
            - -c
            - echo quarter
          restartPolicy: OnFailure
```
</details>

---

## CJ-4 — CronJob a un orario preciso ogni giorno

Creare un CronJob chiamato `daily-backup`

- Usa busybox
- Esegue:
  `sh -c "echo daily backup; date"`
- Deve partire ogni giorno alle 02:30

- Validazione
  - Lo schedule rappresenta minuto 30, ora 2

---
<details>
<summary>Soluzione</summary>

```
k create cronjob daily-backup --image=busybox --schedule="30 2 * * *" --dry-run=client -o yaml > daily-backup.yaml -- sh -c 'echo daily backup; date'
k apply -f daily-backup.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "30 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: daily-backup
            image: busybox
            command:
            - sh
            - -c
            - echo daily backup; date
          restartPolicy: OnFailure
```

| Campo | Valore | Significato |
| ----- | ------ | ----------- |
| minuto | `30` | al minuto 30 |
| ora | `2` | alle 02 |
| giorno mese | `*` | ogni giorno |
| mese | `*` | ogni mese |
| giorno settimana | `*` | qualsiasi giorno |

</details>

---

## CJ-5 — CronJob ogni ora

Creare un CronJob chiamato `hourly-task`

- Usa busybox
- Esegue:
  `date`
- Deve partire ogni ora esatta

- Validazione
  - Parte al minuto zero di ogni ora

---
<details>
<summary>Soluzione</summary>

```
k create cronjob hourly-task --image=busybox --schedule="0 * * * *" --dry-run=client -o yaml > hourly-task.yaml -- date
k apply -f hourly-task.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hourly-task
spec:
  schedule: "0 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hourly-task
            image: busybox
            command:
            - date
          restartPolicy: OnFailure
```
</details>

---

## CJ-6 — CronJob ogni 2 ore

Creare un CronJob chiamato `two-hours-task`

- Usa busybox
- Esegue:
  `echo every two hours`
- Deve partire ogni 2 ore, al minuto zero

- Validazione
  - Lo schedule usa `*/2` nel campo ore

---
<details>
<summary>Soluzione</summary>

```
k create cronjob two-hours-task --image=busybox --schedule="0 */2 * * *" --dry-run=client -o yaml > two-hours-task.yaml -- sh -c 'echo every two hours'
k apply -f two-hours-task.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: two-hours-task
spec:
  schedule: "0 */2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: two-hours-task
            image: busybox
            command:
            - sh
            - -c
            - echo every two hours
          restartPolicy: OnFailure
```
</details>

---

## CJ-7 — CronJob solo nei giorni feriali

Creare un CronJob chiamato `weekday-report`

- Usa busybox
- Esegue:
  `echo weekday report`
- Deve partire dal lunedì al venerdì alle 09:00

- Validazione
  - Lo schedule limita il giorno della settimana

---
<details>
<summary>Soluzione</summary>

```
k create cronjob weekday-report --image=busybox --schedule="0 9 * * 1-5" --dry-run=client -o yaml > weekday-report.yaml -- sh -c 'echo weekday report'
k apply -f weekday-report.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: weekday-report
spec:
  schedule: "0 9 * * 1-5"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: weekday-report
            image: busybox
            command:
            - sh
            - -c
            - echo weekday report
          restartPolicy: OnFailure
```

| Espressione | Significato |
| ----------- | ----------- |
| `1-5` | da lunedì a venerdì |
| `0` oppure `7` | domenica |
| `1` | lunedì |
| `6` | sabato |

</details>

---

## CJ-8 — CronJob nel weekend

Creare un CronJob chiamato `weekend-cleanup`

- Usa busybox
- Esegue:
  `echo weekend cleanup`
- Deve partire sabato e domenica alle 10:00

- Validazione
  - Lo schedule contiene più valori nel giorno settimana

---
<details>
<summary>Soluzione</summary>

```
k create cronjob weekend-cleanup --image=busybox --schedule="0 10 * * 6,0" --dry-run=client -o yaml > weekend-cleanup.yaml -- sh -c 'echo weekend cleanup'
k apply -f weekend-cleanup.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: weekend-cleanup
spec:
  schedule: "0 10 * * 6,0"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: weekend-cleanup
            image: busybox
            command:
            - sh
            - -c
            - echo weekend cleanup
          restartPolicy: OnFailure
```
</details>

---

## CJ-9 — CronJob il primo giorno del mese

Creare un CronJob chiamato `monthly-report`

- Usa busybox
- Esegue:
  `echo monthly report`
- Deve partire il primo giorno di ogni mese alle 00:00

- Validazione
  - Il campo giorno del mese vale `1`

---
<details>
<summary>Soluzione</summary>

```
k create cronjob monthly-report --image=busybox --schedule="0 0 1 * *" --dry-run=client -o yaml > monthly-report.yaml -- sh -c 'echo monthly report'
k apply -f monthly-report.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: monthly-report
spec:
  schedule: "0 0 1 * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: monthly-report
            image: busybox
            command:
            - sh
            - -c
            - echo monthly report
          restartPolicy: OnFailure
```
</details>

---

## CJ-10 — CronJob il quindicesimo giorno del mese

Creare un CronJob chiamato `midmonth-task`

- Usa busybox
- Esegue:
  `date`
- Deve partire il giorno 15 di ogni mese alle 12:00

- Validazione
  - Giorno del mese: 15
  - Ora: 12
  - Minuto: 0

---
<details>
<summary>Soluzione</summary>

```
k create cronjob midmonth-task --image=busybox --schedule="0 12 15 * *" --dry-run=client -o yaml > midmonth-task.yaml -- date
k apply -f midmonth-task.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: midmonth-task
spec:
  schedule: "0 12 15 * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: midmonth-task
            image: busybox
            command:
            - date
          restartPolicy: OnFailure
```
</details>

---

## CJ-11 — CronJob ogni lunedì

Creare un CronJob chiamato `weekly-monday`

- Usa busybox
- Esegue:
  `echo monday task`
- Deve partire ogni lunedì alle 08:00

- Validazione
  - Giorno settimana uguale a `1`

---
<details>
<summary>Soluzione</summary>

```
k create cronjob weekly-monday --image=busybox --schedule="0 8 * * 1" --dry-run=client -o yaml > weekly-monday.yaml -- sh -c 'echo monday task'
k apply -f weekly-monday.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: weekly-monday
spec:
  schedule: "0 8 * * 1"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: weekly-monday
            image: busybox
            command:
            - sh
            - -c
            - echo monday task
          restartPolicy: OnFailure
```
</details>

---

## CJ-12 — CronJob più volte al giorno

Creare un CronJob chiamato `twice-a-day`

- Usa busybox
- Esegue:
  `echo twice a day`
- Deve partire ogni giorno alle 08:00 e alle 18:00

- Validazione
  - Il campo ora contiene due valori separati da virgola

---
<details>
<summary>Soluzione</summary>

```
k create cronjob twice-a-day --image=busybox --schedule="0 8,18 * * *" --dry-run=client -o yaml > twice-a-day.yaml -- sh -c 'echo twice a day'
k apply -f twice-a-day.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: twice-a-day
spec:
  schedule: "0 8,18 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: twice-a-day
            image: busybox
            command:
            - sh
            - -c
            - echo twice a day
          restartPolicy: OnFailure
```
</details>

---

## CJ-13 — CronJob con timezone

Creare un CronJob chiamato `rome-time-cron`

- Usa busybox
- Esegue:
  `date`
- Deve partire ogni giorno alle 09:00 in timezone Europe/Rome

- Validazione
  - Presenza di `timeZone: Europe/Rome`

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: rome-time-cron
spec:
  schedule: "0 9 * * *"
  timeZone: "Europe/Rome"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: rome-time-cron
            image: busybox
            command:
            - date
          restartPolicy: OnFailure
```

```
k apply -f rome-time-cron.yaml
k describe cronjob rome-time-cron
```
</details>

---

## CJ-14 — CronJob senza esecuzioni parallele

Creare un CronJob chiamato `no-overlap-cron`

- Usa busybox
- Esegue:
  `sh -c "date; sleep 90"`
- Parte ogni minuto
- Non deve permettere esecuzioni contemporanee

- Validazione
  - `concurrencyPolicy` impostata a `Forbid`

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: no-overlap-cron
spec:
  schedule: "* * * * *"
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: no-overlap-cron
            image: busybox
            command:
            - sh
            - -c
            - date; sleep 90
          restartPolicy: OnFailure
```

```
k apply -f no-overlap-cron.yaml
k describe cronjob no-overlap-cron
```
</details>

---

## CJ-15 — CronJob che sostituisce il Job precedente

Creare un CronJob chiamato `replace-running-cron`

- Usa busybox
- Esegue:
  `sh -c "date; sleep 120"`
- Parte ogni minuto
- Se il Job precedente è ancora attivo, deve essere sostituito

- Validazione
  - `concurrencyPolicy` impostata a `Replace`

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: replace-running-cron
spec:
  schedule: "* * * * *"
  concurrencyPolicy: Replace
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: replace-running-cron
            image: busybox
            command:
            - sh
            - -c
            - date; sleep 120
          restartPolicy: OnFailure
```

```
k apply -f replace-running-cron.yaml
k describe cronjob replace-running-cron
```

| Valore | Effetto |
| ------ | ------- |
| `Allow` | permette Job paralleli |
| `Forbid` | salta la nuova esecuzione se quella precedente è ancora attiva |
| `Replace` | elimina il Job attivo e crea quello nuovo |

</details>

---

## CJ-16 — CronJob sospeso

Creare un CronJob chiamato `suspended-task`

- Usa busybox
- Esegue:
  `date`
- Schedule ogni minuto
- Deve essere creato ma non deve avviare Job

- Validazione
  - `suspend: true`
  - Nessun Job creato

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: suspended-task
spec:
  schedule: "* * * * *"
  suspend: true
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: suspended-task
            image: busybox
            command:
            - date
          restartPolicy: OnFailure
```

```
k apply -f suspended-task.yaml
k get cronjob
k get jobs
```
</details>

---

## CJ-17 — CronJob con limite di storico

Creare un CronJob chiamato `history-limit-cron`

- Usa busybox
- Esegue:
  `echo history`
- Parte ogni minuto
- Conserva massimo:
  - 2 Job riusciti
  - 1 Job fallito

- Validazione
  - Campi history configurati correttamente

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: history-limit-cron
spec:
  schedule: "* * * * *"
  successfulJobsHistoryLimit: 2
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: history-limit-cron
            image: busybox
            command:
            - sh
            - -c
            - echo history
          restartPolicy: OnFailure
```

```
k apply -f history-limit-cron.yaml
k describe cronjob history-limit-cron
```
</details>

---

## CJ-18 — CronJob con startingDeadlineSeconds

Creare un CronJob chiamato `deadline-schedule-cron`

- Usa busybox
- Esegue:
  `date`
- Parte ogni minuto
- Se Kubernetes è in ritardo di più di 30 secondi, non deve avviare il Job perso

- Validazione
  - `startingDeadlineSeconds: 30`

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: deadline-schedule-cron
spec:
  schedule: "* * * * *"
  startingDeadlineSeconds: 30
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: deadline-schedule-cron
            image: busybox
            command:
            - date
          restartPolicy: OnFailure
```

```
k apply -f deadline-schedule-cron.yaml
k describe cronjob deadline-schedule-cron
```
</details>

---

## CJ-19 — CronJob con Job che fallisce e retry limitato

Creare un CronJob chiamato `failing-cron`

- Usa busybox
- Esegue:
  `exit 1`
- Parte ogni 2 minuti
- Ogni Job deve ritentare massimo 2 volte

- Validazione
  - `backoffLimit: 2` dentro `jobTemplate.spec`

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: failing-cron
spec:
  schedule: "*/2 * * * *"
  jobTemplate:
    spec:
      backoffLimit: 2
      template:
        spec:
          containers:
          - name: failing-cron
            image: busybox
            command:
            - sh
            - -c
            - exit 1
          restartPolicy: Never
```

```
k apply -f failing-cron.yaml
k describe cronjob failing-cron
k get jobs
```
</details>

---

## CJ-20 — CronJob con activeDeadlineSeconds sul Job

Creare un CronJob chiamato `job-timeout-cron`

- Usa busybox
- Parte ogni minuto
- Esegue:
  `sleep 120`
- Ogni Job deve essere terminato dopo 30 secondi

- Validazione
  - `activeDeadlineSeconds: 30` dentro `jobTemplate.spec`

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: job-timeout-cron
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      activeDeadlineSeconds: 30
      template:
        spec:
          containers:
          - name: job-timeout-cron
            image: busybox
            command:
            - sh
            - -c
            - sleep 120
          restartPolicy: Never
```

```
k apply -f job-timeout-cron.yaml
k describe cronjob job-timeout-cron
k get jobs
```
</details>

---

## Cheat Sheet — Campi CronJob

| Campo | Significato |
| ----- | ----------- |
| `schedule` | espressione cron a 5 campi |
| `timeZone` | timezone dello schedule |
| `suspend` | sospende nuove esecuzioni |
| `concurrencyPolicy` | gestione Job sovrapposti |
| `successfulJobsHistoryLimit` | quanti Job riusciti conservare |
| `failedJobsHistoryLimit` | quanti Job falliti conservare |
| `startingDeadlineSeconds` | massimo ritardo accettato per avviare un Job perso |
| `jobTemplate.spec.backoffLimit` | retry del Job generato |
| `jobTemplate.spec.activeDeadlineSeconds` | durata massima del Job generato |

---

## Cheat Sheet — Espressioni cron

Formato:

```text
MINUTO ORA GIORNO_DEL_MESE MESE GIORNO_DELLA_SETTIMANA
```

| Espressione | Significato |
| ----------- | ----------- |
| `* * * * *` | ogni minuto |
| `*/5 * * * *` | ogni 5 minuti |
| `*/15 * * * *` | ogni 15 minuti |
| `0 * * * *` | ogni ora al minuto zero |
| `0 */2 * * *` | ogni 2 ore |
| `30 2 * * *` | ogni giorno alle 02:30 |
| `0 9 * * 1-5` | lunedì-venerdì alle 09:00 |
| `0 10 * * 6,0` | sabato e domenica alle 10:00 |
| `0 0 1 * *` | primo giorno del mese a mezzanotte |
| `0 12 15 * *` | giorno 15 del mese alle 12:00 |
| `0 8 * * 1` | ogni lunedì alle 08:00 |
| `0 8,18 * * *` | ogni giorno alle 08:00 e alle 18:00 |

---

## Regole importanti CKAD

- CronJob crea Job, non Pod direttamente
- Il comando del container sta dentro `jobTemplate`
- `restartPolicy` deve essere `OnFailure` oppure `Never`
- `concurrencyPolicy: Forbid` evita sovrapposizioni
- `suspend: true` blocca nuove esecuzioni
- `startingDeadlineSeconds` riguarda il ritardo dello schedule
- `activeDeadlineSeconds` riguarda la durata massima del Job
- `backoffLimit` va dentro `jobTemplate.spec`, non direttamente sotto `spec` del CronJob

---
