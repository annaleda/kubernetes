
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
```  
``` 
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

``` 
```
---

## LS-3 — Modifica Label Live

Creare Pod `temp-app`
  - Image: nginx

- Task
  - Aggiungere label: environment=dev
  - Poi cambiarla in environment=prod

- Validazione
  - kubectl get pod temp-app --show-labels
```  
```
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

```  
```
---

### LS-5 — Node Selector

Label su un nodo: `disktype=ssd`

Creare Pod `ssd-app`

- Configurazione
  - nodeSelector:
    - disktype=ssd

- Validazione
  - Pod schedulato solo sul nodo corretto
```  
```
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
```  
```
---
