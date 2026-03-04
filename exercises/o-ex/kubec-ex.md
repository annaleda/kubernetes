
###  Kubeconfig (6 esercizi)

## KC-1 — Visualizzare Contesti

- Verificare il kubeconfig corrente
- Obiettivo
  - Elencare tutti i contesti disponibili

---
<details>
<summary>Soluzione</summary>
  
```
k config get-contexts
k config current-context  
```
</details>

---

## KC-2 — Cambiare Contesto

- Cambiare contesto attivo
- Obiettivo
  - Impostare come corrente il contesto `dev-cluster`
- Validazione
  - `kubectl config current-context`

---
<details>
<summary>Soluzione</summary>
  
```
k config use-context dev-cluster
```
</details>

---

## KC-3 — Impostare Namespace di Default

- Impostare namespace di default `testing`
per il contesto corrente

- Validazione
  - `kubectl config view --minify` mostra namespace
---
<details>
<summary>Soluzione</summary>
  
```
k create ns testing
k config set-context --current --namespace=testing

```
</details>

---

### KC-4 — Creare Nuovo Context

- Creare un nuovo context chiamato `staging-context`
- Specifiche
  - cluster: staging-cluster
  - user: staging-user
  - namespace: staging
- Validazione
  - Context visibile in `get-contexts`

---
<details>
<summary>Soluzione</summary>
  
```
k create ns staging
k config set-credentials staging-user --client-certificate=cert.crt --client-key=cert.key
k config set-context staging-context --cluster=staging-cluster --user=staging-user --namespace=staging
```
</details>

---

### KC-5 — Usare KUBECONFIG Multipli

- Impostare variabile ambiente `KUBECONFIG`
con due file kubeconfig
- Obiettivo
  - Verificare che kubectl legga entrambi
- Validazione
  - `kubectl config get-contexts` mostra contesti combinati
---
<details>
<summary>Soluzione</summary>
  
```
# Supponendo di avere:
# config-dev.yaml
# config-staging.yaml
# esporta la variabile d'ambiente

export KUBECONFIG=config-dev.yaml:config-staging.yaml
```
</details>

---

### KC-6 — Debug Accesso Cluster

- Simulare errore di connessione
- Obiettivo
  - Identificare problema (cluster endpoint errato o credenziali)
  - Correggere kubeconfig
- Validazione
  - `kubectl get nodes` funziona
---
<details>
<summary>Soluzione</summary>
  
```
k config view
# verifica endpoints, memoria, cpu

kubectl config set-cluster cluster-name --server=https://correct-api-server:6443
```
</details>

---
