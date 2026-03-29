* [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)

---

### Deployment Strategies (30 esercizi)

---

## DS-1 — Update immagine senza downtime

Creare Deployment `web-app`

* nginx 1.21

* 3 istanze

* Task

  * Aggiornare a nginx 1.25 senza downtime

---

<details>
<summary>Soluzione</summary>

```
k create deploy web-app --image=nginx:1.21 --replicas=3
k set image deployment/web-app nginx=nginx:1.25
k rollout status deployment web-app
```

</details>

---

## DS-2 — Verifica rollout

Deployment: `web-app`

* Task

  * Verificare stato aggiornamento

---

<details>
<summary>Soluzione</summary>

```
k rollout status deployment web-app
```

</details>

---

## DS-3 — Storico rollout

Deployment: `web-app`

---

<details>
<summary>Soluzione</summary>

```
k rollout history deployment web-app
```

</details>

---

## DS-4 — Rollback

Deployment: `web-app`

---

<details>
<summary>Soluzione</summary>

```
k rollout undo deployment web-app
```

</details>

---

## DS-5 — Rollback revisione specifica

Deployment: `web-app`

---

<details>
<summary>Soluzione</summary>

```
k rollout undo deployment web-app --to-revision=1
```

</details>

---

## DS-6 — Strategia Recreate

Creare un Deployment chiamato `recreate-app`

* Usa nginx versione 1.24

* Deve avere 3 istanze

* Comportamento richiesto

  * Durante aggiornamenti, tutte le istanze devono essere terminate prima di crearne di nuove

* Validazione

  * Il comportamento dell’aggiornamento è distruttivo (no sovrapposizione tra versioni)

---

<details>
<summary>Soluzione</summary>

```sh
k create deploy recreate-app --image=nginx:1.24 --replicas=3 --dry-run=client -o yaml > rec.yaml
```

Modifica:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recreate-app
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: recreate-app
  template:
    metadata:
      labels:
        app: recreate-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
```

```sh
k apply -f rec.yaml
```

</details>

---

## DS-7 — Aggiornamento con strategia Recreate

Deployment esistente: `recreate-app`

* Task

  * Aggiornare l’applicazione a nginx versione 1.25

* Validazione

  * I Pod vengono terminati tutti prima della creazione dei nuovi

---

<details>
<summary>Soluzione</summary>

```sh
k set image deployment/recreate-app nginx=nginx:1.25
```

</details>

---

## DS-8 — Controllo istanze non disponibili

Creare un Deployment chiamato `controlled-app`

* Usa nginx

* Deve avere 4 istanze

* Comportamento richiesto

  * Durante aggiornamenti, al massimo 1 istanza può essere non disponibile

* Validazione

  * Il numero di Pod non disponibili non supera mai 1

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: controlled-app
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  selector:
    matchLabels:
      app: controlled-app
  template:
    metadata:
      labels:
        app: controlled-app
    spec:
      containers:
      - name: nginx
        image: nginx
```

```sh
k apply -f controlled.yaml
```

</details>

---

## DS-9 — Controllo istanze extra

Deployment esistente: `controlled-app`

* Task

  * Permettere la creazione temporanea di massimo 2 istanze aggiuntive durante l’aggiornamento

* Validazione

  * Il numero totale di Pod può temporaneamente superare quello desiderato

---

