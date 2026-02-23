
# 📘 Kubernetes in locale con Kind (Windows)

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
