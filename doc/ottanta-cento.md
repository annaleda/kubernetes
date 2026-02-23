# 📘 CKAD – Esercizi e Soluzioni

---

### 81 Deployment con requirements multipli
**Esercizio:** Create a Deployment named payment-api in namespace prod.
The Deployment should:

use image nginx:1.25

have 3 replicas

expose container port 8080

define CPU limit of 300m

define readiness probe on / port 8080
<details>
<summary>Soluzione</summary>

```bash
kubectl create ns prod

kubectl create deployment payment-api \
  --image=nginx:1.25 \
  -n prod \
  --dry-run=client -o yaml > deploy.yaml
```

Modificare il file:

```bash
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: 300m
        readinessProbe:
          httpGet:
            path: /
            port: 8080
```

```bash
kubectl apply -f deploy.yaml
kubectl get deploy -n prod
```
</details>
<details>
  <summary>Teoria</summary>
Deployment completo tipico CKAD:

Namespace corretto

replicas sotto spec

containerPort nel container

resources sotto containers

readinessProbe deve usare la porta corretta

 Se readiness usa porta sbagliata → Pod Ready = False.
</details>

---

### 82 Fix Deployment esistente
**Esercizio:** Update Deployment frontend so that:

image is nginx:1.26

replicas are 4
<details>
<summary>Soluzione</summary>

```bash
kubectl set image deployment/frontend nginx=nginx:1.26
kubectl scale deployment frontend --replicas=4

kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
Non serve riscrivere YAML.

set image modifica container image

scale modifica replicas

 Il nome container deve combaciare con quello nel Deployment.
</details>

---

### 83 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 84 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 85
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 86 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 87 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 88 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 89 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 90 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 91 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 92 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 93 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 94 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 95 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 96 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 97 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 98 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 99 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---

### 100 
**Esercizio:** 

<details>
<summary>Soluzione</summary>

```bash
kubectl get deploy frontend
```
</details>
<details>
  <summary>Teoria</summary>
</details>

---
