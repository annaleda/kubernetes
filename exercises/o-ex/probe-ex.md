
###  Probe (6 esercizi)

## PR-1 — Liveness Probe HTTP

Deployment: web-liveness

Specifiche

Image: nginx

Replicas: 2

Aggiungere Liveness Probe

HTTP GET

Path: /

Port: 80

initialDelaySeconds: 10

periodSeconds: 5

Validazione

kubectl describe pod mostra Liveness configurata
```  
``` 
---

## PR-2 — Readiness Probe HTTP

Deployment: api-readiness

Specifiche

Image: nginx

Replicas: 3

Aggiungere Readiness Probe

HTTP GET

Path: /

Port: 80

initialDelaySeconds: 5

Validazione

Pod diventano Ready

Service riceve endpoint solo se Ready

``` 
```
---

## PR-3 — Liveness con comando (exec)

Pod: exec-liveness

Container

Image: busybox

Command: sleep 3600

Liveness Probe

exec:

cat /tmp/healthy

initialDelaySeconds: 5

Obiettivo

Creare manualmente il file per far passare la probe

Validazione

Senza file il Pod viene riavviato
```  
```
---

### PR-4 — Readiness + Liveness combinate

Deployment: combined-probes

Specifiche

Image: nginx

Liveness

HTTP GET /

Readiness

HTTP GET /

initialDelaySeconds: 15

Validazione

Pod Running ma non Ready nei primi 15 secondi

```  
```
---

### PR-5 — Startup Probe

Deployment: startup-app

Specifiche

Image: nginx

Aggiungere Startup Probe

HTTP GET /

failureThreshold: 30

periodSeconds: 5

Validazione

Startup protegge da restart prematuri
```  
```
---

### PR-6 — Debug Probe Failure

Deployment: broken-probe

Specifiche

Image: nginx

Liveness

HTTP GET /wrongpath

Obiettivo

Identificare motivo dei restart

Correggere il path

Validazione

Pod stabile dopo correzione
```  
```
---
