# 📘 CKAD – Esercizi e Soluzioni

---

### 41 Pod con restartPolicy Never
**Esercizio:**  Create a Pod one-shot with image busybox that runs echo done and does not restart.
<details>
<summary>Soluzione</summary>

```bash
kubectl run one-shot \
  --image=busybox \
  --restart=Never \
  -- echo done

# Verifica
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>
Di default kubectl run può creare un Deployment a seconda delle opzioni.

Con --restart=Never forzi la creazione di un Pod standalone.

Quando il comando termina, il Pod va in stato Completed e non viene riavviato.

Senza --restart=Never → rischio di creare un Deployment.

</details>

---

### 42
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 43
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 44
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 45
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 46
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 47
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 48
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 49
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 50
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 51
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 52
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 53
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 54
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 55
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 56
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 57
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 58
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 59
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

### 60
**Esercizio:** 
<details>
<summary>Soluzione</summary>

```bash
kubectl get pod one-shot
```
</details>
<details>
  <summary>Teoria</summary>

</details>

---

