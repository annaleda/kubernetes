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

---

## ReplicaSet
* Garantisce un numero definito di pod in esecuzione.
* Se un pod muore, viene ricreato.
* Usa label selector per identificare i pod.
* Non gestisce aggiornamenti (lo fa il Deployment).

---

## Deployment
* Gestisce ReplicaSet e aggiornamenti.
* Permette:
  * Rolling update
  * Rollback
  * Scaling
* È la risorsa principale per applicazioni stateless.

---

## Namespace
* Isolamento logico delle risorse.
* Permette ambienti separati (dev, test, prod).
* Namespace di default: `default`.

---

## Labels & Selectors
* Coppie chiave/valore (`app=nginx`).
* Usate per:
  * Collegare Service ↔ Pod
  * ReplicaSet ↔ Pod
  * Deployment ↔ Pod

---

## Annotations
* Metadati non usati per selezione.
* Informazioni descrittive o per tool esterni.

---

## Esercizi

1. Creare un pod nginx via YAML.
2. Creare un deployment con 3 repliche.
3. Scalare il deployment a 5 repliche.
4. Creare un namespace "dev" e deployare un pod dentro di esso.

---


