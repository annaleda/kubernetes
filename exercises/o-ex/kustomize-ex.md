
###  Kustomize (6 esercizi)

## KUS-1 — Base Configuration

- Struttura:

      base/
        deployment.yaml
        service.yaml
        kustomization.yaml

- Deployment image: nginx:1.21

- Obiettivo
  - Applicare la base

- Validazione
  - Deployment creato
---
<details>
<summary>Soluzione</summary>
  
```
mkdir base
cd base
k create ns kustomize
k create deploy deploy --image=nginx:1.21 -n kustomize --dry-run=client -o yaml > deploy.yaml
k expose deploy deploy --name svc --port=80 -n kustomize --dry-run=client -o yaml > svc.yaml

vi kustomization.yaml


apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deploy.yaml
- svc.yaml



k apply -k .
```
</details>

---

## KUS-2 — Overlay Dev

- Struttura:

      overlays/dev/
        kustomization.yaml

- Namespace: dev
  - Image override: nginx:1.23
- Validazione
  - kubectl kustomize overlays/dev mostra override

---
<details>
<summary>Soluzione</summary>
  
```
k create ns dev

mkdir base2
cd base2

k create deploy deploy --image=nginx:1.21 -n dev --dry-run=client -o yaml > deploy.yaml
k expose deploy deploy --name svc --port=80 -n dev --dry-run=client -o yaml > svc.yaml

vi kustomization.yaml


apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deploy.yaml
- svc.yaml


k apply -k .


mkdir -p overlays/dev
cd overlays/dev

vi kustomization.yaml


apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev
resources:
- ../../base
images:
- name: nginx
  newTag: "1.23"



k apply -k .
k kustomize .

->
 nginx:1.23
```
</details>

---

## KUS-3 — Name Prefix

- Applicare namePrefix: dev-
- Validazione
  - Deployment diventa dev-nginx
---
<details>
<summary>Soluzione</summary>
  
```
vi kustomization.yaml

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev
resources:
- ../../base
namePrefix: dev-
images:
- name: nginx
  newTag: "1.23"


k apply -k .
k kustomize .
```
</details>

---

### KUS-4 — ConfigMap Generator

- Generare ConfigMap da file:

      app.properties
              {key1=value1
               key2=value2}

- Usare configMapGenerator

- Validazione
  - ConfigMap generata automaticamente
---
<details>
<summary>Soluzione</summary>
  
```

 cd ../..
 cd base2
 vi app.properties
 vi kustomization.yaml


apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
configMapGenerator:
- name: app-config
  files:
  - app.properties


 k apply -k .

 k kustomize .

 k get cm
NAME                    DATA   AGE
app-config-84gfck8fhb   1      37s
kube-root-ca.crt        1      29m

```
</details>

---

### KUS-5 — Patch Strategica

- Modificare Deployment
  - Aggiungere replicaCount: 3

- Usare patchStrategicMerge
- Validazione
  - Replica aggiornata
---
<details>
<summary>Soluzione</summary>
  
```
vi patch-replica.yaml


apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy
spec:
  replicas: 3


vi kustomization.yaml


apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
configMapGenerator:
- name: app-config
  files:
  - app.properties
patchesStrategicMerge:
- patch-replica.yaml

 k apply -k .

 k kustomize .
```
</details>

---

### KUS-6 — Image Transformer

- Cambiare image:
    - Da nginx:1.21
    - A nginx:1.25
- Usare sezione images: in kustomization
- Validazione
   - Output mostra nuova immagine
---
<details>
<summary>Soluzione</summary>
  
```
cd ..
mkdir base3
cd base3

k create deploy deploy2 --image=nginx:1.21 -n dev --dry-run=client -o yaml > deploy.yaml
k expose deploy deploy2 --name svc2 --port=80 -n dev --dry-run=client -o yaml > svc.yaml

vi kustomization.yaml


apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deploy.yaml
- svc.yaml
images:
- name: nginx
  newTag: "1.25"

k apply -k .
k kustomize .
k get deploy -n dev -o yaml | grep image

```
</details>

---
