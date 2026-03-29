- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
---
### Network Policies (12 esercizi)

---

## NP-0 — Allow All Ingress

- Namespace: `net-open`

- Creare NetworkPolicy
  - Consentire tutto il traffico in ingresso verso i Pod del namespace

- Specifiche
  - PolicyType: `Ingress`
  - Pod selector: tutti i Pod
  - Ingress: consentito da qualsiasi sorgente

- Validazione
  - I Pod possono ricevere traffico in ingresso

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns net-open
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
  namespace: net-open
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - {}
```

```sh
kubectl apply -f np-0.yaml
kubectl get netpol -n net-open
```

</details>

---

## NP-1 — Default Deny Ingress

- Namespace: `net-secure`

- Creare NetworkPolicy
  - Nessun traffico in ingresso consentito

- Validazione
  - I Pod non comunicano tra loro in ingresso

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns net-secure
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: net-secure
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

```sh
kubectl apply -f np-1.yaml
kubectl get netpol -n net-secure
kubectl describe netpol default-deny -n net-secure
```

</details>

---

## NP-2 — Allow Ingress da label

- Namespace: `net-secure`

- Consentire traffico in ingresso solo da Pod con label:
  - `role=frontend`

- Validazione
  - Solo i Pod frontend possono connettersi

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: net-secure
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

```sh
kubectl apply -f np-2.yaml
kubectl run frontend --image=busybox --labels=role=frontend -n net-secure -- sleep 3600
kubectl run backend --image=busybox --labels=role=backend -n net-secure -- sleep 3600
```

</details>

---

## NP-3 — Allow Egress DNS

- Namespace: `net-secure`

- Consentire solo traffico egress verso DNS
  - porta UDP 53
  - porta TCP 53

- Validazione
  - DNS permesso
  - Altro egress bloccato

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: net-secure
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

## NP-4 — Namespace Selector

- Namespace target: `net-secure`

- Consentire traffico in ingresso solo da namespace con label:
  - `access=trusted`

- Validazione
  - Namespace trusted consentito
  - altri namespace bloccati

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-trusted-ns
  namespace: net-secure
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

```sh
kubectl label ns trusted access=trusted
```

</details>

---

## NP-5 — Port Restriction

- Namespace: `net-secure`

- Consentire traffico in ingresso solo su:
  - TCP 80

- Validazione
  - Porta 80 consentita
  - altre porte bloccate

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-80
  namespace: net-secure
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

## NP-6 — Combined Policy

- Namespace: `net-secure`

- Consentire:
  - Ingress da namespace con label `access=trusted`
  - Solo su porta 443

- Validazione
  - Traffico limitato correttamente

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-trusted-443
  namespace: net-secure
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

## NP-7 — Default Deny Egress

- Namespace: `egress-lock`

- Creare NetworkPolicy
  - Nessun traffico in uscita consentito

- Validazione
  - I Pod non possono uscire verso altri servizi

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns egress-lock
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: egress-lock
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

```sh
kubectl apply -f np-7.yaml
kubectl describe netpol default-deny-egress -n egress-lock
```

</details>

---

## NP-8 — Allow ingress solo verso pod backend

- Namespace: `app-net`

- Consentire traffico in ingresso solo ai Pod con label:
  - `tier=backend`

- Solo da Pod con label:
  - `tier=frontend`

- Validazione
  - policy applicata solo ai backend

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns app-net
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-from-frontend
  namespace: app-net
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
```

</details>

---

## NP-9 — Namespace + Pod selector combinati

- Namespace target: `app-net`

- Consentire traffico solo da:
  - namespace con label `team=blue`
  - Pod con label `role=client`

- Validazione
  - Solo quei Pod in quei namespace possono entrare

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-blue-clients
  namespace: app-net
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: blue
      podSelector:
        matchLabels:
          role: client
```

</details>

---

## NP-10 — Allow egress verso un CIDR specifico

- Namespace: `egress-lock`

- Consentire egress solo verso:
  - `10.0.0.0/24`

- Validazione
  - Solo quel range IP è consentito

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-cidr
  namespace: egress-lock
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
```

</details>

---

## NP-11 — Deny except same namespace app=db

- Namespace: `db-net`

- Consentire traffico in ingresso ai Pod:
  - `app=db`

- Solo da Pod:
  - nello stesso namespace
  - con label `app=api`

- Validazione
  - accesso limitato ai Pod api interni al namespace

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns db-net
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-only-from-api
  namespace: db-net
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api
```

</details>

---

## NP-12 — Combined ingress + egress

- Namespace: `full-lock`

- Requisiti
  - Ingress consentito solo da Pod `role=frontend`
  - Egress consentito solo verso porta 53 UDP/TCP

- Validazione
  - policyTypes contiene entrambi
  - Ingress ed Egress limitati

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns full-lock
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-egress-combo
  namespace: full-lock
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
  egress:
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

</details>

---

## Cheatsheet NetworkPolicy

```sh
# vedere le networkpolicy
kubectl get netpol -A

# descrivere una policy
kubectl describe netpol <name> -n <namespace>

# vedere labels namespace
kubectl get ns --show-labels

# vedere labels pod
kubectl get pods --show-labels -n <namespace>
```

---

## Mini schema mentale

```text
podSelector        -> a quali Pod si applica la policy
from / to          -> chi può entrare / dove si può uscire
namespaceSelector  -> filtra namespace
podSelector        -> filtra Pod
ports              -> limita le porte
ipBlock            -> limita CIDR/IP

Se una policy seleziona un Pod:
- Ingress non esplicitamente consentito = negato
- Egress non esplicitamente consentito = negato
```
