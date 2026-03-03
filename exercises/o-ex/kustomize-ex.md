
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
```
</details>

---

### KUS-4 — ConfigMap Generator

- Generare ConfigMap da file:

      app.properties

- Usare configMapGenerator

- Validazione
  - ConfigMap generata automaticamente
---
<details>
<summary>Soluzione</summary>
  
```  
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
```
</details>

---
