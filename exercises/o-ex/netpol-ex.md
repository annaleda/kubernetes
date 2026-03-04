
###  Network policies(6 esercizi)

---

## NP-0 — Allow All Ingress

- Namespace: `net-open`

- Creare NetworkPolicy
  - Consentire tutto il traffico in ingresso verso i pod del namespace
- Spec
  - Policy Type: Ingress
  - Pod selector: tutti i pod
  - Ingress: consentito da qualsiasi sorgente
- Validazione
  - I pod possono comunicare liberamente in ingresso

---
<details>
<summary>Soluzione</summary>
  
```
kubectl create ns net-open

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
  namespace: net-open
spec:
  podSelector: {}          # match all pods

  policyTypes:
  - Ingress

  ingress:
  - {}                     # permette qualsiasi traffico ingress

```
</details>

---

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
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector: {}   # si applica a tutti i pod del namespace
  policyTypes:
  - Ingress

  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend

kubectl run frontend --image=busybox --labels=role=frontend -- sleep 3600
kubectl run backend --image=busybox --labels=role=backend -- sleep 3600
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
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress

  egress:
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
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
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-trusted-ns
spec:
  podSelector: {}
  policyTypes:
  - Ingress

  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          access: trusted
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
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-80
spec:
  podSelector: {}
  policyTypes:
  - Ingress

  ingress:
  - ports:
    - protocol: TCP
      port: 80

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
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-trusted-443
spec:
  podSelector: {}
  policyTypes:
  - Ingress

  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          access: trusted
    ports:
    - protocol: TCP
      port: 443
```
</details>

---
