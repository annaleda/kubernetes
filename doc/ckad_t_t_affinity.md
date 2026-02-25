# Node, Taints and Tolerations / Affinity
## Teoria

---
## Node

Un **Node** è una macchina (VM o fisica) che esegue i Pod.

Può essere:

- **Worker Node** → esegue i Pod  
- **Control Plane Node** → gestisce il cluster (API Server, Scheduler, etcd…)

Nel CKAD ti concentri principalmente sui **worker node**.

---

## Componenti principali di un Node

Ogni Node esegue:

- **kubelet** → comunica con l’API Server e avvia i Pod assegnati
- **container runtime** → esegue i container (es. containerd)
- **kube-proxy** → gestisce networking e regole dei Service

---

## Scheduling dei Pod

Quando crei un Pod:

1. Lo **Scheduler** sceglie un Node
2. La scelta si basa su:
   - Resource requests
   - Node Selector
   - Affinity / Anti-affinity
   - Taints & Tolerations
3. Il Pod viene assegnato al Node
4. Il kubelet lo avvia

---

Visualizzare i node:

```bash
kubectl get nodes
```
Dettaglio di un node:

```bash
kubectl describe node <nome-node>
```
Vedere su quale node gira un Pod:
```bash
kubectl get pod -o wide
```
Aggiungere una label a un Node (utile per nodeSelector):
```bash
kubectl label nodes <nome-node> disktype=ssd
```

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
## NodeSelector

In Kubernetes il **nodeSelector** è il modo più semplice per dire allo scheduler:

> “Esegui questo Pod solo su nodi che hanno una certa label.”

---

<img width="1138" height="589" alt="Immagine 2026-02-24 140157" src="https://github.com/user-attachments/assets/ad9d9562-7614-4529-aceb-d7ef5aaf9523" />

Ogni nodo può avere delle **label**, ad esempio:

- `disktype=ssd`
- `environment=prod`
- `gpu=true`

Con `nodeSelector`, il Pod verrà schedulato **solo** su nodi che matchano esattamente quelle label.

Se nessun nodo soddisfa la condizione → il Pod rimane in **Pending**.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: nginx
    image: nginx
```
<img width="1115" height="590" alt="Immagine 2026-02-24 140305" src="https://github.com/user-attachments/assets/34ec0ae2-2451-44fb-a77b-360c78edfec8" />

---

## Affinity

L’**Affinity** serve a dire allo scheduler dove preferisco andare.

Esistono due grandi categorie:

### Node Affinity

Permette di scegliere nodi con determinate **label**.

<img width="1079" height="580" alt="Immagine 2026-02-24 140843" src="https://github.com/user-attachments/assets/5b80fc47-aa2d-44be-a44f-6549b77cc6b9" />

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