<details>
<summary>Soluzione</summary>

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 2
```

```sh
k apply -f controlled.yaml
```

</details>

---

## DS-10 — Pausa aggiornamento

Deployment: `controlled-app`

* Task

  * Mettere in pausa l’aggiornamento

* Validazione

  * L’aggiornamento viene sospeso

---

<details>
<summary>Soluzione</summary>

```sh
k rollout pause deployment controlled-app
```

</details>

---

## DS-11 — Ripresa aggiornamento

Deployment: `controlled-app`

* Task

  * Riprendere l’aggiornamento

---

<details>
<summary>Soluzione</summary>

```sh
k rollout resume deployment controlled-app
```

</details>

---

## DS-12 — Riavvio controllato

Deployment: `controlled-app`

* Task

  * Riavviare tutte le istanze senza modificare la configurazione

---

<details>
<summary>Soluzione</summary>

```sh
k rollout restart deployment controlled-app
```

</details>

---

## DS-13 — Modifica configurazione senza eliminare

Deployment: `web-app`

* Task

  * Modificare la configurazione senza eliminare la risorsa

* Vincolo

  * Non ricreare il Deployment

---

<details>
<summary>Soluzione</summary>

```sh
k get deploy web-app -o yaml > d.yaml
vi d.yaml
k apply -f d.yaml
```

</details>

---

## DS-14 — Aggiornamento immagine senza YAML

Deployment: `web-app`

* Task

  * Aggiornare l’immagine senza usare file YAML

---

<details>
<summary>Soluzione</summary>

```sh
k set image deployment/web-app nginx=nginx:1.26
```

</details>

---

## DS-15 — Scaling dinamico

Deployment: `web-app`

* Task

  * Portare il numero di istanze a 6

---

<details>
<summary>Soluzione</summary>

```sh
k scale deployment web-app --replicas=6
```

</details>

---

## DS-16 — Diagnosi rollout

Deployment: `web-app`

* Problema

  * L’aggiornamento non termina

* Task

  * Analizzare lo stato

---

<details>
<summary>Soluzione</summary>

```sh
k rollout status deployment web-app
k describe deployment web-app
```

</details>

---

## DS-17 — Verifica versione in esecuzione

Deployment: `web-app`

* Task

  * Verificare quale versione è in esecuzione nei Pod

---

<details>
<summary>Soluzione</summary>

```sh
k get pods -o wide
```

</details>

---

## DS-18 — Riavvio senza modifiche

Deployment: `web-app`

* Task

  * Riavviare i Pod senza cambiare configurazione

---

<details>
<summary>Soluzione</summary>

```sh
k rollout restart deployment web-app
```

</details>

---

## DS-19 — Aggiornamento rapido

Deployment: `web-app`

* Task

  * Cambiare immagine con un singolo comando

---

<details>
<summary>Soluzione</summary>

```sh
k set image deployment/web-app nginx=nginx:latest
```

</details>

---

## DS-20 — Verifica ReplicaSet

Deployment: `web-app`

* Task

  * Verificare le versioni dei ReplicaSet

---

<details>
<summary>Soluzione</summary>

```sh
k get rs
```

</details>

---

## DS-21 — Aggiornamento errato

Deployment: `web-app`

* Task

  * Impostare una versione non valida e osservare il comportamento

---

<details>
<summary>Soluzione</summary>

```sh
k set image deployment/web-app nginx=nginx:wrong
k rollout status deployment web-app
```

</details>

---

## DS-22 — Ripristino versione funzionante

Deployment: `web-app`

* Task

  * Ripristinare una versione funzionante

---

<details>
<summary>Soluzione</summary>

```sh
k rollout undo deployment web-app
```

</details>

---

## DS-23 — Analisi eventi

Deployment: `web-app`

* Task

  * Analizzare eventi e problemi

---

<details>
<summary>Soluzione</summary>

```sh
k describe deployment web-app
```

</details>

---

## DS-24 — Ciclo completo aggiornamento

Creare Deployment `full-cycle-app`

* Task

  * Creare
  * Aggiornare
  * Verificare
  * Tornare indietro

---

<details>
<summary>Soluzione</summary>

```sh
k create deploy full-cycle-app --image=nginx:1.21
k set image deployment/full-cycle-app nginx=nginx:1.25
k rollout status deployment full-cycle-app
k rollout undo deployment full-cycle-app
```

</details>

---


---

### STRATEGIES

---

## DS-25 — Blue/Green base

Creare due Deployment:

* `app-blue`

  * nginx versione 1.21
  * 3 istanze
* `app-green`

  * nginx versione 1.25
  * 3 istanze

Creare un Service chiamato `app-service`

* Il traffico deve andare inizialmente solo verso la versione `blue`

* Obiettivo

  * Preparare due versioni della stessa applicazione
  * Esporre inizialmente solo la versione attuale

* Validazione

  * Il Service invia traffico solo ai Pod della versione blue

---

<details>
<summary>Soluzione</summary>

```sh
k create deploy app-blue --image=nginx:1.21 --replicas=3
k create deploy app-green --image=nginx:1.25 --replicas=3
k expose deploy app-blue --name app-service --port=80
```

</details>

---

## DS-26 — Switch traffico verso Green

Deployment esistenti:

* `app-blue`
* `app-green`

Service esistente:

* `app-service`

* Task

  * Spostare il traffico dalla versione `blue` alla versione `green`
  * Non eliminare nessuno dei due Deployment

* Obiettivo

  * Rendere attiva la nuova versione cambiando solo il punto di instradamento del traffico

* Validazione

  * Il Service invia traffico solo ai Pod della versione green

---

<details>
<summary>Soluzione</summary>

```sh
k edit svc app-service
```

</details>

---

## DS-27 — Rollback Blue/Green

Scenario:

* La versione `green` è attiva tramite `app-service`

* Task

  * Tornare immediatamente alla versione `blue`

* Obiettivo

  * Ripristinare la versione precedente senza ricreare le applicazioni

* Validazione

  * Il Service invia di nuovo traffico ai Pod della versione blue

---

<details>
<summary>Soluzione</summary>

```sh
k edit svc app-service
```

</details>

---

## DS-28 — Canary base

Creare due Deployment:

* `app-stable`

  * nginx versione 1.21
  * 4 istanze
* `app-canary`

  * nginx versione 1.25
  * 1 istanza

Creare un Service chiamato `app-service`

* Il traffico deve raggiungere entrambe le versioni

* Obiettivo

  * Esporre una nuova versione a una piccola parte del traffico
  * Mantenere la maggior parte del traffico sulla versione stabile

* Validazione

  * Il Service raggiunge sia la versione stabile sia la versione canary
  * La distribuzione del traffico favorisce la versione stabile

---

<details>
<summary>Soluzione</summary>

```sh
k create deploy app-stable --image=nginx:1.21 --replicas=4
k create deploy app-canary --image=nginx:1.25 --replicas=1
k expose deploy app-stable --name app-service --port=80
```

</details>

---

## DS-29 — Incremento traffico Canary

Deployment esistenti:

* `app-stable`

* `app-canary`

* Task

  * Aumentare la quota di traffico verso la nuova versione
  * Ridurre proporzionalmente quella verso la versione stabile

* Obiettivo

  * Estendere gradualmente la diffusione della nuova versione senza passare subito al 100%

* Validazione

  * La versione canary riceve più traffico rispetto a prima
  * La versione stabile resta comunque ancora attiva

---

<details>
<summary>Soluzione</summary>

```sh
k scale deployment app-canary --replicas=3
k scale deployment app-stable --replicas=2
```

</details>

---

## DS-30 — Canary verso rilascio completo

Scenario:

* La versione canary è stata verificata con successo

* Task

  * Portare tutta l’applicazione sulla nuova versione
  * Rimuovere la vecchia versione

* Obiettivo

  * Completare il passaggio dalla distribuzione graduale al rilascio totale

* Validazione

  * Tutto il traffico raggiunge solo la nuova versione
  * La versione stabile non è più presente

---

<details>
<summary>Soluzione</summary>

```sh
k scale deployment app-canary --replicas=5
k delete deployment app-stable
```

</details>

---

