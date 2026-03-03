
###  Auth (6 esercizi)

## AUTH-1 — Role + RoleBinding

- Namespace: `dev-auth`
- Creare Role: `pod-reader`
  - Permessi: get, list, watch su pods
- Creare RoleBinding: `pod-reader-rb`
  - Collegare Role a utente dev-user
- Validazione
  - `kubectl auth can-i list pods --as=dev-user -n dev-auth`
---
<details>
<summary>Soluzione</summary>
  
```
k create ns dev-auth
k config set-credentials dev-user --client-key=tls.key --client-certificate=tls.crt
k create role pod-reader --verb=get,list,watch --resource=pods -n dev-auth
k create rolebinding pod-reader-rb --user=dev-user --role=pod-reader -n dev-auth

```
</details>

---

## AUTH-2 — Limitare Accesso ai ConfigMap

- Namespace: `secure-ns`
- Creare Role
  - Solo get su configmaps
- Associare a ServiceAccount
- Validazione
  - ServiceAccount può leggere ma non creare ConfigMap

---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

## AUTH-3 — ClusterRole

- Creare ClusterRole: `node-reader`
  - get, list su nodes
- Creare ClusterRoleBinding
- Validazione
  - Utente può leggere nodes cluster-wide
---
<details>
<summary>Soluzione</summary>
  
```
# k create ns dev-auth
# k config set-credentials dev-user --client-key=tls.key --client-certificate=tls.crt
k create clusterrole node-reader --verb=get --resource=nodes -n dev-auth
k create clusterrolebinding node-reader-rb --user=dev-user --clusterrole=node-reader -n dev-auth

k auth can-i get nodes --as=dev-user -n dev-auth

```
</details>

---

### AUTH-4 — Verifica Permessi

- Usare comando:

      kubectl auth can-i
  
- Obiettivo
  - Verificare se un utente può eliminare pod

---
<details>
<summary>Soluzione</summary>
  
```
 k auth can-i delete po --as=dev-user -n dev-auth

```
</details>

---

### AUTH-5 — Deny by Default

- Namespace: `restricted`
- Creare ServiceAccount senza RoleBinding
- Validazione
  - Non può creare Pod
---
<details>
<summary>Soluzione</summary>
  
```
 k create ns restricted
 k create sa restricted-sa -n restricted --dry-run=client -o yaml > restricted-sa.yaml

k auth can-i create pods --as=system:serviceaccount:restricted:restricted-sa -n restricted

```
</details>

---

### AUTH-6 — Multiple RoleBinding

- Namespace: `multi-role`
- Creare 2 Role diverse
- Associare entrambe alla stessa ServiceAccount
- Validazione
  - Permessi cumulativi funzionano
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---
