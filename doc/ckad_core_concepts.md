# CKAD Core Concepts

## Teoria

---

## Pod
* Unità minima deployabile in Kubernetes.
* Può contenere uno o più container.
* I container nel pod condividono:
  * Network (stesso IP)
  * Storage (volumi)
* I pod sono effimeri.
* In produzione si usano tramite Deployment.
  
<img width="1207" height="476" alt="Immagine 2026-02-24 121543" src="https://github.com/user-attachments/assets/39e0ef6d-923e-4ec3-90ba-4924dda03b48" />


- Prima creazione veloce → `kubectl create -f pod.yaml`
- Workflow reale / modifiche → `kubectl apply -f pod.yaml`
- Vuoi generare YAML → `--dry-run=client -o yaml`
```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```
- Vuoi salvare il YAML su file → usa `>` (redirect output)
```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
```
---

## ReplicaSet
* Garantisce un numero definito di pod in esecuzione.
* Se un pod muore, viene ricreato.
* Usa label selector per identificare i pod.
* Non gestisce aggiornamenti (lo fa il Deployment).

<img width="619" height="648" alt="Immagine 2026-02-24 121319" src="https://github.com/user-attachments/assets/cacac28c-97af-4a3a-bcf1-153ce1984123" />

- Prima creazione veloce → `kubectl create -f replicaset.yaml`
- Workflow reale / modifiche → `kubectl apply -f replicaset.yaml`
- Vuoi generare YAML → `--dry-run=client -o yaml`
```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
```
- Vuoi salvare il YAML su file → usa `>` (redirect output)
```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > replicaset.yaml
```
- Con `vi replicaset.yaml` sostituisci il `kind` in `ReplicaSet` e inserisci:

```bash
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
   ...
```
---


## Deployment
* Gestisce ReplicaSet e aggiornamenti.
* Permette:
  * Rolling update
  * Rollback
  * Scaling
* È la risorsa principale per applicazioni stateless.

<img width="1237" height="672" alt="image" src="https://github.com/user-attachments/assets/bdb8e63a-a03f-44fb-a39f-18fdc5c45c24" />

- Prima creazione veloce → `kubectl create -f deployment.yaml`
- Workflow reale / modifiche → `kubectl apply -f deployment.yaml`
- Vuoi generare YAML → `--dry-run=client -o yaml`
```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
```
- Vuoi salvare il YAML su file → usa `>` (redirect output)
```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```
---

<img width="1051" height="460" alt="Immagine 2026-02-24 120518" src="https://github.com/user-attachments/assets/7b9f6bb1-ae95-4f42-a38b-233093224345" />

## Namespace
* Isolamento logico delle risorse.
* Permette ambienti separati (dev, test, prod).
* Namespace di default: `default`.

<img width="1184" height="619" alt="Immagine 2026-02-24 120951" src="https://github.com/user-attachments/assets/eacae482-2964-497c-a5bc-2c058a7157e5" />

---

## Labels & Selectors
* Coppie chiave/valore (`app=nginx`).
Le label sono fondamentali in Kubernetes e vengono usate per:

- Collegare **Service ↔ Pod**
- Collegare **ReplicaSet ↔ Pod**
- Collegare **Deployment ↔ Pod**
- Definire regole nelle **NetworkPolicy**
- Applicare **Affinity / Anti-Affinity**
- Selezionare risorse con `kubectl`

> Label = etichetta

> Selector = filtro

> Controller / Service / NetworkPolicy usano selector per collegare risorse

> Kubernetes usa label per identificare,
> selector per scegliere.

<img width="727" height="536" alt="Immagine 2026-02-24 144906" src="https://github.com/user-attachments/assets/65cd2545-ab2f-43e3-8faa-be6440d5454c" />
<img width="727" height="222" alt="Immagine 2026-02-24 145048" src="https://github.com/user-attachments/assets/cf3cc83d-fd76-4ebf-8996-ea0276350fdd" />
<img width="723" height="527" alt="Immagine 2026-02-24 145312" src="https://github.com/user-attachments/assets/32f12be6-f2f5-4043-914f-0cc9d2231d40" />

---


## Annotations
* Metadati non usati per selezione.
* Informazioni descrittive o per tool esterni.

<img width="604" height="598" alt="Immagine 2026-02-24 145429" src="https://github.com/user-attachments/assets/cec38abc-1bbc-483b-b597-594812a101ba" />

---

## Esercizi

1. Creare un pod nginx via YAML.
2. Creare un deployment con 3 repliche.
3. Scalare il deployment a 5 repliche.
4. Creare un namespace "dev" e deployare un pod dentro di esso.

---


