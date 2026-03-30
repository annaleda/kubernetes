- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 
###  Labels and Selectors (24 esercizi)

## LS-1 — Deployment con Label Specifiche

Creare un Deployment chiamato `frontend-app`

- Specifiche
  - Image: nginx
  - Replicas: 3

Labels:
  - app=frontend
  - tier=web

- Obiettivo
  - Verificare che i Pod abbiano le stesse label

- Validazione
  - kubectl get pods --show-labels
  - Label coerenti tra Deployment e Pod
---
<details>
<summary>Soluzione</summary>
  
```
k create deploy frontend-app --image=nginx --replicas=3 --dry-run=client -o yaml > deploy1.yaml

vi deploy1.yaml


apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: frontend-app
  name: frontend-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      tier: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: frontend
        tier: web
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}


k apply -f deploy1.yaml

kubectl get pods --show-labels

```
</details>

---

## LS-2 — Service con Selector Multipli

Creare Deployment `backend-app`

- Labels richieste
  - app=backend
  - tier=api

Creare Service `backend-service`

- Specifiche Service
  - Type: ClusterIP
  - Porta: 80

- Selector:
  - app=backend
  - tier=api

- Validazione
  - Service seleziona solo i Pod corretti
  - kubectl describe svc backend-service mostra Endpoints

---
<details>
<summary>Soluzione</summary>
  
```
k create deploy backend-app --image=nginx --replicas=3 --dry-run=client -o yaml > deploy2.yaml

vi deploy2.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: backend-app
  name: backend-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
      tier: api
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: backend
        tier: api
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}


k apply -f deploy2.yaml

kubectl get pods --show-labels

k expose deploy backend-app --name backend-service --port=80 --dry-run=client -o yaml > svc1.yaml

k apply -f svc1.yaml

k get svc

k get pod -l app=backend
```
</details>

---

## LS-3 — Modifica Label Live

Creare Pod `temp-app`
  - Image: nginx

- Task
  - Aggiungere label: environment=dev
  - Poi cambiarla in environment=prod

- Validazione
  - kubectl get pod temp-app --show-labels
---
<details>
<summary>Soluzione</summary>
  
```
 k run temp-app --image=nginx -l environment=dev -o yaml > pod-label.yaml

 k label po temp-app environment=prod --overwrite

k get pod temp-app --show-labels

NAME       READY   STATUS              RESTARTS   AGE   LABELS
temp-app   1/1     Running             0          83s   environment=prod

```
</details>

---

### LS-4 — Selector Non Matchante (Debug)

Creare Deployment `api-app`

- Label: app=api
- Creare Service `api-service`
- Selector: app=backend

- Obiettivo
  - Identificare perché il Service non ha endpoint
  - Correggere il selector

- Validazione
  - Service ha endpoint dopo la correzione

---
<details>
<summary>Soluzione</summary>
  
```
k create deploy api --image=nginx --dry-run=client -o yaml > api-app.yaml

vi api-app.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: api
  name: api-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: api
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: api
    spec:
      containers:
      - image: nginx
        name: nginx

k apply -f api-app.yaml

k expose deploy api-app --name api-service --port=80 --dry-run=client -o yaml > api-service.yaml

vi api-service.yaml

# Modificare selector con label app: backend

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: api
  name: api-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: backend

k get pods --show-labels
k get svc api-service -o wide
k describe svc api-service
k get endpoints api-service

# fix rimettere selector con label app: api


k get svc api-service
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
api-service   ClusterIP   10.96.207.250   <none>        80/TCP    63s

k describe svc api-service
```
</details>

---

### LS-5 — Node Selector

Label su un nodo: `disktype=ssd`

Creare Pod `ssd-app`

- Configurazione
  - nodeSelector:
    - disktype=ssd

- Validazione
  - Pod schedulato solo sul nodo corretto
---
<details>
<summary>Soluzione</summary>
  
```
kubectl label node ckad-exam-control-plane disktype=ssd

 k get nodes --show-labels
NAME                      STATUS   ROLES           AGE   VERSION    LABELS
ckad-exam-control-plane   Ready    control-plane   13d   v1.35.0    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,
                                                                    disktype=ssd,kubernetes.io/arch=amd64,
                                                                    kubernetes.io/hostname=ckad-exam-control-plane,
                                                                    kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=

k run ssd-app --image=nginx --dry-run=client -o yaml > ssd-app.yaml

vi ssd-app.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ssd-app
  name: ssd-app
spec:
  containers:
  - image: nginx
    name: ssd-app
    resources: {}
  nodeSelector:
    disktype: ssd
  dnsPolicy: ClusterFirst
  restartPolicy: Always

k get po ssd-app -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP       NODE                   
ssd-app   0/1     Running   0          24s   <none>   ckad-exam-control-plane

```
</details>

---

### LS-6 — Set-Based Selectors

Creare Deployment `multi-env-app`

- Labels possibili:
  - environment=dev
  - environment=staging
  - environment=prod

