
###  Kubeconfig (6 esercizi)

## KC-1 — Visualizzare Contesti

- Verificare il kubeconfig corrente
- Obiettivo
  - Elencare tutti i contesti disponibili
- Validazione
  - `kubectl config get-contexts`
---
<details>
<summary>Soluzione</summary>
  
```  
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
```
</details>

---
