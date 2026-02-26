## 🧩 Kustomize

Kustomize è uno strumento integrato in `kubectl` che permette di personalizzare i manifest Kubernetes **senza usare template**.

A differenza di Helm:
- Non usa variabili
- Non usa motore di templating
- Lavora tramite patch e overlay

È incluso in kubectl (non serve installarlo separatamente nelle versioni moderne).

---

### Verifica Kustomize

Controlla che sia disponibile:

```bash
kubectl version --client
```

Test rapido:

```bash
kubectl kustomize .
```
Oppure:
```bash
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

resources:
- deployment.yaml
- service.yaml

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

### Aggiungere label comuni
```bash
commonLabels:
  environment: dev
```  
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



---
