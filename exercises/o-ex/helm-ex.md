
### Helm (6 esercizi)

## HELM-1 — Installazione Chart

Namespace: helm-test

Installare chart nginx

Specifiche

Release name: web-release

Chart: bitnami/nginx

Validazione

helm list -n helm-test
```  
``` 
---

## HELM-2 — Override Values via CLI

Installare release custom-nginx

Specifiche

ReplicaCount: 3

Service type: ClusterIP

Usare flag --set

Validazione

kubectl get deployment mostra 3 repliche

``` 
```
---

## HELM-3 — Override con values.yaml

Creare file custom-values.yaml

Specifiche

replicaCount: 2

service.type: NodePort

Installare release file-nginx usando file

Validazione

Service è NodePort
```  
```
---

### HELM-4 — Upgrade Release

Aggiornare release web-release

Specifiche

Cambiare replicaCount a 4

Validazione

helm history web-release

Deployment aggiornato

```  
```
---

### HELM-5 — Rollback Release

Eseguire rollback della release web-release

Alla revisione precedente

Validazione

helm history mostra nuova revisione rollback
```  
```
---

### HELM-6 — Template Rendering

Eseguire render locale del chart nginx

Specifiche

Non installare nel cluster

Output in file nginx-rendered.yaml

Validazione

File contiene manifest Kubernetes
```  
```
---
