
###  Configuration (6 esercizi)

## CONF-1 — ConfigMap

- Creare ConfigMap: `app-config`
  - key: APP_MODE
  - value: production
- Montare in Pod config-pod
- Validazione
  - Variabile disponibile nel container
```  
``` 
---

## CONF-2 — Secret

- Creare Secret: `db-secret`
  - DB_USER=admin
  - DB_PASS=pass123
  - Montare come volume
- Validazione
  - File presente nel container

``` 
```
---

## CONF-3 — ResourceQuota

- Namespace: `quota-ns`
- Impostare ResourceQuota
  - Pods: 2
  - Requests CPU totali: 500m
- Validazione
  - Creazione terzo pod fallisce
```  
```
---

### CONF-4 — LimitRange

- Namespace: `limit-ns`
- Creare LimitRange
  - Default CPU limit: 200m
  - Default memory limit: 128Mi
- Validazione
  - Pod senza limiti riceve default

```  
```
---

### CONF-5 — Requests vs Limits

- Deployment: `resource-test`
- Container
  - Requests: 100m
  - Limits: 200m
- Validazione
  - Differenza visibile nel describe
```  
```
---

### CONF-6 — ConfigMap + Secret insieme

- Pod: `env-app`
- Usare
  - ConfigMap per config non sensibili
  - Secret per credenziali
- Validazione
  - Entrambi montati correttamente
```  
```
---
