# Taints and Tolerations / Affinity
## Teoria

---
In Kubernetes lo scheduler decide su quale nodo eseguire un Pod.
Per influenzare questa decisione possiamo usare:

- Taints & Tolerations → controllo “negativo” (repulsione)

- Affinity / Anti-Affinity → controllo “positivo” (attrazione o preferenza)


## Taints and Tolerations

I Taints si applicano ai nodi e servono per respingere i Pod.

<img width="833" height="618" alt="Immagine 2026-02-24 134609" src="https://github.com/user-attachments/assets/d739c7ba-e618-41e8-99c5-90fbe4950eb6" />

Una taint ha questo formato:

key=value:effect


Dove `effect` può essere:

- `NoSchedule` → il Pod non viene schedulato
- `PreferNoSchedule` → lo scheduler cerca di evitarlo
- `NoExecute` → i nuovi pod non saranno programmati sul nodo e che i pod esistenti sul nodo, se presenti, saranno sfrattati se non tollerano il taint.

<img width="873" height="325" alt="Immagine 2026-02-24 135344" src="https://github.com/user-attachments/assets/5844af51-a8d6-4184-8f00-619add76c89e" />

---

## Affinity

L’**Affinity** serve a dire allo scheduler dove preferisco andare.

Esistono due grandi categorie:

### Node Affinity

Permette di scegliere nodi con determinate **label**.

Esempio concettuale:

> “Schedula questo Pod solo su nodi con label `disktype=ssd`”

Può essere:

- `requiredDuringSchedulingIgnoredDuringExecution` → obbligatoria
- `preferredDuringSchedulingIgnoredDuringExecution` → preferenza

---

### Pod Affinity / Anti-Affinity

- **Pod Affinity** → voglio stare vicino ad altri Pod
- **Pod Anti-Affinity** → voglio stare lontano da altri Pod

Esempio:

- Microservizi che comunicano spesso → **Pod Affinity**
- Repliche della stessa app su nodi diversi → **Pod Anti-Affinity** (alta disponibilità)

---