- Obiettivo
  - Recuperare solo Pod con:
    - environment in (dev, staging)

- Validazione
  - Usare selector set-based via CLI
---
<details>
<summary>Soluzione</summary>
  
```
k create deploy multi-env-app --replicas=2 --image=nginx --dry-run=client -o yaml > multi-env-app.yaml
# modifica labels
# Modificare il file per aggiungere label di default:

template:
  metadata:
    labels:
      app: multi-env-app
      environment: dev
k apply -f multi-env-app.yaml

# Modifica manuale delle label dei Pod



k get pods

# Prendi i nomi dei Pod e modifica le label:

k label pod <pod1> environment=dev --overwrite
k label pod <pod2> environment=staging --overwrite
k label pod <pod3> environment=prod --overwrite

kubectl get pods -l 'environment in (dev,staging)'
```
</details>

---

Cheatsheet Label

```

k label node l-node key=value
k label deploy...
k label svc...
k label cm...
k label crd-ex...
k label po...

# modifica
k label po l-po key=value --overwrite
# delete
k label po l-po key-

```

## LS-7 — Label mismatch tra ReplicaSet e Pod (debug)

- Deployment: `mismatch-app`

- Configurazione
  - selector:
    app=web
  - template label:
    app=frontend

- Obiettivo
  - Identificare perché i Pod NON vengono creati

- Validazione
  - Nessun Pod Running
  - Fix del problema

---

<details>
<summary>Soluzione</summary>
```
k create deploy mismatch-app --image=nginx --replicas=2 --dry-run=client -o yaml > mismatch-app.yaml
```
Modificare il file introducendo l'errore:  

```
  apiVersion: apps/v1
kind: Deployment
metadata:
  name: mismatch-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx


k apply -f mismatch-app.yaml
```

Problema:
- selector ≠ label template


Fix:

```yaml
selector:
  matchLabels:
    app: web

template:
  metadata:
    labels:
      app: web
```

Debug:

```sh
k describe deploy mismatch-app
```

</details>

---

## LS-8 — Service seleziona subset di Pod

- Deployment: `multi-tier-app`

- Labels:
  - app=myapp
  - tier=frontend
  - tier=backend

- Creare Service:
  - Nome: `frontend-svc`
  - Selector:
    - tier=frontend

- Obiettivo
  - Il Service deve selezionare solo frontend

---

<details>
<summary>Soluzione</summary>

```yaml

apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    tier: frontend
  ports:
  - port: 80
```

Verifica:

```sh
k get pods --show-labels
k get endpoints frontend-svc
```

</details>

---

## LS-9 — Label su Deployment già esistente

- Deployment: `update-label-deploy`

- Task
  - Aggiungere label:
    team=devops

- Obiettivo
  - Applicare label sia al Deployment che ai Pod

---

<details>
<summary>Soluzione</summary>

```sh
k label deploy update-label-deploy team=devops
```

 per i Pod:

```sh
k edit deploy update-label-deploy
```

aggiungere:

```yaml
template:
  metadata:
    labels:
      team: devops
```

</details>

---

## LS-10 — Selector complesso con matchExpressions

- Deployment: `expr-app`

- Labels:
  - env=prod
  - version=v1

- Selector:
  - env in (prod)
  - version notin (v2)

---

<details>
<summary>Soluzione</summary>

```yaml
selector:
  matchExpressions:
  - key: env
    operator: In
    values:
    - prod
  - key: version
    operator: NotIn
    values:
    - v2
```

</details>

---

## LS-11 — Label filtering CLI avanzato

- Obiettivo
  - Trovare Pod:
    - app=myapp
    - ma NON environment=prod

---

<details>
<summary>Soluzione</summary>

```sh
kubectl get pods -l 'app=myapp,environment!=prod'
```

Altri esempi:

```sh
kubectl get pods -l 'environment in (dev,staging)'
kubectl get pods -l 'tier notin (backend)'
```

</details>

---

## LS-12 — NodeSelector + Label dinamica

- Pod: `dynamic-node-app`

- Task
  - Etichettare un nodo:
    gpu=true
  - Usare nodeSelector per schedulare il Pod

---

<details>
<summary>Soluzione</summary>

```sh
k label node <node-name> gpu=true
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dynamic-node-app
spec:
  containers:
  - name: app
    image: nginx
  nodeSelector:
    gpu: "true"
```

Verifica:

```sh
k get pod -o wide
```

</details>

---
## LS-13 — Label su Pod in fase di creazione

Creare un Pod chiamato `labeled-pod`

- Specifiche
  - Image: nginx

- Labels richieste
  - app=test
  - env=dev

- Validazione
  - `kubectl get pod labeled-pod --show-labels`

---

<details>
<summary>Soluzione</summary>

```sh
k run labeled-pod --image=nginx --labels=app=test,env=dev
k get pod labeled-pod --show-labels
```

</details>

---

## LS-14 — Rimuovere una label da un Pod

