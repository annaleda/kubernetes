
###  Ingress (6 esercizi)

## ING-1 — Ingress Base

Creare Deployment ingress-web

Creare Service ingress-svc

Creare Ingress web-ingress

Specifiche

Host: app.local

Path: /

Backend: ingress-svc

Validazione

kubectl describe ingress
```  
``` 
---

## ING-2 — Path Based Routing

Due Deployment:

frontend

backend

Ingress

/front → frontend

/back → backend

Validazione

Routing corretto

``` 
```
---

## ING-3 — TLS

Creare Secret TLS

Ingress

TLS abilitato

Host: secure.local

Validazione

Sezione TLS presente
```  
```
---

### ING-4 — Multiple Hosts

Ingress

app1.local

app2.local

Backend distinti

Validazione

Regole host separate

```  
```
---

### ING-5 — Default Backend

Ingress con default backend

Validazione

Richiesta a host sconosciuto va al default
```  
```
---

### ING-6 — Rewrite Target

Usare annotation rewrite-target

Path /app → /

Validazione

Routing corretto
```  
```
---
