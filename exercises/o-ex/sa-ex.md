
### ServiceAccount (6 esercizi)

## SA-1 — Creazione ServiceAccount

Namespace: sa-test

Creare ServiceAccount: app-sa

Pod: sa-pod

Image: nginx

Usare serviceAccountName: app-sa

Validazione

Pod usa SA corretto
```  
``` 
---

## SA-2 — Default ServiceAccount

Creare Pod senza specificare SA

Validazione

Usa default ServiceAccount

``` 
```
---

## SA-3 — Disabilitare Automount

Pod: no-token-pod

Configurazione

automountServiceAccountToken: false

Validazione

Nessun token montato in /var/run/secrets
```  
```
---

### SA-4 — ServiceAccount + Role

Creare Role con permessi su pods

Collegarla a ServiceAccount

Verificare accesso con kubectl auth can-i

```  
```
---

### SA-5 — SA in Deployment

Deployment: sa-deployment

Replicas: 2

serviceAccountName: custom-sa

Validazione

Tutti i Pod usano SA corretto
```  
```
---

### SA-6 — Multiple Namespace SA

Creare ServiceAccount con stesso nome in 2 namespace

Validazione

Sono entità distinte
```  
```
---
