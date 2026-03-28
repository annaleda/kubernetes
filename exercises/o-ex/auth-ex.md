
###  Auth (12 esercizi)

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
- Creare Role `cm-reader` 
  - Solo get su configmaps
- Associare a ServiceAccount
- Validazione
  - ServiceAccount può leggere ma non creare ConfigMap

---
<details>
<summary>Soluzione</summary>
  
```
k create ns secure-ns
k create role cm-reader --verb=get --resource=configmaps -n secure-ns

kubectl create rolebinding cm-reader-rb --role=cm-reader --serviceaccount=secure-ns:default -n secure-ns

k auth can-i create cm  -n secure-ns --as=system:serviceaccount:secure-ns:default
k auth can-i get cm  -n secure-ns --as=system:serviceaccount:secure-ns:default
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
# k config set-credentials dev-user --client-key=tls.key --client-certificate=tls.crt
k create clusterrole node-reader --verb=get --resource=nodes 
k create clusterrolebinding node-reader-rb --user=dev-user --clusterrole=node-reader 

k auth can-i get nodes --as=dev-user 

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
k create ns multi-role
k create sa multi-sa -n multi-role
k create role role-pod-reader --verb=get,list,watch --resource=pods -n multi-role
k create role role-cm-reader  --verb=get --resource=configmaps -n multi-role

k create rolebinding rb-pods --role=role-pod-reader --serviceaccount=multi-role:multi-sa -n multi-role
k create rolebinding rb-cm --role=role-cm-reader --serviceaccount=multi-role:multi-sa -n multi-role

k auth can-i get pods -n multi-role --as=system:serviceaccount:multi-role:multi-sa
k auth can-i create pods -n multi-role --as=system:serviceaccount:multi-role:multi-sa
k auth can-i get configmap -n multi-role --as=system:serviceaccount:multi-role:multi-sa
```
</details>

---

## AUTH-7 — Role con risorsa specifica

- Namespace: `limited-ns`

- Creare Role: `single-pod-reader`
  - Permessi: get su pods
  - Solo sul pod: `mypod`

- Creare RoleBinding per user `dev-user`

- Validazione
  - Può leggere solo quel pod

---

<details>
<summary>Soluzione</summary>

```sh
k create ns limited-ns

k create role single-pod-reader \
  --verb=get \
  --resource=pods \
  --resource-name=mypod \
  -n limited-ns

k create rolebinding single-pod-rb \
  --role=single-pod-reader \
  --user=dev-user \
  -n limited-ns
```

```sh
k auth can-i get pod mypod --as=dev-user -n limited-ns
k auth can-i get pod otherpod --as=dev-user -n limited-ns
```

</details>

---

## AUTH-8 — ServiceAccount custom

- Namespace: `sa-ns`

- Creare ServiceAccount: `custom-sa`

- Pod: `sa-pod`
  - Usa `custom-sa`

- Obiettivo
  - Il pod deve usare ServiceAccount specifico

---

<details>
<summary>Soluzione</summary>

```sh
k create ns sa-ns
k create sa custom-sa -n sa-ns
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
  namespace: sa-ns
spec:
  serviceAccountName: custom-sa
  containers:
  - name: nginx
    image: nginx
```

```sh
k describe pod sa-pod -n sa-ns
```

</details>

---

## AUTH-9 — ClusterRole per più namespace

- Creare ClusterRole: `pod-reader-global`
  - get, list pods

- Associare a ServiceAccount in namespace `team-a`

- Obiettivo
  - Accesso multi-namespace

---

<details>
<summary>Soluzione</summary>

```sh
k create ns team-a
k create sa team-sa -n team-a

k create clusterrole pod-reader-global \
  --verb=get,list \
  --resource=pods

k create clusterrolebinding team-binding \
  --clusterrole=pod-reader-global \
  --serviceaccount=team-a:team-sa
```

```sh
k auth can-i get pods \
  --as=system:serviceaccount:team-a:team-sa -A
```

</details>

---

## AUTH-10 — Debug RBAC (permesso negato)

- Namespace: `debug-ns`

- Scenario
  - ServiceAccount NON riesce a leggere Pod

- Obiettivo
  - Identificare e risolvere il problema

---

<details>
<summary>Soluzione</summary>

```sh
k auth can-i get pods \
  --as=system:serviceaccount:debug-ns:default \
  -n debug-ns
```

Se NO:

```sh
k create role pod-reader \
  --verb=get,list \
  --resource=pods \
  -n debug-ns

k create rolebinding rb \
  --role=pod-reader \
  --serviceaccount=debug-ns:default \
  -n debug-ns
```

</details>

---

## AUTH-11 — Role con più risorse

- Namespace: `multi-resource`

- Creare Role:
  - get su pods
  - get su services

- Associare a ServiceAccount

---

<details>
<summary>Soluzione</summary>

```sh
k create ns multi-resource
k create sa multi-sa -n multi-resource

k create role multi-role \
  --verb=get \
  --resource=pods,services \
  -n multi-resource

k create rolebinding multi-rb \
  --role=multi-role \
  --serviceaccount=multi-resource:multi-sa \
  -n multi-resource
```

</details>

---

## AUTH-12 — Verifica accesso cross-namespace

- Namespace:
  - ns1
  - ns2

- ServiceAccount in ns1

- Obiettivo
  - Verificare accesso a risorse in ns2

---

<details>
<summary>Soluzione</summary>

```sh
k create ns ns1
k create ns ns2

k create sa test-sa -n ns1
```

Test:

```sh
k auth can-i get pods \
  --as=system:serviceaccount:ns1:test-sa \
  -n ns2
```

risultato: NO

Per abilitare:

```sh
k create role pod-reader \
  --verb=get \
  --resource=pods \
  -n ns2

k create rolebinding rb \
  --role=pod-reader \
  --serviceaccount=ns1:test-sa \
  -n ns2
```

</details>

---
