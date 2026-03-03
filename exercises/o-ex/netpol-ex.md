
###  Network policies(6 esercizi)

## NP-1 — Default Deny

- Namespace: `net-secure`

- Creare NetworkPolicy
  - Nessun traffico consentito

- Validazione
  - Pod non comunicano tra loro
```  
``` 
---

## NP-2 — Allow Ingress da Label

- Consentire traffico solo da Pod con label `role=frontend`
- Validazione
  - Solo frontend comunica

``` 
```
---

## NP-3 — Allow Egress DNS

- Consentire egress solo verso porta 53
- Validazione
  - DNS funziona
  - Altro traffico bloccato
```  
```
---

### NP-4 — Namespace Selector

- Consentire traffico solo da namespace `trusted`
- Validazione
  - Altri namespace bloccati

```  
```
---

### NP-5 — Port Restriction

- Consentire traffico solo su porta 80
- Validazione
  - Porta diversa bloccata
```  
```
---

### NP-6 — Combined Policy

- Consentire:
  - Ingress da namespace trusted
  - Solo porta 443

- Validazione
  - Traffico limitato correttamente
```  
```
---
