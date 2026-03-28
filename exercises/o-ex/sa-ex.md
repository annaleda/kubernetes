
### ServiceAccount (12 esercizi)

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

## SA-7 — ServiceAccount con Secret manuale

- Namespace: `sa-secret-ns`

- Creare ServiceAccount: `custom-sa`

- Creare Secret manuale (token)

- Associare il Secret alla ServiceAccount

- Validazione
  - Secret visibile nella SA

---

<details>
<summary>Soluzione</summary>

```sh
k create ns sa-secret-ns
k create sa custom-sa -n sa-secret-ns
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: custom-sa-token
  namespace: sa-secret-ns
  annotations:
    kubernetes.io/service-account.name: custom-sa
type: kubernetes.io/service-account-token
```

```sh
k apply -f secret.yaml
k describe sa custom-sa -n sa-secret-ns
```

</details>

---

## SA-8 — Override ServiceAccount default

- Namespace: `override-ns`

- Configurazione
  - Modificare default ServiceAccount
  - automountServiceAccountToken: false

- Obiettivo
  - Tutti i Pod NON montano token di default

---

<details>
<summary>Soluzione</summary>

```sh
k create ns override-ns
k edit sa default -n override-ns
```

Aggiungere:

```yaml
automountServiceAccountToken: false
```

Verifica:

```sh
k run test --image=nginx -n override-ns
k exec -it test -n override-ns -- ls /var/run/secrets
```

</details>

---

## SA-9 — Pod con SA sbagliato (debug)

- Pod: `wrong-sa-pod`

- Problema
  - ServiceAccount non esiste

- Obiettivo
  - Identificare errore e correggere

---

<details>
<summary>Soluzione</summary>

Errore:

```sh
k describe pod wrong-sa-pod
```

👉 `serviceaccount not found`

Fix:

```sh
k create sa correct-sa
```

oppure modificare Pod:

```yaml
serviceAccountName: default
```

</details>

---

## SA-10 — SA con accesso limitato a namespace

- Namespace: `limited-sa`

- Creare:
  - ServiceAccount
  - Role (solo get pods)
  - RoleBinding

- Obiettivo
  - Accesso SOLO nel namespace

---

<details>
<summary>Soluzione</summary>

```sh
k create ns limited-sa
k create sa limited-sa -n limited-sa

k create role pod-reader \
  --verb=get \
  --resource=pods \
  -n limited-sa

k create rolebinding rb \
  --role=pod-reader \
  --serviceaccount=limited-sa:limited-sa \
  -n limited-sa
```

```sh
k auth can-i get pods \
  --as=system:serviceaccount:limited-sa:limited-sa \
  -n limited-sa
```

</details>

---

## SA-11 — SA + imagePullSecrets

- Namespace: `registry-ns`

- Creare:
  - Secret docker-registry
  - ServiceAccount con imagePullSecrets

- Pod usa quella SA

- Obiettivo
  - Pull da registry privato

---

<details>
<summary>Soluzione</summary>

```sh
k create ns registry-ns
```

```sh
k create secret docker-registry reg-secret \
  --docker-username=user \
  --docker-password=pass \
  --docker-server=myrepo \
  -n registry-ns
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: reg-sa
  namespace: registry-ns
imagePullSecrets:
- name: reg-secret
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-pod
  namespace: registry-ns
spec:
  serviceAccountName: reg-sa
  containers:
  - name: app
    image: myrepo/private-image
```

</details>

---

## SA-12 — Verifica token dentro container

- Pod: `token-check`

- Obiettivo
  - Verificare presenza token ServiceAccount

- Validazione
  - File presente nel container

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: token-check
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","ls /var/run/secrets/kubernetes.io/serviceaccount && sleep 3600"]
```

```sh
k exec -it token-check -- ls /var/run/secrets/kubernetes.io/serviceaccount
```

</details>

---