Pod esistente: `labeled-pod`

- Task
  - Rimuovere la label `env`

- Validazione
  - `kubectl get pod labeled-pod --show-labels`
  - La label `env` non è più presente

---

<details>
<summary>Soluzione</summary>

```sh
k label pod labeled-pod env-
k get pod labeled-pod --show-labels
```

</details>

---

## LS-15 — Service con selector su singola label

Creare Deployment `simple-app`

- Specifiche
  - Image: nginx
  - Replicas: 2
  - Label:
    - app=simple

Creare Service `simple-svc`

- Selector
  - app=simple

- Validazione
  - `kubectl get endpoints simple-svc`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: simple
  template:
    metadata:
      labels:
        app: simple
    spec:
      containers:
      - name: nginx
        image: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: simple-svc
spec:
  selector:
    app: simple
  ports:
  - port: 80
    targetPort: 80
```

```sh
k apply -f ls-15.yaml
k get endpoints simple-svc
```

</details>

---

## LS-16 — Pod selector con più condizioni via CLI

- Obiettivo
  - Trovare Pod con:
    - app=frontend
    - tier=web

- Validazione
  - Usare selector corretto da CLI

---

<details>
<summary>Soluzione</summary>

```sh
kubectl get pods -l app=frontend,tier=web
```

</details>

---

## LS-17 — Label su Namespace

Creare namespace `team-a`

- Task
  - Aggiungere label:
    - team=blue

- Validazione
  - `kubectl get ns --show-labels`

---

<details>
<summary>Soluzione</summary>

```sh
k create ns team-a
k label ns team-a team=blue
k get ns --show-labels
```

</details>

---

## LS-18 — MatchExpressions con Exists

Creare Deployment `exists-app`

- Specifiche
  - Image: nginx
  - Replicas: 2

- Selector
  - key: `track`
  - operator: `Exists`

- Template labels
  - track=stable

- Validazione
  - Deployment valido
  - Pod creati correttamente

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: exists-app
spec:
  replicas: 2
  selector:
    matchExpressions:
    - key: track
      operator: Exists
  template:
    metadata:
      labels:
        track: stable
    spec:
      containers:
      - name: nginx
        image: nginx
```

```sh
k apply -f exists-app.yaml
k get deploy
k get pods --show-labels
```

</details>

---

## LS-19 — MatchExpressions con NotIn

Creare Deployment `notin-app`

- Specifiche
  - Image: nginx
  - Replicas: 1

- Selector
  - env notin (test)

- Template labels
  - env=prod

- Validazione
  - Deployment creato correttamente

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notin-app
spec:
  replicas: 1
  selector:
    matchExpressions:
    - key: env
      operator: NotIn
      values:
      - test
  template:
    metadata:
      labels:
        env: prod
    spec:
      containers:
      - name: nginx
        image: nginx
```

</details>

---

## LS-20 — Service senza endpoint per typo nella label

Creare Deployment `typo-app`

- Label nei Pod
  - app=frontend

Creare Service `typo-svc`

- Selector errato
  - app=fronend

- Obiettivo
  - Individuare il typo e correggerlo

- Validazione
  - Endpoints presenti dopo il fix

---

<details>
<summary>Soluzione</summary>

Problema:
- `fronend` ≠ `frontend`

Debug:

```sh
kubectl get pods --show-labels
kubectl describe svc typo-svc
kubectl get endpoints typo-svc
```

Fix:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: typo-svc
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
```

</details>

---

## LS-21 — Label e selector su ReplicaSet

Creare un ReplicaSet chiamato `label-rs`

- Specifiche
  - Image: nginx
  - Replicas: 2
  - Label: app=rsdemo

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
  name: label-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: rsdemo
  template:
    metadata:
      labels:
        app: rsdemo
    spec:
      containers:
      - name: nginx
        image: nginx
```

```sh
k apply -f label-rs.yaml
k get rs
k get pods --show-labels
```

</details>

---

## LS-22 — Selezionare Pod senza una label

- Obiettivo
  - Trovare Pod che NON hanno la label `team`

- Validazione
  - Usare selector CLI corretto

---

<details>
<summary>Soluzione</summary>

```sh
kubectl get pods -l '!team'
```

</details>

---

## LS-23 — Aggiungere label a un Service

Service esistente: `simple-svc`

- Task
  - Aggiungere label:
    - owner=platform

- Validazione
  - `kubectl get svc simple-svc --show-labels`

---

<details>
<summary>Soluzione</summary>

```sh
k label svc simple-svc owner=platform
k get svc simple-svc --show-labels
```

</details>

---

## LS-24 — Selector multiplo con namespace e labels

- Obiettivo
  - Trovare Pod nel namespace `default` con:
    - app=frontend
    - tier=web
    - environment!=prod

- Validazione
  - Usare comando CLI corretto

---

<details>
<summary>Soluzione</summary>

```sh
kubectl get pods -n default -l 'app=frontend,tier=web,environment!=prod'
```

</details>

---
