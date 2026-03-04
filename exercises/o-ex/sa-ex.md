
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
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
  namespace: sa-test
spec:
  containers:
  - name: nginx
    image: nginx

# non mettere sa
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
apiVersion: v1
kind: Pod
metadata:
  name: no-token-pod
spec:
  automountServiceAccountToken: false

  containers:
  - name: nginx
    image: nginx


kubectl exec -it no-token-pod -- ls /var/run/secrets/kubernetes.io
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
 k create ns sa-test
 k create sa sa-roling -n sa-test
 k create role sa-role --verb=list,get --resource=pods -n sa-test
 k create rolebinding sa-roleb --serviceaccount=sa-test:sa-roling --role=sa-role -n sa-test

 k auth can-i list pods --as=system:serviceaccount:sa-test:sa-roling -n sa-test
 k auth can-i get pods --as=system:serviceaccount:sa-test:sa-roling -n sa-test
 
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
 k create sa custom-sa
 k create deploy sa-deployment --replicas=2 --image=nginx --dry-run=client -o yaml > dep.yaml

 vi dep.yaml

  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: sa-deployment
    name: sa-deployment
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: sa-deployment
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: sa-deployment
      spec:
        containers:
        - image: nginx
          name: nginx
        serviceAccountName: custom-sa


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
k create ns one
k create ns two
k create sa sa-duplicate -n one
k create sa sa-duplicate -n two

kubectl get sa -n one
kubectl get sa -n two
```
</details>

---
