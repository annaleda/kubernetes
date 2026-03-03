
###  Labels and Selectors (6 esercizi)

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
```
</details>

---
