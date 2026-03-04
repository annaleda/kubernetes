
### ServiceAccount (6 esercizi)

## SA-1 — Creazione ServiceAccount

- Namespace: `sa-test`
- Creare ServiceAccount: `app-sa`
- Pod: `sa-pod`
  - Image: nginx
  - Usare serviceAccountName: app-sa
- Validazione
  - Pod usa SA corretto
---
<details>
<summary>Soluzione</summary>
  
```
 k create ns sa-test
 k create sa app-sa
 k run sa-pod --image=nginx --dry-run=client -o yaml > sa-pod.yaml
 

 vi sa-pod.yaml


  apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: null
    labels:
      run: sa-pod
    name: sa-pod
  spec:
    containers:
    - image: nginx
      name: sa-pod
    serviceAccountName: app-sa

k apply -f sa-pod.yaml
```
</details>

---

## SA-2 — Default ServiceAccount

- Creare Pod senza specificare SA
- Validazione
  - Usa default ServiceAccount

---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

## SA-3 — Disabilitare Automount

- Pod: no-token-pod
- Configurazione
  - automountServiceAccountToken: false
- Validazione
  - Nessun token montato in `/var/run/secrets`
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

### SA-4 — ServiceAccount + Role

- Creare Role con permessi su pods
- Collegarla a ServiceAccount
  - Verificare accesso con `kubectl auth can-i`

---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

### SA-5 — SA in Deployment

- Deployment: `sa-deployment`
- Replicas: 2
- serviceAccountName: custom-sa
- Validazione
  - Tutti i Pod usano SA corretto
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

### SA-6 — Multiple Namespace SA

- Creare ServiceAccount con stesso nome in 2 namespace
- Validazione
  - Sono entità distinte
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---
