- [ Home ](../readme.md)   | [ Teoria ](../arguments.md)   
--- 
### 📘 Kubernetes in locale con Kind (Windows) + Kubeconfig

## 1️⃣ Prerequisiti

Installare:

* Docker Desktop
* kubectl
* Kind

Verifiche:

```powershell
docker ps
kubectl version --client
kind --version
```

---

## 2️⃣ Creare un cluster

```powershell
kind create cluster
```

Verifica:

```powershell
kubectl get nodes
```

Output atteso:

```
kind-control-plane   Ready   control-plane
```

---

## 3️⃣ Test rapido

### Creare un deployment

```powershell
kubectl create deployment nginx --image=nginx
```

### Verificare i pod

```powershell
kubectl get pods
```

### Esporre il deployment

```powersshell
kubectl expose deployment nginx --type=NodePort --port=80
kubectl get svc
```

---

## 4️⃣ Eliminare il cluster

```powershell
kind delete cluster
```

---

# Kubeconfig 

> Il kubeconfig non è parte degli obiettivi principali dell’esame CKAD,
> ma è fondamentale nella pratica reale.

---

## Cos’è il kubeconfig

Il **kubeconfig** è il file che contiene le informazioni per connettersi a un cluster Kubernetes.

Di default si trova in: `~/.kube/config`


Serve a `kubectl` per sapere:

- A quale **cluster** connettersi
- Con quale **utente**
- Quale **contesto** usare

---

## Struttura logica del kubeconfig

Un kubeconfig contiene tre elementi principali:

- **Clusters** → dove si trova il cluster (endpoint API Server)
- **Users** → credenziali (certificati, token, ecc.)
- **Contexts** → combinazione di cluster + user + namespace

Il **context** è ciò che usi realmente quando esegui `kubectl`.

---

## Vedere il kubeconfig

Mostrare il contesto attuale:

```bash
kubectl config current-context
```

Vedere tutti i contesti:

```bash
kubectl config get-contexts
```
Vedere l’intero kubeconfig:

```bash
kubectl config view
```
Cambiare contesto
```bash
kubectl config use-context <nome-context>
```
Usare un namespace di default nel context

Impostare un namespace nel context corrente:
```bash
kubectl config set-context --current --namespace=dev
```
Dopo questo comando non serve più usare -n dev.

Usare un kubeconfig diverso

Puoi specificare un file kubeconfig alternativo:
```bash
kubectl --kubeconfig=altro-config.yaml get pods
```
Oppure tramite variabile d’ambiente:
```bash
export KUBECONFIG=altro-config.yaml
```
---

### kubeadm

---

kubeadm è uno strumento CLI usato per Inizializzare e configurare un cluster Kubernetes.
È il tool ufficiale per creare cluster Kubernetes in modo semplice e standardizzato.

>  `kubeadm ` non è un componente del cluster, ma uno strumento di bootstrap.



Con  `kubeadm ` puoi:
 - Creare il control plane
 - Configurare certificati TLS
 - Generare file kubeconfig
 - Joinare nodi worker
 - Aggiornare il cluster

Per inizializzare il cluster si usa il comando `kubeadm init `

Questo comando:
  - Avvia il control plane
  - Genera certificati
  - Crea file  `/etc/kubernetes/admin.conf `
  - Mostra il comando  `kubeadm join `
  - Aggiungere un worker node

Sul nodo worker:

 ```
kubeadm join <ip-control-plane>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
 ```

Questo collega il nodo worker al control plane.

---

- Collegamento con kubeconfig

Dopo   `kubeadm init `, viene creato:

 ```
/etc/kubernetes/admin.conf
 ```

Per usare kubectl come utente normale:

 ```
mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config
 ```

Ora kubectl può comunicare con il cluster.

---
- Static Pods creati da kubeadm

Dopo l’inizializzazione, kubeadm crea i componenti del control plane come Static Pods.

Puoi verificarli con:

 ```
ls /etc/kubernetes/manifests/
 ```

- Vengono avviati dal kubelet:
  - kube-apiserver.yaml
  - kube-controller-manager.yaml
  - kube-scheduler.yaml
  - etcd.yaml


---

Differenza tra tool principali
   - kubeadm        ->	Crea cluster
   - kubectl	      -> Gestisce risorse
   - kubelet        ->	Gira su ogni nodo
   - etcd           ->	Database del cluster
   - kube-apiserver ->	Espone API Kubernetes

---

 Flusso 
 ```
kubeadm init
        ↓
Crea control plane (Static Pods)
        ↓
Genera certificati
        ↓
Genera kubeconfig
        ↓
kubectl usa kubeconfig
        ↓
kubectl comunica con kube-apiserver
```
---
