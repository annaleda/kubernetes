
###  Auth (6 esercizi)

## AUTH-1 — Role + RoleBinding

- Namespace: `dev-auth`
- Creare Role: `pod-reader`
  - Permessi: get, list, watch su pods
- Creare RoleBinding
  - Collegare Role a utente dev-user
- Validazione
  - `kubectl auth can-i list pods --as=dev-user -n dev-auth`
```  
``` 
---

## AUTH-2 — Limitare Accesso ai ConfigMap

- Namespace: `secure-ns`
- Creare Role
  - Solo get su configmaps
- Associare a ServiceAccount
- Validazione
  - ServiceAccount può leggere ma non creare ConfigMap

``` 
```
---

## AUTH-3 — ClusterRole

- Creare ClusterRole: `node-reader`
  - get, list su nodes
- Creare ClusterRoleBinding
- Validazione
  - Utente può leggere nodes cluster-wide
```  
```
---

### AUTH-4 — Verifica Permessi

- Usare comando:

      kubectl auth can-i
  
- Obiettivo
  - Verificare se un utente può eliminare pod

```  
```
---

### AUTH-5 — Deny by Default

- Namespace: `restricted`
- Creare ServiceAccount senza RoleBinding
- Validazione
  - Non può creare Pod
```  
```
---

### AUTH-6 — Multiple RoleBinding

- Namespace: `multi-role`
- Creare 2 Role diverse
- Associare entrambe alla stessa ServiceAccount
- Validazione
  - Permessi cumulativi funzionano
```  
```
---
