
### CDR and Operators (6 esercizi)

## CRD-1 — Creare CRD Base

Definire CRD: apps.stable.example.com

Kind: MyApp

Version: v1

Scope: Namespaced

Validazione

kubectl get crd
```  
``` 
---

## CRD-2 — Creare Custom Resource

Creare oggetto MyApp

Nome: example-app

Validazione

kubectl get myapp

``` 
```
---

## CRD-3 — Versioning CRD

Aggiungere versione v2 al CRD

Validazione

CRD mostra multiple versioni
```  
```
---

### CRD-4 — Status Subresource

Abilitare subresource status nel CRD

Validazione

kubectl get myapp -o yaml mostra status

```  
```
---

### CRD-5 — RBAC per CRD

Creare Role che permette get su MyApp

Validazione

kubectl auth can-i get myapp
```  
```
---

### CRD-6 — Concept Operator

Descrivere come un Operator controller osserva MyApp

Simulare creando Deployment basato su campo spec

Validazione

Deployment creato manualmente come simulazione
```  
```
---
