
###  Network policies(6 esercizi)

## NP-1 — Default Deny (podSelector)

- Namespace: `net-secure`

- Creare NetworkPolicy
  - Nessun traffico consentito

- Validazione
  - Pod non comunicano tra loro
---
<details>
<summary>Soluzione</summary>
  
```
kubectl create ns net-secure

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: net-secure
spec:
  podSelector: {}   # match all pods
  policyTypes:
  - Ingress

k run pod1 --image=busybox -n net-secure -- sleep 3600
k run pod2 --image=busybox -n net-secure -- sleep 3600

k exec -it pod1 -n net-secure -- sh
k get netpol -n net-secure
k describe netpol default-deny -n net-secure
```
</details>

---

## NP-2 — Allow Ingress da Label

- Consentire traffico solo da Pod con label `role=frontend`
- Validazione
  - Solo frontend comunica

---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

## NP-3 — Allow Egress DNS

- Consentire egress solo verso porta 53
- Validazione
  - DNS funziona
  - Altro traffico bloccato
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

### NP-4 — Namespace Selector

- Consentire traffico solo da namespace `trusted`
- Validazione
  - Altri namespace bloccati

---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

### NP-5 — Port Restriction

- Consentire traffico solo su porta 80
- Validazione
  - Porta diversa bloccata
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

### NP-6 — Combined Policy

- Consentire:
  - Ingress da namespace trusted
  - Solo porta 443

- Validazione
  - Traffico limitato correttamente
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---
