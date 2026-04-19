
###  Kustomize (12 esercizi)

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

## KUS-7 — Common Labels

- Applicare label comune:
  - env: dev

- Usare `commonLabels`

- Validazione
  - Tutte le risorse hanno la label `env=dev`

---
<details>
<summary>Soluzione</summary>

```
vi kustomization.yaml

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
commonLabels:
  env: dev

k apply -k .
k get deploy --show-labels
```

</details>

---

## KUS-8 — Namespace override

- Impostare namespace: `kustom-ns`

- Validazione
  - Tutte le risorse sono nel namespace corretto

---
<details>
<summary>Soluzione</summary>

```
k create ns kustom-ns

vi kustomization.yaml

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: kustom-ns
resources:
- deployment.yaml
- service.yaml

k apply -k .
k get deploy -n kustom-ns
```

</details>

---

## KUS-9 — Secret Generator

- Generare Secret:
  - username=admin
  - password=secret

- Usare `secretGenerator`

- Validazione
  - Secret creato

---
<details>
<summary>Soluzione</summary>

```
vi kustomization.yaml

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
secretGenerator:
- name: db-secret
  literals:
  - username=admin
  - password=secret

k apply -k .
k get secret
```

</details>

---

## KUS-10 — Name Suffix

- Applicare suffix:
  - `-test`

- Validazione
  - Nome risorse modificato

---
<details>
<summary>Soluzione</summary>

```
vi kustomization.yaml

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
nameSuffix: -test

k apply -k .
k get deploy
```

</details>

---

## KUS-11 — Patch JSON6902

- Modificare Deployment
  - replicas: 5

- Usare `patchesJson6902`

- Validazione
  - Replica aggiornata

---
<details>
<summary>Soluzione</summary>

```
vi patch.json

[
  {
    "op": "replace",
    "path": "/spec/replicas",
    "value": 5
  }
]

vi kustomization.yaml

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
patchesJson6902:
- target:
    version: apps/v1
    kind: Deployment
    name: deploy
  path: patch.json

k apply -k .
k get deploy
```

</details>

---

## KUS-12 — Multiple Resources

- Aggiungere risorsa:
  - configmap.yaml

- Validazione
  - Tutte le risorse applicate

---
<details>
<summary>Soluzione</summary>

```
vi configmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: extra-config
data:
  key: value

vi kustomization.yaml

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
- configmap.yaml

k apply -k .
k get all
```

</details>

---

