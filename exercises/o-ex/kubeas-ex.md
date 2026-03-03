
###  Kube-apiserver (6 esercizi)

## KA-1 — Identificare Endpoint API Server

- Recuperare endpoint del cluster
- Validazione
  - `kubectl cluster-info`
```  
``` 
---

## KA-2 — Verificare Certificati

- Esaminare certificati nel kubeconfig
- Obiettivo
  - Identificare campo certificate-authority-data
- Validazione
  - `kubectl config view`

``` 
```
---

## KA-3 — Abilitare Audit Log (Concettuale)

- Descrivere come si abilita audit logging
nel kube-apiserver

- Obiettivo
  - Identificare flag `--audit-policy-file`
```  
```
---

### KA-4 — Admission Controllers

- Identificare admission controller attivi
- Obiettivo
  - Comprendere ruolo di:
    - NamespaceLifecycle
    - LimitRanger
    - ResourceQuota
- Validazione
  - Descrizione teorica o flag API server

```  
```
---

### KA-5 — API Resources

- Elencare tutte le risorse API disponibili
- Validazione
  - `kubectl api-resources`
```  
```
---

### KA-6 — Versione API

- Verificare versione cluster
- Validazione
  - `kubectl version --short`
```  
```
---
