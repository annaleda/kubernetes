
# 📘 Kubernetes in locale con Kind (Windows) + Kubeconfig

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

# Kubeconfig (Non è argomento CKAD)

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
