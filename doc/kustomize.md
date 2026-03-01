## 🧩 Kustomize

Kustomize è uno strumento integrato in `kubectl` che permette di personalizzare i manifest Kubernetes **senza usare template**.

A differenza di Helm:
- Non usa variabili
- Non usa motore di templating
- Lavora tramite patch e overlay

È incluso in kubectl (non serve installarlo separatamente nelle versioni moderne).

---

### Verifica Kustomize

```bash
kubectl version --client
kubectl kustomize .
kubectl apply -k .
```
---

### 📁 Struttura Base

Esempio directory:

```bash
my-app/
├── deployment.yaml
├── service.yaml
└── kustomization.yaml
```
---
### 📄 kustomization.yaml minimo
```bash
resources:
- deployment.yaml
- service.yaml
```
Applicazione:

```bash
kubectl apply -k .
```

### Modificare il numero di repliche (Patch)

deployment.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
```
patch.yaml

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
```

kustomization.yaml

```bash
resources:
- deployment.yaml

patches:
- path: patch.yaml
```
Applicare:

```bash
kubectl apply -k .
```
---
### Generare ConfigMap automaticamente

`configMapGenerator`

```bash
configMapGenerator:
- name: app-config
  literals:
  - ENV=prod
  - DEBUG=false
```
Kustomize creerà automaticamente la ConfigMap.

---
### Aggiungere prefisso o suffisso ai nomi

```bash
namePrefix: dev-
```
Tutti i resource name diventeranno:
  - dev-web
  - dev-service

```bash
nameSuffix: -v1
```
Tutti i resource name diventeranno:
  - dev-web-v1
  - dev-service-1

### Aggiungere label comuni
```bash
commonLabels:
  environment: dev
```

### Image Transformation

Permette di modificare immagini senza toccare il deployment.

Cambiare tag
```
images:
- name: nginx
  newTag: 1.27
```
Cambiare registry (prefix)
```
images:
- name: nginx
  newName: registry.company.com/nginx
```
Cambiare nome + tag
```
images:
- name: nginx
  newName: registry.company.com/nginx
  newTag: v2
```

### Patch – Strategic Merge Patch 

- Modifica campi YAML come se stessi riscrivendo solo una parte del manifest.

Esempio: Cambiare Replicas

deployment.yaml
```
spec:
  replicas: 1
```
patch.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
```
kustomization.yaml
```
resources:
- deployment.yaml

patches:
- path: patch.yaml
```

---

### JSON Patch (patchesJson6902)

- Usa operazioni esplicite:
   - op
   - path
   - value

Esempio
```
patchesJson6902:
- target:
    group: apps
    version: v1
    kind: Deployment
    name: web
  patch: |-
    - op: replace
      path: /spec/replicas
      value: 3
```
- Significato di patch: |-
```
| → stringa multilinea

- → rimuove newline finale
```
> Serve perché JSON6902 richiede una stringa YAML.

| Operazione | Significato        |
| ---------- | ------------------ |
| add        | Aggiunge campo     |
| replace    | Sostituisce valore |
| remove     | Rimuove campo      |

- path
  - Formato tipo:
```
/spec/template/spec/containers/0/image
```
Regole:

```
/ = root
Array usa indice numerico
```
- value
   - Nuovo valore da applicare.

Esempio:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.25
```
Vuoi:

- Cambiare l’immagine in nginx:1.27,senza modificare il file originale.

- Soluzione con JSON Patch (patchesJson6902)
kustomization.yaml
```
resources:
- deployment.yaml
```

patchesJson6902:
```
- target:
    group: apps
    version: v1
    kind: Deployment
    name: web
  patch: |-
    - op: replace
      path: /spec/template/spec/containers/0/image
      value: nginx:1.27
```
---

- configMapGenerator
```
configMapGenerator:
- name: app-config
  literals:
  - ENV=prod
  - DEBUG=false
```
> Genera automaticamente ConfigMap con hash nel nome.
 
---

### Pattern Base / Overlay

Struttura tipica:

```bash
my-app/
├── base/
│   ├── deployment.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

📄 base/kustomization.yaml
```bash
resources:
- deployment.yaml
```
📄 overlays/prod/kustomization.yaml
```bash
resources:
- ../../base
patches:
- path: replica-patch.yaml
```
Applicare overlay:

```bash
kubectl apply -k overlays/prod
```
🔍 Generare YAML senza applicare

```bash
kubectl kustomize .
```
Utile per vedere il risultato finale.

### Differenza Helm vs Kustomize
|Helm	              |Kustomize                   |
|-------------------|----------------------------|
|Template engine	  | Patch engine               |
|values.yaml	      |Overlay                     | 
|Package manager	  |Personalizzazione YAML      |
|Installazione chart|	Modifica manifest esistenti|


Esempio versione:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml

namePrefix: dev-

images:
- name: nginx
  newTag: 1.27
```

---
